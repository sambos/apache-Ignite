# Zeppelin

#### script that kicks of Zeppelin and Spark Master/Workder nodes
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
