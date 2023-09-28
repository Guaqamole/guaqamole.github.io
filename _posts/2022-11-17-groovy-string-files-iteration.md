---
title: String & Files & Iteration in Groovy
author: avokey
date: 2022-11-14 18:32:00 -0500
categories: [Infra, Scripting Language]
tags: [Groovy, Jenkins, Scripting Language, Syntax]
---

![Desktop View](/221117groovy/default_post_image.png){: width="972" height="589" }
_Fig 1. Apache Groovy._

**Jenkins**에서 API 자동화 테스트를 해야할 때 `Jenkins Pipeline`을 자주 사용한다. 젠킨스 파이프라인을 구성하기 위해서는 파이프라인 문법으로 작성해야하는데 이때 Groovy가 사용되므로 오늘 포스트에서 간단한 문법 정도만 작성하려고 한다.

[GroovyWebConsole](https://groovyconsole.appspot.com) 여기가 Groovy Online Compiler중에선 가장 빠른거같다.



## String

### Get firstline of string

~~~scala
def string = "hello!\nMyName Is\nJohn."

string.eachLine{ line, count ->
	def firstline = ""
	if(count==0){
		firstline = line
	}
	println(firstline)
}
~~~


### Substring & indexOf

~~~scala
status = "HTTP/1.1 200 OK"

int start = status.indexOf(' ')
int end = status.indexOf('OK')
String status_code = status.substring(start, end).trim()

println(status_code)
> 200
~~~

### Replace String

~~~scala
tmp = "/var/lib/conf/docker/knk/"

println("** FAILED! ** " + tmp.replace("$baseDir", "").replaceAll("/",""))
> knk
~~~

### Contains

~~~scala
str = "Hello World!"

println(str.contains("world"));
> false

println(str.contains("World"));
> true
~~~