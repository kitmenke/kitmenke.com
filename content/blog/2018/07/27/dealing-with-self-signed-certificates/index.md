---
author: Kit
categories:
- Notes
date: 2018-07-27T10:45:00Z
title: Dealing with Self-signed Certificates
---

When working in a corporate environment, you'll often have to deal with self-signed certificates that are used
to secure internal dev tools like Artifactory or a git server. 

If your PC isn't setup to trust these certificates, you'll get an error like:
```
sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

Atlassian has some [really good documentation around this](https://confluence.atlassian.com/kb/unable-to-connect-to-ssl-services-due-to-pkix-path-building-failed-779355358.html)
including a small utility named SSLPoke that you can use to test connectivity.

```bash
java SSLPoke yourserver.local 443
# optionally add -Djavax.net.ssl.trustStore="C:\some\path\to\cacerts" to point to a specific keystore
```

If you point to a keystore which can't be found, you'll get this especially cryptic error:
```
java.lang.RuntimeException: Unexpected error: java.security.InvalidAlgorithmParameterException: the trustAnchors parameter must be non-empty
```

Now that you can reproduce the issue, export the self-signed certificate from the server using one of two ways.

## Using Chrome Dev Tools

In chrome, open dev tools -> Security tab -> View Certificate.

Go to the details tab, and click Copy to File.

Choose DER encoded binary X.509 (.CER).

Save to your PC as `yourserver.cer`.

Convert the .cer to a .pem:
```
openssl x509 -inform der -in yourserver.cer -out yourserver.pem
```

## Using openssl

If you have git bash, then you can run this easily on Windows:

```
echo "" | openssl s_client -servername yourserver.local -connect yourserver.local:443 -prexit 2>/dev/null | sed -n -e '/BEGIN\ CERTIFICATE/,/END\ CERTIFICATE/ p'
```

Save this output as yourserver.pem.

## Import the pem into your keystore

Now all that is left is to import it into your local keystore which will exist somewhere like 
`C:\Program Files\Java\jdk1.8.0_72\jre\lib\security\`. You may have to add it to multiple keystores if you use an IDE
like IntelliJ which comes with its down embedded JDK `C:\Program Files\JetBrains\IntelliJ IDEA 2017.3.2\jre64\lib\security\`.

To import:
```
keytool -import -noprompt -trustcacerts -alias yourserver -file yourserver.pem -keystore cacerts -storepass changeit
```

Test again with SSLPoke to verify you get the `Successfully connected` message!