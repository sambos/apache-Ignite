# apache-Ignite
apache ignite poc work   

Nice article to try out Apache Ignite and Apache Spark with Zeppelin   
> Use this link  https://github.com/vkulichenko/zeppelin-demo

Note:- The spark-master.sh is not scripted to run on windows properly.. I tried to run it as follows manually and works just fine ..   
I have tried this demo with Apache Spark 1.5.1 and Apache Ignite 1.5.0-final 
```
./bin/spark-class.cmd org.apache.spark.deploy.master.Master --host <my_pc>   

For the worker:   
./bin/spark-class.cmd org.apache.spark.deploy.worker.Worker spark://<mypc>:7077   
To wrap these in services, I've user yasw or nssm.   
```   



