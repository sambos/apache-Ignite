## Example: Spark with Ignite Cache
- This example shows how to configure Ignite cache and then load data into it for running sql queries   
- it assumes you have configured the ignite project and started ignite node   
- you can use the ignite-cache.xml file provided under ignite distribution   
-- `apache-ignite-1.5.0.final-src\examples\config   `


#### Add dependencies to your maven project

```xml

   <properties>
     <ignite.version>1.5.0.final</ignite.version>
   </properties>
   
	    <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>4.1.0.RELEASE</version>
	    </dependency>

 		<dependency>
		    <groupId>org.apache.ignite</groupId>
		    <artifactId>ignite-core</artifactId>
		    <version>${ignite.version}</version>
		</dependency> 
		
		<dependency>
			<groupId>org.apache.ignite</groupId>
			<artifactId>ignite-spark_2.10</artifactId>
			<version>${ignite.version}</version>
		</dependency>

		<dependency>
		    <groupId>org.apache.ignite</groupId>
		    <artifactId>ignite-spring</artifactId>
		    <version>${ignite.version}</version>
		</dependency>
		
		<dependency>
		    <groupId>org.apache.ignite</groupId>
		    <artifactId>ignite-indexing</artifactId>
		    <version>${ignite.version}</version>
		</dependency> 	    

```
#### Scala example

```scala
    /** Type alias for `QuerySqlField`. */
    type ScalarCacheQuerySqlField = QuerySqlField @field
    
    // case class with indexes defined
      case class Person (
        @ScalarCacheQuerySqlField(index = true) val id: Int,
        @ScalarCacheQuerySqlField(index = true) val name: String,
        @ScalarCacheQuerySqlField(index = true) val salary: Int
        ) extends Serializable {

        }
        
    //create spark context    
    val sc = new SparkContext( new SparkConf().setMaster("local[*]").setAppName("IgniteSparkExample"))
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    
    // this is used to implicitly convert an RDD to a DataFrame.
    import sqlContext.implicits._    
    
   //create igniteContext with example cache    
   val igniteContext = new IgniteContext[String, Person](sc,  "example-cache.xml")
   //create cache object
   val cacheCfg = new CacheConfiguration[String, Person]()
   
  //set cache name 
  cacheCfg.setName("partition")
  // set indexed types
  cacheCfg.setIndexedTypes(classOf[String], classOf[Person]) // table has "Person" name
  //load cash from cache configuration
  val cacheRdd = igniteContext.fromCache(cacheCfg)
  
// example 1
    val nonCacheRdd = sc.parallelize(0 to 10000, 2).map(i ⇒ new Person(i, "name" + i, i * 100))
    val ncDf = nonCacheRdd.toDF()
    ncDf.registerTempTable("temp")
    val ncdf2 = ncDf.sqlContext.sql("select *  from temp where salary > 1000")
    println(ncdf2.count())  
    
// Example 2 - ignite will automatically create table with name PERSON and index keys as defined in case class
    //cacheRdd.savePairs(sc.parallelize(0 to 10000, 2).map(i ⇒ (String.valueOf(i), new Person(i, "name" + i, i * 100))))
    cacheRdd.saveValues(sc.parallelize(0 to 10000, 2).map(i ⇒  new Person(i, "name" + i, i * 100)))
    val df2 = cacheRdd.sql("select id, name, salary from Person where name = ? and salary = ?", "name50", 5000)
    val df3 = cacheRdd.sql("select *  from Person where salary > ?", 1000)
    //df2.printSchema
    println(df3.count)    
```

