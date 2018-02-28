# Basic hadoop python streaming

### Hadoop streaming vs. Native Java execution
 
Hadoop provides a streaming API which supports any programming languages that can read from the standard input stdin and write to the standard output stdout.
Hadoop streaming API uses the standard Linux streams as the interface between Hadoop and the program. 
Thus, input data is passed via the stdin to a map function, which processes it line by line and writes to the stdout. 
Input to the reduce function is stdin (which is guaranteed to be sorted by key by Hadoop) and the results are output to stdout.\
(resource: https://www.princeton.edu/researchcomputing/computational-hardware/hadoop/mapred-tut/)

## Steps

## 1. Set all you need for your hadoop cluster
Prerequisites: account at Metacentrum and access to Hadoop - https://www.metacentrum.cz/en/hadoop/\
MetaCentrum Hadoop cluster documentation: https://wiki.metacentrum.cz/w/index.php?title=Hadoop_documentation&setlang=en

Add this to .ssh/config
```
# MetaCentrum

Host hador
HostName hador.ics.muni.cz
User <yourlogin>
Port 22
```

Try to log in to Hadoop Cluster
```
$ ssh hador
```

Metacentrum Hadoop Web Accessibility - https://wiki.metacentrum.cz/wiki/Hadoop_documentation#Web_accessibility   
(Gain Kerberos ticket on your local machine and run web interface 
)
```sh
$ scp <yourlogin>@hador.ics.muni.cz:/etc/krb5.conf .
$ env KRB5_CONFIG=krb5.conf kinit <yourlogin>@META
$ chromium-browser --auth-server-whitelist="*.ics.muni.cz" &
```

Check https://hador-c1.ics.muni.cz:50470/  


## 2. Download Hadoop Streaming jar
```sh
$ wget http://central.maven.org/maven2/org/apache/hadoop/hadoop-streaming/2.6.0/hadoop-streaming-2.6.0.jar
```

## 3. Implement mapper and reducer. Make sure the file has execution permission (chmod +x mapper.py and chmod +x reducer.py)

## 4. Download file which you want to process and put it into Hadoop’s HDFS.
```sh
$ hdfs dfs -mkdir input
$ wget https://goo.gl/KyDfc7 -O shake.txt
$ hdfs dfs -D dfs.block.size=1048576 -put shake.txt /user/<user>/input/shake.txt
$ hdfs dfs -ls
```

## 5. Run the MapReduce job
```sh
$ hadoop jar hadoop-streaming-2.6.0.jar \
    -D mapreduce.job.name='Word count python streaming' \ 
    -input input \ 
    -output output \
    -mapper mapper.py \
    -reducer reducer.py \
    -file mapper.py \
    -file reducer.py 
```

Check the sorted output
```sh
$ hdfs dfs -get output .\
$ sort -k 2 -g -r part-00000 > sorted.txt
```

Hador MapReduce Job history - https://hador-c2.ics.muni.cz:19890/jobhistory

## Documentation
```sh
yourlogin@hador:~$ hadoop version\
Hadoop 2.6.0-cdh5.10.0\
```
http://hadoop.apache.org/docs/r2.6.0/hadoop-mapreduce-client/hadoop-mapreduce-client-core/HadoopStreaming.html