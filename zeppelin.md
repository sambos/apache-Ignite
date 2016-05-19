# Zeppelin
[Apache Zeppelin](https://zeppelin.incubator.apache.org/) is a neat web based notebook style interface for interactive data analytics. you can run your spark, hive or sql queries interactively. You can even run your scala program and see the results and share them.       
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
#### Zeppelin Impala Interpreter with Kerberos authenticatation
You can use the [steps described here in HiveImpalaJdbcDriver.java](https://github.com/sambos/ApacheSpark/blob/master/Impala.md) and incorporate into your own zeppeling interpreter. Zeppelin by default uses LDAP authentication and does not provide kerberos authentication out of the box. You can however write your own interpreter using the following steps...
* Copy hive-sites.xml to ZEPPELIN_HOME/conf
* Make sure kerberos is enabled under hive-sites.xml
```xml
  <property>
    <name>hive.server2.authentication</name>
    <value>kerberos</value>
  </property>
```
* Create your own ImpalaInterpreter that extends from org.apache.zeppelin.interpreter.Interpreter , you can refer to the [HiveInterpreter code](https://github.com/apache/incubator-zeppelin/blob/master/hive/src/main/java/org/apache/zeppelin/hive/HiveInterpreter.java) or copy the same code in your class the modify the code to use kerberos that initializes connection as follows :
```java
## Initializes the interpreter as follows .. this will enable 

  static {
    Interpreter.register(
        "impala",
        "impala",
        ImpalaInterpreter.class.getName(),
        new InterpreterPropertyBuilder()
            .add(COMMON_MAX_LINE, MAX_LINE_DEFAULT, "Maximum line of results")
            .add(DEFAULT_DRIVER, "org.apache.hive.jdbc.HiveDriver", "Hive JDBC driver")
            .add(DEFAULT_URL, "jdbc:hive2://localhost:10000", "The URL for HiveServer2.")
            .add(DEFAULT_USER, "hive", "The hive user")
            .add(DEFAULT_PASSWORD, "", "The password for the hive user")
            .add(DEFAULT_KEYTAB_LOCATION, "/home/alias/alias.keytab", "default location of the keytab file")
            .add(DEFAULT_REALM,"UNITOPR.UNITINT.TEST.STATEFARM.ORG","default realm").build());
  }
  
## replace the connection initialization code as follows...

    if (null == connection) {
      Properties properties = propertiesMap.get(propertyKey);
      Class.forName(properties.getProperty(DRIVER_KEY));
      String url = properties.getProperty(URL_KEY);
      String user = properties.getProperty(USER_KEY);
      String password = properties.getProperty(PASSWORD_KEY);
      String realm = properties.getProperty("realm");
	  String keytabloc = properties.getProperty(KEYTAB_KEY); 
	  
	  
	  if(null != keytabloc){
		  
		  System.setProperty("java.security.krb5.realm", realm);
		  System.setProperty("java.security.krb5.kdc", realm + ":88");
		  org.apache.hadoop.conf.Configuration conf = new org.apache.hadoop.conf.Configuration();
          conf.set("hadoop.security.authentication", "Kerberos");
          UserGroupInformation.setConfiguration(conf);
          
	      try{    
	          UserGroupInformation.loginUserFromKeytab(user + "@" + realm, keytabloc); //"qdbr@UNITOPR.UNITINT.TEST.STATEFARM.ORG"
	      } catch (IOException e) {
	          e.printStackTrace();
	      }      
	      
		  connection = DriverManager.getConnection(url);
		  
	  }else if (null != user && null != password) {
        connection = DriverManager.getConnection(url, user, password);
      } else {
    	  
        connection = DriverManager.getConnection(url, properties);
      }
    }
```
* Package the impala interpreter and its jars to ZEPPELIN_HOME/interpreter/impala folder
* Create an entry into zeppelin-sites.xml for this interepreter
* Restart the zeppelin
* Open zeppelin home, add new interpreter for impala, provide the url, and location for your keytab
* save the interpreter
* enjoy with %impala
