# Spark Scala CheetSheet
This puts my utils of spark together, especially stats related
http://twitter.github.io/effectivescala/
http://www.tutorialspoint.com/scala/index.htm
https://twitter.github.io/scala_school/

## UDF and partial functions

Percentile, usually 5th, 25th, 50th, 75th, 95th are used
```
val percentile = udf((values: scala.collection.mutable.WrappedArray[Float], p: Float) => {

      val sortedValues = values.sorted
      val index = (sortedValues.length * p).toInt

      if (sortedValues.length % 2 == 0) {
        (sortedValues(Math.max(index - 1, 0)) + sortedValues(index)) / 2
      } else {
        sortedValues(index)
      }
    })
```

Window Function 
```
def recommend4Items(featureDF: DataFrame, maxUsers: Float): DataFrame = {
  val results = predictUserItemPair(featureDF)
  results.groupBy("prediction").count().show()
  val window = Window.partitionBy("itemId").orderBy(desc("prediction"), desc("probability"))
  results.withColumn("rank", rank.over(window))
    .where(col("rank") <= maxUsers)
    .drop("rank")
}
```


lazy load fix serialization 
In spark, map, ruduce, groupby and other functions, driver gets everything closed in a closure, then sends it to executors. Other things are done by driver, for example, val ran = new Random(), since the closure needs it then driver has to send it through socket, but ran is not serializable, then it cause problems. So it needs lazy load, lazy means, the driver does not new it or send the object, but the executer will new an object when it needs it. 
```def getNegativeSamples(indexed: DataFrame): DataFrame = {
  val indexedDF = indexed.select("userId", "itemId", "label")
  val minMaxRow = indexedDF.agg(max("userId"), max("itemId")).collect()(0)
  val (userCount, itemCount) = (minMaxRow.getInt(0), minMaxRow.getInt(1))
  val sampleDict = indexedDF.rdd.map(row => row(0) + "," + row(1)).collect().toSet
  val dfCount = indexedDF.count.toInt
  import indexed.sqlContext.implicits._
  
@transient lazy val ran = new Random(System.nanoTime())
 
  val negative = indexedDF.rdd
    .map(x => {
      val uid = x.getAs[Int](0)
      val iid = Math.max(ran.nextInt(itemCount), 1)
      (uid, iid)
    })
    .filter(x => !sampleDict.contains(x._1 + "," + x._2)).distinct()
    .map(x => (x._1, x._2, 1))
    .toDF("userId", "itemId", "label")
  negative
}
```

Abstract class vs trait
Abstract class is class, inherit from one class, it takes constructors.
Trait can be extended with multiple traits,  but trait does not have constructors, will not be able to pass parameters.
 
User defined functions vs Defineed functions. 
```
val categoricalUDF = udf(Utils.categoricalFromVocabList(Array("F", "M")))
def categoricalUDF(list:Array[String]) = udf(Utils.categoricalFromVocabList(list))
categoricalUDF(Array("F", "M”))
```

Partial function is good for user defined functions
```def buckBucket(bucketSize: Int): (String, String) => Int = {
  val func = (col1: String, col2: String) =>
    (Math.abs((col1 + "_" + col2).hashCode()) % bucketSize + 0)
  func
}
val bucketUDF = udf(buckBucket(100))
Df.withColumn(“x”,bucketUDF(col1, col2) )
```

Udf complete format
```
  /**
    * find dynamic geohash given a fine geohash and the dynamic geohash dictionary
    */
  private def matchSingleGeohashUdf(geohashSet: Set[String], max: Int, min: Int) = {
    val func: (String => String) = (arg: String) => {
      val geohashKeys = (for (i <- max to min by -1) yield arg.slice(0, i))
      geohashKeys.find(x => geohashSet.contains(x)).getOrElse(arg.slice(0, min))
    }
    udf(func)
  }
```

Scala has first-class functions → Function values are objects
Assign function values to variables
Pass function values as arguments to higher order functions
```
    def searchGeohash(geohashSet: Set[String], max: Int, min: Int) = {
      val func: (String => String) = (arg: String) => {
        val geohashKeys = (for (i <- max to min by -1) yield arg.slice(0, i))
        geohashKeys.find(x => geohashSet.contains(x)).getOrElse(arg.slice(0, min))
      }
      udf(func)
    }
    locationDF.select(col("*"))
          .withColumn("dynamicGeohash", searchGeohash(dynamicGeohashSet.value, maxPrecision, minPrecision)(col("geohash")))
```



MLLib and ML Vectors
