---
title: Spark Wordcount vs Scala Wordcount
author: guaqamole
date: 2023-04-28 18:32:00 +0900
categories: [Data, Spark]
tags: [Spark, Scala, RDD, Functional Programming, Spark Wordcount, Practical]
image:
  path: /common/spark.png
---

이번 포스트에선 `Scala`의 **Array API**를 통해 WordCount를 수행하고, `Spark`의 **RDD API**로 동일하게 WordCount를 수행 했을 때 **어떤 차이점이 있는지 알아볼 예정**이다.

<br>

****


### Scala WordCount

#### 1. Install Scala

~~~bash
cd $SPARK_BASE
wget https://downloads.lightbend.com/scala/2.12.15/scala-2.12.15.tgz && tar -zxvf scala-2.12.15.tgz
mv scala-2.12.15 scala && rm -rf scala-2.12.15.tgz 
~~~

<br>

#### 2.  환경 변수 & scala 파일 생성

~~~python
vi ~/.bashrc
export SCALA_HOME=$SPARK_BASE/scala
export SPARK_PROJECT=$SPARK_BASE/projects/spark
export PATH=$SCALA_HOME/bin:$PATH
. ~/.bashrc

# Source
mkdir -p $SPARK_BASE/projects/spark/src/lab/core/wordcount
vi $SPARK_BASE/projects/spark/src/lab/core/wordcount/WordCount.scala

# Destination (for compile)
mkdir -p $SPARK_BASE/projects/spark/target/lab/core/wordcount
~~~

<br>

#### 3. WordCountScala 코드작성

`vi WordCount.scala`

~~~scala
package lab.core.wordcount

import scala.io.Source
import java.io.PrintWriter

object WordCountScala {

  def main(args: Array[String]): Unit = {
    println(">>>> WordCountScala....")
    
    var input = "/spark/spark3/README.md"
    var output = "/spark/spark3/README.wordcount_scala"
    var delimiter = " "
    
    // 데이터(파일) 읽기, 줄(line) 단위 읽기 => arr: Array[String]
    val arr = Source.fromFile(input).getLines.toArray
    println(">>>> Line Count : " + arr.count(line => true))
    
    // 단어(word) 추출 + 레코드(element)화 => arr2: Array[String]
    val arr2 = arr.flatMap(line => line.split(delimiter))

    // 튜플 구성 (word, 1) => arr3: Array[(String, Int)]
    val arr3 = arr2.map(word => (word, 1))

    // 단어(word)별 Group => arr4: Array[(String, Array[(String, Int)])]
    val arr4 = arr3.groupBy(tuple => tuple._1).toArray

    // 그룹핑 배열 값(value)에서 "1"만 남기기 => arr5: Array[(String, Array[Int])]
    val arr5 = arr4.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.map(tuple => tuple._2)))

    // 그룹핑 배열 값(value) 모두 더하여 하나의 값으로 만들기(reduce) => arr6: Array[(String, Int)]
    val arr6 = arr5.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.reduce((v1, v2) => v1 + v2)))
    
    // 단어별 개수(word count) 출력
    arr6.foreach(word_count => println(">>>> word count : " + word_count))
    
    // 개수(count) 내림차순(descending) 정렬 => arr7: Array[(String, Int)]
    val arr7 = arr6.sortBy(tuple => -tuple._2)

    // 단어별 개수(word count) 출력 (count desc)
    arr7.foreach(word_count => println(">>>> word count (count desc) : " + word_count))

    // 정렬 기준 정의 (count desc, word asc) => ordering: Ordering[(String, Int)]
    val ordering = new Ordering[(String, Int)] {
      override def compare(a: (String, Int), b: (String, Int)) = {
        if((a._2 compare b._2) == 0) (a._1 compare b._1) else -(a._2 compare b._2)
      }
    }
    
    // 개수(count) 내림차순(descending) + 단어(word) 오름차순(ascending) 정렬 => arr8: Array[(String, Int)]
    val arr8 = arr6.sortBy(tuple => tuple)(ordering)

    // 단어별 개수(word count) 출력 (count desc, word asc)
    arr8.foreach(word_count => println(">>>> word count (count desc, word asc) : " + word_count))

    // 단어별 개수(word count) 파일로 저장
    new PrintWriter(output) { write(arr8.mkString("\n")); close }
  }

}
~~~

`var input = "/spark/spark3/README.md"`  해당 부분은 파일 경로를 지정해준 내역이다. 참고하자.



<br>

#### 4. 컴파일 & jar로 아카이브

~~~bash
cd $SPARK_BASE/projects/spark
scalac -d target/ src/lab/core/wordcount/WordCountScala.scala

find $SPARK_PROJECT/target
/spark/projects/spark/target
/spark/projects/spark/target/lab
/spark/projects/spark/target/lab/core
/spark/projects/spark/target/lab/core/wordcount
/spark/projects/spark/target/lab/core/wordcount/WordCount$.class
/spark/projects/spark/target/lab/core/wordcount/WordCount$$anon$2.class
/spark/projects/spark/target/lab/core/wordcount/WordCount$$anon$1.class
/spark/projects/spark/target/lab/core/wordcount/WordCount.class

cd $SPARK_PROJECT/target && jar cvf ../wordcountscala.jar ./lab/core/wordcount/*
added manifest
adding: lab/core/wordcount/WordCount$$anon$1.class(in = 4457) (out= 1493)(deflated 66%)
adding: lab/core/wordcount/WordCount$$anon$2.class(in = 1136) (out= 643)(deflated 43%)
adding: lab/core/wordcount/WordCount.class(in = 639) (out= 520)(deflated 18%)
adding: lab/core/wordcount/WordCount$.class(in = 8628) (out= 3480)(deflated 59%)
~~~

<br>

#### 5. wordcount application 실행

~~~bash
cd $SPARK_PROJECT
scala -classpath "$SPARK_PROJECT/wordcountscala.jar" lab.core.wordcount.WordCountScala
>>>> WordCountScala....
>>>> Line Count : 109
>>>> word count : (site,,1)
...
...

# Cat과 More 명령어로 정상적으로 결과가 나왔는지 확인
cat $SPARK_HOME/README.wordcount_scala | more
~~~

<br>

<br>

### Spark WordCount

#### 1. WordCountSpark class 생성

~~~python
vi $SPARK_BASE/projects/spark/src/lab/core/wordcount/WordCountSpark.scala
~~~



#### 2. 코드 작성

~~~scala
package lab.core.wordcount

import org.apache.spark.SparkContext

object WordCountSpark {

  def main(args: Array[String]): Unit = {
    println(">>>> WordCountSpark....")
    
    var input = "/spark/spark3/README.md"
    var output = "/spark/spark3/README.wordcount_spark"
    var delimiter = " "

    val sc = new SparkContext()
    val rdd = sc.textFile(input)
    println(">>>> Line Count : " + rdd.count())
    val rdd2 = rdd.flatMap(line => line.split(delimiter))
    val rdd3 = rdd2.map(word => (word, 1))
    val rdd4 = rdd3.groupBy(tuple => tuple._1)
    val rdd5 = rdd4.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.map(tuple => tuple._2)))
    val rdd6 = rdd5.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.reduce((v1, v2) => v1 + v2)))
    rdd6.foreach(word_count => println(">>>> word count : " + word_count))
    val rdd7 = rdd6.sortBy(tuple => -tuple._2)
    rdd7.foreach(word_count => println(">>>> word count (count desc) : " + word_count))

   
    val ordering = new Ordering[(String, Int)] {
      override def compare(a: (String, Int), b: (String, Int)) = {
        if((a._2 compare b._2) == 0) (a._1 compare b._1) else -(a._2 compare b._2)
      }
    }
    
    val rdd8 = rdd6.sortBy(tuple => tuple)(ordering, implicitly[scala.reflect.ClassTag[(String, Int)]])

    rdd8.foreach(word_count => println(">>>> word count (count desc, word asc) : " + word_count))

    rdd8.saveAsTextFile(output)

    sc.stop()
  }
}
~~~

<br>

#### 3. 컴파일 & jar로 아카이브

**diff :** Spark 전용으로 컴파일 할 경우에는 `spark-core` jar가 필요하다. 

~~~bash
cd $SPARK_BASE/projects/spark
scalac -classpath "$SPARK_HOME/jars/spark-core_2.12-3.2.1.jar" -d target/ src/lab/core/wordcount/WordCountSpark.scala


cd $SPARK_PROJECT/target && jar cvf ../wordcountspark.jar ./lab/core/wordcount/WordCountSpark*.class
added manifest
adding: lab/core/wordcount/WordCount$$anon$1.class(in = 4457) (out= 1493)(deflated 66%)
adding: lab/core/wordcount/WordCount$$anon$2.class(in = 1136) (out= 643)(deflated 43%)
adding: lab/core/wordcount/WordCount.class(in = 639) (out= 520)(deflated 18%)
adding: lab/core/wordcount/WordCount$.class(in = 8628) (out= 3480)(deflated 59%)
~~~

<br>

#### 4. Spark-submit으로 실행

`--master` 옵션을 주지 않으면 default로 `local` 에서 실행하게 된다. 주의하자.

~~~bash
cd $SPARK_PROJECT/target && $SPARK_HOME/bin/spark-submit --master spark://spark-master01:7177 --class lab.core.wordcount.WordCountSpark $SPARK_PROJECT/wordcountspark.jar
...
...
22/11/14 18:23:38 INFO MapOutputTrackerMasterEndpoint: Asked to send map output locations for shuffle 2 to 192.168.51.183:45308
22/11/14 18:23:38 INFO TaskSetManager: Finished task 1.0 in stage 12.0 (TID 17) in 35 ms on 1*.**.**.**3 (executor 2) (1/2)
22/11/14 18:23:38 INFO TaskSetManager: Finished task 0.0 in stage 12.0 (TID 16) in 55 ms on 1*.**.**.**4 (executor 0) (2/2)
~~~

<br>

#### 5. 결과 확인

`var output="/spark/spark3/README.wordcount_spark"`  에서 확인해보자.



##### local에서 실행했을 경우

input으로 넣었던 README.md가 **분산처리**되어 결과도 `local`에 **분산저장**이 된것을 확인 할 수 있다.

~~~bash
$SPARK_HOME/bin/spark-submit --class lab.core.wordcount.WordCountSpark $SPARK_PROJECT/wordcountspark.jar
ls $SPARK_HOME | grep README
README.md
README.wordcount_scala
README.wordcount_spark

cd $SPARK_HOME/README.wordcount_spark && ls -al
part-00000
.part-00000.crc
part-00001
.part-00001.crc
_SUCCESS
._SUCCESS.crc

cat part-* | more
~~~

<br>

##### application으로 실행했을 경우

아래와 같이 Spark Application으로 실행 할 경우, Spark Job이 병렬로 처리되어 각각 다른 노드에 파일이 분산 저장된것을 볼 수 있다.

~~~bash
ssh spark-worker01 ls $SPARK_HOME/README.wordcount_spark
_temporary

ssh spark-worker02 ls $SPARK_HOME/README.wordcount_spark
_temporary

ssh spark-worker03 ls $SPARK_HOME/README.wordcount_spark
ls: cannot access /spark/spark3/README.wordcount_spark: No such file or directory
~~~

<br>
