# apache-Ignite
apache ignite poc work   

### Apache Ignite vs Apache Spark main difference
> Main difference: Ignite is an in-memory computing system (treats RAM as the primary storage). Whereas Spark (and others) _only_ use RAM for computation and not as storage. The memory-first approach, is faster because the system can provide far better indexing, improved fetch time, avoid ser/deser etc.

### Does Apache Ignite replaces Apache Spark ?
> TBD


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


#### Loading and running queries   
You may have to use the maven shade plugin to build a jar that you can run on docker or other Linux virtual servers   

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-shade-plugin</artifactId>
	<version>1.7</version>
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
			<transformer
				implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
				<resource>META-INF/spring.handlers</resource>
			</transformer>
			<transformer
				implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
				<resource>META-INF/spring.schemas</resource>
			</transformer>
			<transformer
				implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
				<mainClass>org.apache.ignite.zeppelin.Node</mainClass>
			</transformer>
		</transformers>
	</configuration>

	<executions>
		<execution>
			<phase>package</phase>
			<goals>
				<goal>shade</goal>
			</goals>
		</execution>
	</executions>
</plugin>

```

#### Inspecting contents of Data Grid

Ignite provides command line interface (Visor) for exploring cache. You can initiate Visor using the following command   
> bin/ignitevisorcmd.sh


