# apache-Ignite
apache ignite poc work   

Nice article to try out Apache Ignite and Apache Spark with Zeppelin   
> Use this link  https://github.com/vkulichenko/zeppelin-demo

#### Starting Spark Master & Worder Nodes
Note:- The spark-master.sh is not scripted to run on windows properly.. I tried to run it as follows manually and works just fine ..   
I have tried this demo with Apache Spark 1.5.1 and Apache Ignite 1.5.0-final 
```
./bin/spark-class.cmd org.apache.spark.deploy.master.Master --host <my_pc>   

For the worker:   
./bin/spark-class.cmd org.apache.spark.deploy.worker.Worker spark://<mypc>:7077   
To wrap these in services, I've user yasw or nssm.   
```   
#### Starting Zeppelin
If you want to run the zeppelin on Cygwin without the "Could not find or load main class org.apache.zeppelin.server.ZeppelinServer" error, You should change the zeppelin running command in the "bin/zeppelin-daemon.sh" as follows.
```
  nohup nice -n $ZEPPELIN_NICENESS $ZEPPELIN_RUNNER $JAVA_OPTS -classpath `cygpath -wp $CLASSPATH` $ZEPPELIN_MAIN >> "${ZEPPELIN_OUTFILE}" 2>&1 < /dev/null &

```

#### Start Ignite node
Clone and import eclipse project https://github.com/vkulichenko/zeppelin-demo.git
run the Node.java program and you should see `Ignite node started OK`

Also if you would like to see web console for H2 - use the following command   
`./ignite.sh -J-DIGNITE_H2_DEBUG_CONSOLE`   




