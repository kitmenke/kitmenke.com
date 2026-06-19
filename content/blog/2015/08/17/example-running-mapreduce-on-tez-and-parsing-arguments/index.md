---
author: Kit
categories:
- Hadoop
date: 2015-08-17T11:02:19Z
guid: http://kitmenke.com/blog/?p=802
id: 802
title: Example running MapReduce on tez and parsing arguments
---

I ran into some trouble executing a simple MapReduce program on TEZ. I kept reading about the special &#8220;-Dmapreduce.framework.name=yarn-tez&#8221; parameter you could pass to your MR job to make it automatically switch frameworks without modifying the configs for your entire cluster but couldn't get the argument to be parsed correctly.

After doing some research, I found that your MapReduce class must extend **Configured** and implement **Tool**. In addition to parsing the _generic arguments_ correctly, you also get arguments parsed for you automatically.

This allows you to run your job like this:

```
hadoop jar HelloMapReduce-0.0.1-SNAPSHOT.jar com.ehi.hadoop.mapreduce.WordCount -Dmapreduce.framework.name=yarn-tez -D my.property=bananas /user/someone/input /user/someone/output
```

There are three types of parameters here:

  1. Generic hadoop parameters (in this case -Dmapreduce.framework.name=yarn-tez allows me to override the default mapreduce framework)
  2. MapReduce parameters parsed automatically just by extending the **Configured** class (-D my.property=bananas, note the space after the -D)
  3. Parameters not parsed automatically (/user/someone/input and /user/someone/output are last)

Here is the code based on the [Hadoop Word Count example](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0) and [SSaikia_JtheRocker's StackOverflow answer](http://stackoverflow.com/a/17760457/98933):

```java
/**
 * To run on the cluster: 
 * hadoop jar HelloMapReduce-0.0.1-SNAPSHOT.jar com.ehi.hadoop.mapreduce.WordCount -Dmapreduce.framework.name=yarn-tez -D my.property=bananas /user/someone/input /user/someone/output
 * 
 * Based on the Hadoop Word Count example:
 * http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0
 * 
 * Along with SSaikia_JtheRocker's answer here:
 * http://stackoverflow.com/a/17760457/98933
 * 
 * @author Kit Menke
 *
 */
public class WordCount extends Configured implements Tool {
    private static final Logger LOG = LoggerFactory.getLogger(WordCount.class);

    public static void main(String[] args) throws Exception {
        LOG.info("com.ehi.hadoop.mapreduce.WordCount main");
        int exitCode = ToolRunner.run(new WordCount(), args);
        System.exit(exitCode);
    }

    @Override
    public int run(String[] args) throws Exception {
        LOG.info("com.ehi.hadoop.mapreduce.WordCount running...");

        Configuration conf = getConf();

        // get the argument passed using -D my.property=value
        String myPropertyValue = conf.get("my.property");
        LOG.info("myPropertyValue: {}", myPropertyValue);

        // get the arguments passed last (without -D)
        Path inputPath = new Path(args[0]);
        Path outputDir = new Path(args[1]);
        LOG.info("Input: {}, Output: {}", inputPath, outputDir);

        // job setup
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        return (job.waitForCompletion(true) ? 0 : 1);
    }

    public static class TokenizerMapper extends Mapper&lt;Object, Text, Text, IntWritable&gt; {

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            // Configuration conf = context.getConfiguration();

            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer&lt;Text, IntWritable, Text, IntWritable&gt; {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable&lt;IntWritable&gt; values, Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
}
```

&nbsp;