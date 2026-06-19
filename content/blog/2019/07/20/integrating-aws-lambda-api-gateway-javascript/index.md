---
author: Kit
date: 2019-07-20T15:00:06-05:00
#lastmod: 2019-07-20T15:00:06-05:00
categories:
- Cloud
tags:
- AWS
- Serverless
title: Integrating AWS Lambda with API Gateway in JavaScript
---

My latest project in AWS has involved AWS API Gateway and AWS Lambda deployed using [Serverless Framework](https://serverless.com/). 
The integration between AWS Lambda and API Gateway is a little tricky to get exactly right, so I'm documenting some key
findings here.

### Integration between Lambda and API Gateway

First and foremost, all exceptions thrown from within your lambda function should be caught and handled. You can choose
whether to log and/or pass an error message back to the client but from the lambda perspective, it should always return
success. As for what that means exactly in the code for a node.js lambda connected with API Gateway, here is an example
for the happy path scenario:

```javascript
module.exports.hello = function(event, context, callback) {
  // callback's first parameter is null indicating this lambda succeeded
  // API Gateway will return 200 to the client since we've given statusCode
  callback(null, {
    "statusCode": 200,
    "headers": { "content-type": "application/json" },
    "body": JSON.stringify({ "message": "everything is ok!" })
  });
}
```

In the event that something goes wrong (for example some bad arguments are passed to your API and you want to let the
client know) you still return success from lambda but change the statusCode:

```javascript
module.exports.hello = function(event, context, callback) {
  // callback's first parameter is null indicating this lambda succeeded
  // Setting statusCode to 400 means API Gateway will return 400 Bad Request
  // to the client
  callback(null, {
    "statusCode": 400, // bad request
    "headers": { "content-type": "application/json" },
    "body": JSON.stringify({ "message": "something went horribly wrong" })
  });
}
```

In both these cases, the first argument to the callback is `null` which means success from the **AWS Lambda** 
perspective. We can easily control how API Gateway responds by changing the `statusCode`.

### Things Not To Do

If an error is thrown from your lambda you will get a `502 BAD GATEWAY` and a generic error. The same is true if you 
return an error using the callback.

```javascript
module.exports.hello = function(event, context, callback) {
  // Don't do this! Don't let errors escape your lambda!
  throw new Error("something went horribly wrong");
  // Don't do this either:
  callback(Error("Something went horribly wrong"));
}

// resuls in 502 BAD GATEWAY and a generic message from API Gateway:
// {"message": "Internal server error"}
```

See [AWS Lambda Function Handler in Node.js](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html)
for more information.

### A Better Way

Wouldn't it be nice if there was an easier way to communicate success or failure to API Gateway? And do so in a way that
clearly represents what we are doing? For example:

```javascript
const Response = require('../common/response.js');

module.exports.hello = function(event, context, callback) {
  try {
    // your code goes here to create a result...
    let result = { "message": "hello world!" };
    Response.success(callback, result);
  } catch (e) {
    Response.error(callback, 'Unhandled exception in handler', e);
  }
}
```

Well you are in luck, below is a class you can use to easily handle this stuff:

```javascript
/*
 * Response exports
 */

const HEADERS = {
  "content-type": "application/json",
  "Access-Control-Allow-Origin": "*", // Required for CORS support to work
  "Access-Control-Allow-Credentials": true // Required for cookies, authorization headers with HTTPS
};

 /**
  * Return a success response to API Gateway
  * @param {*} callback 
  * @param {*} data 
  */
module.exports.success = (callback, data) => {
  if (!callback) {
    throw new Error("Callback was not specified!");
  }
  callback(null, {
    statusCode: 200,
    headers: HEADERS,
    body: JSON.stringify(data)
  });
};

/**
 * Return an error response to API Gateway
 * @param {*} callback 
 * @param {*} msg 
 * @param {*} ex 
 */
var errorResponse = (callback, msg, ex, code) => {
  if (!callback) {
    throw new Error("Callback was not specified!");
  }
  const status = typeof code === "number" && code >= 100 && code <= 530 ? code : 500;
  console.error("errorResponse: ", status, msg, "exception: ", ex);
  // TODO: Add code and more_info properties like twilio? https://www.twilio.com/docs/usage/twilios-response
  const data = {
    "status": status,
    "message": msg
  };
  var response = {
    "statusCode": status,
    "headers": HEADERS,
    "body": JSON.stringify(data),
    "isBase64Encoded": false
  };
  // This is a "success" from the lambda point of view because we are passing
  // response as the second parameter.
  // However, by changing the statusCode we can control how API Gateway responds
  // and by returning a code other than 200 signal an error.
  // More info https://github.com/serverless/serverless/issues/4119
  callback(null, response);
};

module.exports.error = errorResponse;
module.exports.badRequest = (callback, msg, e) => {
  errorResponse(callback, msg, e, 400);
};
module.exports.unauthorized = (callback, msg, e) => {
  errorResponse(callback, msg, e, 401);
};
module.exports.forbidden = (callback, msg, e) => {
  errorResponse(callback, msg, e, 403);
};

/**
 * Use to error out the Lambda. This will return a 502 Bad Gateway from API Gateway
 * so most of the time you probably should use error (above) instead of fatal.
 * @param {*} callback 
 * @param {*} msg 
 */
module.exports.fatal = (callback, msg) => {
  callback(new Error(msg));
}
```
Good luck!