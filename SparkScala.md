# Spark Scala CheetSheet
This puts my utils of spark together, especially stats related
http://twitter.github.io/effectivescala/
http://www.tutorialspoint.com/scala/index.htm
https://twitter.github.io/scala_school/

## Use spark
download a version, from https://spark.apache.org/downloads.html, cd that home path
```
spark-shell
```

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
In this exmaple, each executor has an object of ran to produce random numnbers, it is better than put the ran in closure in terms of distribution.
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
 
Functions VS methods applied for udfs
No (x:String) things like this needed. 
Anonymous functions are first-class functions → Function values are objects
Assign function values to variables.
Pass function values as arguments to higher order functions
```
val categoricalUDF = udf(Utils.categoricalFromVocabList(Array("F", "M")))
def categoricalUDF(list:Array[String]) = udf(Utils.categoricalFromVocabList(list))
categoricalUDF(Array("F", "M”))
```

Simple udf functions
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

Partially applied functions and udf
Partially applied function is good for giving parameters partially from some perspective.
```
def buckBucket(bucketSize: Int): (String, String) => Int = {
  val func = (col1: String, col2: String) =>
    (Math.abs((col1 + "_" + col2).hashCode()) % bucketSize + 0)
  func
}
val bucketUDF = udf(buckBucket(100)) //here 100 is partially applied in advance. 
Df.withColumn(“x”,bucketUDF(col1, col2))
```
 
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

Functional language and why

Functional programming supports higher-order functions and lazy evaluation features.

Functional programming languages don’t support flow Controls like loop statements and conditional statements like If-Else and Switch Statements. They directly use the functions and functional calls.

Efficient Parallel Programming − Functional programming languages have NO Mutable state, so there are no state-change issues. One can program "Functions" to work parallel as "instructions". Such codes support easy reusability and testability. Especially for big data.

Lazy Evaluation − Functional programming supports Lazy Functional Constructs like Lazy Lists, Lazy Maps, etc.

Apply function in Scala

1. Every function in Scala can be treated as an object, every object can be treated as a function, provided it has the apply method. 
There are many usage cases when we would want to treat an object as a function. 
Such objects can be used in the function notation:
```$xslt
// we will be able to use this object as a function, as well as an object
object Foo {
  var y = 5
  def apply (x: Int) = x + y
}

Foo (1) // using Foo object in function notation 
```
2. The most common scenario of using an apply function is a factory pattern, and companion. Synatic sugar, and multiple ways of building objects
```
class Foo(x) {
  val y = 5
}

object Foo {
  def apply (x: Int) = New class Foo(x)
  def apply (x: Float) = New class Foo(x)
}

val foo1 = Foo (1) // build an object 
```
3. You can also define an apply function in class, after you build an object of that class in whatever way.
then you can call apply function. In this exmaple, 
```
class c1(x:Float) ={
    def apply(x:String)= {
        x.toInt
    }
}
class c2(y:Float) extends c1(y:Float)
object c2 ={
    def apply(y:Float) = new c2(y)
}

val tmp = c2(10.0)
tmp("10.0")
```

In big data, what happens in driver/executer

Scala implicit

implicit class

implicit object(need better understanding)
    Like any object, an implicit object is a singleton but it is marked implicit so that the compiler can find if it is looking for an implicit value of the appropriate type.
    A typical use case of an implicit object is a concrete, singleton instance of a trait which is used to define a type class.
    People use implicit object instead of implicit class so you don't need to explicitly import the class with implicits, since implicits in companion object will be searched by Scala compiler as well.
```
trait TensorNumeric[@specialized(Float, Double) T]
abstract UndefinedTensorNumeric(typeName:String) extends TensorNumeric
object TensorNumeric{
    implicit object NumericFloat extends UndefinedTensorNumeric[Float]{}
    implicit object NumericDouble extends UndefinedTensorNumeric[Double]{}
}
val predict: Int = ev.toType[Int](_output.max(1)._2.valueAt(1))

```
## MLLib and ML Vectors
ML Vector 
DataFrame related APIs use org.apache.spark.ml.linalg.Vector, but the old mllib use org.apache.spark.mllib.linalg.Vector, 
org.apache.spark.mllib.util.MLUtils.convertVectorColumnsToML and other APIs are used to convert the data from one type to another.

## BigDL related data stracture
BigDL Tensor, basic data strcture to hold the data
```
trait Activity
class Table extends Activity
trait Tensor[T] extends Serializable with TensorMath[T] with Activity 
class DenseTensor extends Tensor
``` 
BigDL Sample, MiniBatch
    Sample represents the features and labels of a data sample, features and labels are tensors.
    BigDL optimizer takes RDD[Sample[T]] or DataSet[D] currently. Eventually, everything should be packed into these two, I use RDD[Sample[T]] often.
    RDD[Sample[T]] is converted into Iterater[MiniBatch[T]] while building an Optimizer using SampleToMiniBatch transformer. 
    //how to get grab data from every node to build the MiniBatch?
BigDL DataSet and Transformer, all kinds of rdd manipulation 
    DataSet are sent to Optimizer directly. It takes a transformer and manipluate the data. In the optimizer, it only caches the original dataset, and apply a couple of transformer later
```   
Trait AbstractDataSet[D,DataSequence]{
  def transform[C: ClassTag](transformer: Transformer[D, C]): DataSet[C]
  def -> [C: ClassTag](transformer: Transformer[D, C]): DataSet[C] = {this.transform(transformer)}
} 
```
ImageFrame, ImageFeature and Transformer
    It includes all image transformation, eventually to a list of ImageFeature which is a hashMap of sample and label(could be maipulated by BigDL optimizer and related) and other attributes could be manipluated by OpenCV
    Transformer defines a function which you want to apply to some data, Transformer can take an imageFrame and ImageFrame can take a transformer.
    Eventually, features are array of numbers in dataframe to be sent to DLClassifier's fit 
```
trait Transformer{
    def ->[C](other:Transformer): new ChainedTransformer(this, other) // take a transformer and chain it
}
abstract FeatureTransformer extends Transformer{
    def transform(feature:ImageFeature):ImageFeature //use OpenCV methodologies to transform
    def apply(imageFrame: ImageFrame): ImageFrame = {
        imageFrame.transform(this)
     }
}
```
``` 
Trait ImageFrame{
    def transform(transformer:Transformer):ImageFrame
    def ->(transformer:FeatureTransformer):ImageFrame = this.transform(transformer)
}
```
```
class ImageFeature extends Serializable {
  private val state = new mutable.HashMap[String, Any]() // it uses HashMap to store all these data,original bytes read from image file, an opencv mat, pixels in float array, image label, sample, meta data and so on
  val sample = "sample"
  val label = "label"
}
``` 
BigDL AbstractModule, all kinds of layers, Container for all models
``` 
abstract class AbstractModule[A <: Activity: ClassTag, B <: Activity: ClassTag, T: ClassTag](implicit ev: TensorNumeric[T]) extends Serializable with InferShape{
abstract class TensorModule[T: ClassTag](implicit ev: TensorNumeric[T]) extends AbstractModule[Tensor[T], Tensor[T], T]
class Linear[T:ClassTag](val inputSize: Int, val outputSize: Int)(implicit ev: TensorNumeric[T]) extends TensorModule[T] with Initializable
```
```
abstract class Container[A <: Activity : ClassTag,B <: Activity : ClassTag, T: ClassTag](implicit ev: TensorNumeric[T]) extends AbstractModule[A, B, T]
abstract class DynamicContainer[A <: Activity : ClassTag, B <: Activity : ClassTag, T: ClassTag](implicit ev: TensorNumeric[T]) extends Container[A, B, T]
class Sequential[T: ClassTag](implicit ev: TensorNumeric[T]) extends DynamicContainer[Activity, Activity, T]
```

