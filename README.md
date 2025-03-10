# How to write, test, and run Hadoop programs locally with IntelliJ and Maven

The following instructions allow you to write, test, and run a Hadoop program
*locally* in IntelliJ, without configuring the Hadoop environment on your own
machine or using a cluster.

This tutorial is based on [Hadoop: IntelliJ结合Maven本地运行和调试MapReduce程序
(无需搭载Hadoop和HDFS环境)](https://www.polarxiong.com/archives/Hadoop-Intellij%E7%BB%93%E5%90%88Maven%E6%9C%AC%E5%9C%B0%E8%BF%90%E8%A1%8C%E5%92%8C%E8%B0%83%E8%AF%95MapReduce%E7%A8%8B%E5%BA%8F-%E6%97%A0%E9%9C%80%E6%90%AD%E8%BD%BDHadoop%E5%92%8CHDFS%E7%8E%AF%E5%A2%83.html),
[How-to: Create an IntelliJ IDEA Project for Apache
Hadoop](https://blog.cloudera.com/blog/2014/06/how-to-create-an-intellij-idea-project-for-apache-hadoop/)
and [Developing Hadoop Mapreduce Application within IntelliJ IDEA on Windows
10](https://bigdataproblog.wordpress.com/2016/05/20/developing-hadoop-mapreduce-application-within-intellij-idea-on-windows-10/).

## Requirements

- [IntelliJ IDEA](https://www.jetbrains.com/idea/download)
- JDK
- Linux or macOS

## Instructions

**Warning**: Some steps and some interface details may be slightly different in
your version of IntelliJ, due to developments in this program. The main ideas
presented in this document *should* still be valid though.

### Create a new project

In IntelliJ, Go to `File`, `New`, `Project`, then select `Maven` (or `Maven
Archetype`) on the left of the pop-up window.

Set the `Project name` and `Project location`. In this tutorial, we will be
"creating" the popular Hadoop example of the `WordCount` application from the
original Hadoop [MapReduce
Tutorial](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html),
so use `WordCount` as project name. For the location, select a location in your
hard drive that you are going to remember. Now, select a JDK. Hadoop only
supports Java up to version 11 (they are working on supporting more recent
versions), so be sure to have a JDK of that version or earlier. You can use
IntelliJ to download a new one, if needed, by clicking on `Download JDK...` in
the dropdown menu. Any flavor of JDK 11 would work. <!--
, and hit `Next`.
![new_project](images/new_project.png)
![new_maven](images/new_maven.png)
-->
Select `org.apache.maven.archetypes:maven-archetype-quickstart` as the
archetype.

If required, fill in the `GroupId` (e.g.,
with your name) and `ArtifactId` (e.g., with the name of your project, i.e,
`WordCount` in our case), then hit `Finish` (or `Create`).

<!--
![name_loc](images/name_loc.png)
-->

### Configure dependencies
A file called `pom.xml` should open automatically in the IntelliJ editor. If it
does not, find it in the Project browser on the left, and double-click on it to
open it.

If there is any existing `<dependencies>` block please delete anything between
that tag and `</dependencies>` (these two tags included).

Paste the following 2 blocks before the last `</project>` tag.

```xml
<repositories>
    <repository>
        <id>apache</id>
        <url>http://maven.apache.org</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-minicluster</artifactId>
        <version>3.4.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>3.4.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.4.1</version>
    </dependency>
</dependencies>
```

A new version of Hadoop may have come out when you read these instructions. Check
the latest versions available in the Maven repository for
[hadoop-minicluster](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-minicluster),
[hadoop-mapreduce-client-core](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-mapreduce-client-core)
[hadoop-common](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common),
and update the version numbers above accordingly.

The full `pom.xml` is:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>yourname</groupId>
    <artifactId>Wordcount</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <repositories>
        <repository>
            <id>apache</id>
            <url>http://maven.apache.org</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-minicluster</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.4.1</version>
        </dependency>
    </dependencies>
</project>
```

A little icon with a blue `m` and a "refresh" symbol (two arrows in a circle)
should appear somewhere around your window. Please click it: it will make sure
you have all the needed dependencies to run Hadoop programs, i.e., all the Java
libraries needed.

### Create the WordCount class

Select the `Project`&rarr;`src`&rarr;`main`&rarr;`java` folder on the left
pane, then do `File`, `New`, `Java Class` and use **WordCount** as the name of
the class. You can delete the `App` file that was created.

![new_class](images/new_class.png)

Paste the Java code below into `WordCount.java` (this code is taken from the
original Hadoop [MapReduce
Tutorial](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)).

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```
![wordcount](images/wordcount.png)

### Prepare to run
The `WordCount` program scans all text files in the folder specified by the
first command line argument, and outputs the total number of lines in which each
word appears, writing these numbers into files in the folder specified by the
second command line argument.

Create a folder named `input` under the project's root folder (so, at the same
level as the `src` folder), and drag/copy some text files from your computer
inside this folder (or, e.g., look for plaintext files on the web, e.g., of
Shakespeare's plays).

![sample_text](images/sample_text.png)

Then set the two command line arguments. Select `Run`&rarr;`Edit Configurations`.
![edit_config](images/edit_config.png)

Add a new `Application` configuration, set the `Name` to `WordCount`, set the
`Main` class to `WordCount`, and set `Program arguments` to `input output`. This
way, the program will read the input from the `input` folder, and save the
results to the `output` folder. Do *not* create the `output` folder, as Hadoop
will create the folder automatically. If the folder exists, Hadoop will raise
exceptions (thus, you have to manually delete the `output` folder before every
time you run the program).
![new_app](images/new_app.png)
![config](images/config.png)


### Run
Select `Run`&rarr;`Run 'WordCount'` to run the Hadoop program. If you re-run the
program, remember to delete the `output` folder before each run.
![run_app](images/run_app.png)

Results are saved in the file `output/part-r-00000`.
![result](images/result.png)


## Build Runnable JAR with Dependencies
You can build a single jar file with your program and all necessary dependencies
(e.g., Hadoop libraries) so you can transfer the jar file to another machine to
run it.

Add the following `build` block to `pom.xml`, at the same level of
the `repositories` block and the `dependencies` block.
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.14.0</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.6.0</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <filters>
                            <filter>
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <!-- Path to your main class, include package path if needed -->
                                <mainClass>WordCount</mainClass>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Then in a terminal, `cd` to the directory containing the `pom.xml` file, and
run the following command:

```bash
mvn package

```

This command will build `WordCount-1.0-SNAPSHOT-jar-with-dependencies.jar`
and save it in the `target` directory. If it fails, remove the `AppTest` file
from the `src/test/java/` folder, and try again.

To run your Hadoop program, execute the following
command:
```bash
java -jar target/WordCount-1.0-SNAPSHOT-jar-with-dependencies.jar input output
```

## Sample Project
See [WordCount](WordCount/).
