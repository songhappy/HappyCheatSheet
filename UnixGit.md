# Unix command and git command CheetSeet

1. Get fetch all branches from remote
```
Git fetch --all
```
2. git copy branches from a remote and build the same ones on local
```
git branch -r | grep -v '\->' | while read keras; do git branch --track "${keras#origin/}" "$keras"; done
git pull --all
```
3. git remote add url
```
git remote add intel-analytics https://github.com/intel-analytics/analytics-zoo.git
```

4. Set up local of spark in intellij
-Dspark.master="local[1]”


5. Get the loss
```
cat output  | grep "Loss is" |  awk 'NF>1{print $NF}' | sed -e "s/.\n/\n/g" > loss.txt
```
analytics-zoo-bigdl_0.7.1-spark_2.1.0-0.4.0-SNAPSHOT-jar-with-dependencies.jar
6. Install jars in local environment
```
mvn install:install-file -Dfile=bigdl-SPARK_2.2-0.5.0-jar-with-dependencies.jar -DgroupId=com.intel.analytics.bigdl -DartifactId=bigdl-spark-2.2 -Dversion=0.5.0 -Dpackaging=jar
mvn install:install-file -Dfile=analytics-zoo-bigdl_0.7.1-spark_2.1.0-0.4.0-SNAPSHOT-jar-with-dependencies.jar -DgroupId=com.intel.analytics.zoo -DartifactId=analytics-zoo-bigdl_0.7.1-spark_2.1.0  -Dversion=0.4.0-SNAPSHOT -Dpackaging=jar
```

7. Git rebase
```
git pull --rebase  intel-analytics master
```

8. CTRL SHIFT CMD + for superscript
CTRL CMD - for subscript



