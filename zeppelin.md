# Zeppelin
[Apache Zeppelin](https://zeppelin.incubator.apache.org/) is neat web based notebook style interface for interactive data analytics. 
Apache Zeppelin is pretty straight forward to install and run on linux/mac. But you may run into classpath related issues (where interpreters were not loaded) with Windows, just like i went through. Eventually i discarded the idea and installed it on my mac or virtual host. Here is a neat little script that kicks off the apache spark and zeppelin that you may want to try instead of starting and stopping them manually. This script checks if there are already running processes.


#### script that kicks of Zeppelin and Spark Master/Worker nodes
```shell

#!/bin/bash
SPARKHOME=/<path>/majeed/spark-1.5.2-bin-hadoop2.6
ZEPPELINHOME=/<path>/majeed/zeppelin/zeppelin-0.5.6-incubating-bin-all

PID=""
check_process() {
  echo "checking $1"
  [ "$1" = "" ]  && return 0;
  PID=($(jps | grep $1 | awk '{print $1}'))

        if [ "$PID" == "" ]
        then
          echo "not running"
        else
          echo "is already running with PID=$PID"
          kill -9 $PID
        fi
}

start_process(){

        cd $SPARKHOME
        ./bin/spark-class org.apache.spark.deploy.master.Master --host your.host.com &
        sleep 3
        echo $! "spartk master node started"

        ./bin/spark-class org.apache.spark.deploy.worker.Worker spark://your.host.com:7077 &
        sleep 3
        echo $! "spark worker node started"

        cd $ZEPPELINHOME
        ./bin/zeppelin-daemon.sh restart
        echo $! "zeppelin started"
}

check_process "Master";
check_process "Worker";
check_process "ZeppelinServer";
start_process
echo "done!"

```

#### Apache Zeppelin Restful API
Apache zeppelin has a neat REST interface that allows you to do CRUD operations on its notebook and paragraphs. It also exposes REST interface for starting and queries jobs. I find them useful for testing or mocking you prototype.

```
http://localhost:9090/api/notebook

For more APIs ,visit    
https://zeppelin.incubator.apache.org/docs/0.5.6-incubating/rest-api/rest-notebook.html   
```
