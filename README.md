mysql_to_hive

sqoop import -m 1 --connect 'jdbc:mysql://localhost:3306/word' --username root -password acadgild --table word --hive-import --create-hive-table --hive-table word.word

hdfs_to_mysql

sqoop export -m 1 --connect 'jdbc:mysql://localhost:3306/word' --username root -P --table word --export-dir /word.txt

mysql_to_hdfs

sqoop import -m 1 --connect 'jdbc:mysql://localhost:3306/word' --username root -P --table word --target-dir /import


sqoop import -m 1 --connect 'jdbc:mysql://localhost:3306/word' --username root -password acadgild --table word --target-dir /import --incremental append --check-column sal --last-value 250


lead lag sal

from(select name,sal, lag(sal) over (partition by (dep) order by sal desc) as new from word)s select s.name,s.new where (s.new-s.sal) > 100;

sqoop import --connect jdbc:mysql://localhost:3306/sqoop --username root -P --hive-import --table sqoop1 --create-hive-table --hive-table db.sqtest --m 1 --driver com.mysql.jdbc.Driver


sqoop import 
    --connect jdbc:mysql://localhost/serviceorderdb 
    --username root -P 
    --table customercontactinfo 
    --columns "customernum,customername" 
    --hbase-table customercontactinfo 
    --column-family CustomerName 
    --hbase-row-key customernum -m 1
-
sudo service mysql start
a = load 'hdfs://localhost:9000/project/flume_data/StatewiseDistrictwisePhysicalProgress.xml' using org.apache.pig.piggybank.storage.XMLLoader('row') as (x:chararray);
B = foreach a generate REPLACE(x,'[\\n]','') as x;
C = foreach B generate REGEX_EXTRACT_ALL(x,'.*(?:<State_Name>)([^<]*).*(?:<District_Name>)([^<]*).*(?:<Project_Objectives_IHHL_BPL>)([^<]*).*(?:<Project_Objectives_IHHL_APL>)([^<]*).*(?:<Project_Objectives_IHHL_TOTAL>)([^<]*).*(?:<Project_Objectives_SCW>)([^<]*).*(?:<Project_Objectives_School_Toilets>)([^<]*).*(?:<Project_Objectives_Anganwadi_Toilets>)([^<]*).*(?:<Project_Objectives_RSM>)([^<]*).*(?:<Project_Objectives_PC>)([^<]*).*(?:<Project_Performance-IHHL_BPL>)([^<]*).*(?:<Project_Performance-IHHL_APL>)([^<]*).*(?:<Project_Performance-IHHL_TOTAL>)([^<]*).*(?:<Project_Performance-SCW>)([^<]*).*(?:<Project_Performance-School_Toilets>)([^<]*).*(?:<Project_Performance-Anganwadi_Toilets>)([^<]*).*(?:<Project_Performance-RSM>)([^<]*).*(?:<Project_Performance-PC>)([^<]*).*');
D = FOREACH C GENERATE FLATTEN ($0);
STORE D INTO ' hdfs://localhost:9000/xmlfile/' USING PigStorage (',');
-
loadJson = LOAD '/olympic.json' USING JsonLoader('athelete:chararray,age:INT,country:chararray,year:chararray,closing:chararray,sport:chararray,gold:INT,silver:INT,bronze:INT,total:INT');
-
hive> set hive.auto.convert.join=true;
set hive.auto.convert.join.noconditionaltask=true;
-
import org.apache.hadoop.hive.ql.exec.UDAF;
import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;

public class hiveudf extends UDAF {
  public static class udfeval implements UDAFEvaluator
  {
	  
  public static class column
  {
	  int sum = 0;
  }
private column col = null;

public udfeval()
{
	super();
	init();
}
	@Override
	public void init() {
		// TODO Auto-generated method stub
		col = new column(); 
	}

public boolean iterate(int value)
{
	if(col==null)
	{
		return true;
		
	}
	else
	{
		col.sum = col.sum+value;
		return true;
	}
}
public boolean merge(column other)
{
	if (other == null)
	{
		return true;
		
	}
	else
	{
		col.sum = col.sum + other.sum;
		return true;
	}
	

  }

public column terminatePartial()
{
	return col;
}
public  int terminate()
{
	return col.sum;
	
}

  
  } 
}
---
import java.util.ArrayList;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class hiveudf2 extends UDF{
	public Text evaluate(String str ,ArrayList<String> str1)
	{
		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < str1.size(); i++) {
			sb.append(str1.get(i));
			sb.append(str);
			
		}
		return new Text(sb.toString());
	}
	
}

--

hive> set hive.auto.convert.join=true;

hive> set hive.auto.convert.join.noconditionaltask=true;

--
public class ToolMapReduce extends Configured implements Tool {
 
    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new Configuration(), new ToolMapReduce(), args);
        System.exit(res);
    }

 public int run(String[] args) throws Exception {
 
        // When implementing tool
        Configuration conf = this.getConf();
 
        // Create job
        Job job = new Job(conf, "Tool Job");
        job.setJarByClass(ToolMapReduce.class);
 
        // Setup MapReduce job
        // Do not specify the number of Reducer
        job.setMapperClass(Mapper.class);
        job.setReducerClass(Reducer.class);
 
        // Specify key / value
        job.setOutputKeyClass(LongWritable.class);
        job.setOutputValueClass(Text.class);
 
        // Input
        FileInputFormat.addInputPath(job, new Path(args[0]));
        job.setInputFormatClass(TextInputFormat.class);
 
        // Output
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        job.setOutputFormatClass(TextOutputFormat.class);
 
        // Execute job and return status
        return job.waitForCompletion(true) ? 0 : 1;
    }
}



select TIMESTAMPDIFF(YEAR,DATE_FORMAT(dob,'%Y-%m-%d'),CURDATE()) from dob;



//Collect frequency of all the words who have a length more than some specified dynamic value



import java.io.*;

import org.apache.hadoop.conf.*;
import org.apache.hadoop.fs.*;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.*;
import org.apache.hadoop.mapreduce.lib.input.*;
import org.apache.hadoop.mapreduce.lib.output.*;
import org.apache.hadoop.util.*;

public class DynamicConfigurations extends Configured implements Tool {

    public static void main(String[] args) throws Exception {
      int exitCode = ToolRunner.run(new DynamicConfigurations(), args);
      System.exit(exitCode);
    }

    public int run(String[] args) throws Exception {
      // Check arguments.
    	if (args.length != 2) {
        String usage =
          "Usage: " +
          "hadoop jar Configurations " +
          "<input dir> <output dir>\n";
        System.out.printf(usage);
        System.exit(-1);
      }
 
      String jobName = "WordCount";

      String inputDir = args[0];
      String outputDir = args[1];

      // Define input path and output path.
      Path inputPath = new Path(inputDir);
      Path outputPath = new Path(outputDir);

      Configuration configuration = getConf();
      
      //We will pass parameter as -D min.word.length=<value>
            
      FileSystem fs = outputPath.getFileSystem(configuration);
      fs.delete(outputPath, true);

      Job job = Job.getInstance(configuration);
      job.setJobName(jobName);
      job.setJarByClass(DynamicConfigurations.class);
      job.setMapperClass(WordMapper.class);
      job.setReducerClass(WordReducer.class);
      job.setOutputKeyClass(Text.class);
      job.setOutputValueClass(IntWritable.class);
      TextInputFormat.setInputPaths(job, inputPath);

      job.setOutputFormatClass(TextOutputFormat.class);
      FileOutputFormat.setOutputPath(job, outputPath);

      // Submit the map-only job.
      return job.waitForCompletion(true) ? 0 : 1;
    }
  
  public static class WordMapper extends
      Mapper<LongWritable, Text, Text, IntWritable> {
	  
    @Override
    public void map(LongWritable key, Text value, Context context)
        throws IOException, InterruptedException {

    	String line = value.toString();
    	for (String word : line.split(" ")) {
            if ((word.length() == 0) || (word.length() < Integer.parseInt(context.getConfiguration().get("min.word.length")))) 
            { 
             	continue; 
            }
            context.write(new Text(word), new IntWritable(1));
          }
        }
    }
  
  public static class WordReducer extends
  Reducer<Text, IntWritable, Text, IntWritable> {
 
	@Override
	public void reduce(Text key, Iterable<IntWritable> values, Context context)
	    throws IOException, InterruptedException {

		int sum = 0;
		for (IntWritable value : values) {
			sum += value.get();
		}
		context.write(key, new IntWritable(sum));
	}
  }
}
