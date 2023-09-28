---
title: Scala Functions
author: avokey
date: 2022-11-15 18:32:00 -0500
categories: [Data, Spark]
tags: [Programming, Scala, Functional Programming, Practical]
---
<br>

![Desktop View](/221115scala/default_post_image.png)

### Functions

#### Declaration

~~~scala
def functionName ([list of parameters]) : [return type]
~~~

#### Definition

~~~scala
def functionName ([list of parameters]) : [return type] = {
   function body
   return [expr]
}
~~~

##### Example 1 - Two Parameters

~~~scala
object add {
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b
      return sum
   }
}

scala> add.addInt(1,2)
res15: Int = 3
~~~

##### Example 2 - No Parameters

아래 예제를 보면 `Unit` 이란 Type을 return한다. 여기서 `Unit` 은 뭘까?

Java에서 `void` 와 비슷한 타입이다. 아무것도 return 하지 않지만, Scala의 Method는 반드시 값을 return해야한다.

따라서 아무것도 return 하고 싶지 않다면 저렇게 마지막에 `Unit`을 넣어줘야한다.

~~~scala
object Hello {
   def printMe( ) : Unit = {
      println("Hello, Scala!")
   }
}

scala> Hello.printMe()
Hello, Scala!
~~~

<br>

### Functions with Variable Arguments

#### Multiple Arguments

~~~scala
object Demo {
   def main(args: Array[String]) {
      printStrings("Hello", "Scala", "Python");
   }
   
   def printStrings( args:String* ) = {
      var i : Int = 0;
      
      for( arg <- args ){
         println("Arg value[" + i + "] = " + arg );
         i += 1;
      }
   }
}

scala> Demo.printStrings("Hello", "Scala", "Python")
Arg value[0] = Hello
Arg value[1] = Scala
Arg value[2] = Python
~~~