### Example: Spark with Ignite Cache
- This example shows how to configure Ignite cache and then load data into it for running sql queries   
- it assumes you have configured the ignite project and started ignite node   
- you can use the ignite-cache.xml file provided under ignite distribution   
-- `apache-ignite-1.5.0.final-src\examples\config   `

```scala
    /** Type alias for `QuerySqlField`. */
    type ScalarCacheQuerySqlField = QuerySqlField @field
    
      case class Person (
        @ScalarCacheQuerySqlField(index = true) val id: Int,
        @ScalarCacheQuerySqlField(index = true) val name: String,
        @ScalarCacheQuerySqlField(index = true) val salary: Int
        ) extends Serializable {

        }
        
    val sc = new SparkContext( new SparkConf().setMaster("local[1]").setAppName("IgniteSparkLogAnalyzer"))
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    
    // this is used to implicitly convert an RDD to a DataFrame.
    import sqlContext.implicits._    
    
   val igniteContext = new IgniteContext[String, Person](sc,  "example-cache.xml")
   val cacheCfg = new CacheConfiguration[String, Person]()
   
  cacheCfg.setName("partition")
  cacheCfg.setIndexedTypes(classOf[String], classOf[Person]) // table has "Person" name
  val cacheRdd = igniteContext.fromCache(cacheCfg)
  
    val nonCacheRdd = sc.parallelize(0 to 10000, 2).map(i ⇒ new Person(i, "name" + i, i * 100))
    val ncDf = nonCacheRdd.toDF()
    ncDf.registerTempTable("temp")
    val ncdf2 = ncDf.sqlContext.sql("select *  from temp where salary > 1000")
    println(ncdf2.count())  
    
    //cacheRdd.savePairs(sc.parallelize(0 to 10000, 2).map(i ⇒ (String.valueOf(i), new Person(i, "name" + i, i * 100))))
    cacheRdd.saveValues(sc.parallelize(0 to 10000, 2).map(i ⇒  new Person(i, "name" + i, i * 100)))
    val df2 = cacheRdd.sql("select id, name, salary from Person where name = ? and salary = ?", "name50", 5000)
    val df3 = cacheRdd.sql("select *  from Person where salary > ?", 1000)
    //df2.printSchema
    println(df3.count)    
```

