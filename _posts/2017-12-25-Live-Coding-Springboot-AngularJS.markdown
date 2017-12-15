---
layout: post
title:  "10mn Live-coding SpringBoot + AngularJS"
date:   2017-12-15 23:30:00
categories: 
tags: live-coding java springboot AngularJS
---

<H1>10mn Live-coding SpringBoot + AngularJS</H1>

The lesson for students today was a basic "Todo" web application, storing todo-items(= message+ category) in a relational database.
 
The architecture is minimalist:
<ul>
<li>client-side = AngularJs</li>
<li>server-side = java springboot and Rest/Json endpoint</li>
</ul>

The goal was to have a minimalist solution, <b>Small is Beautifull</b> .. and The faster possible to code.
<BR/>
As a result, I have coded it in 9 minutes (after several trainings, first try in 22 minutes).

The source code is all contained in
<ul>
<li> pom.xml generated from spring initializer (not even edited)</li>
<li> 24 lines of html</li> 
<li> 26 lines of javascript</li> 
<li> 53 lines of java (4 classes and 1 interface)</li> 
</ul>


<H1>YouTube</H1>

I have published it on YouTube  (my first uploaded video on Youtub)
<A href="https://www.youtube.com/watch?v=582rSLIy_ng">youtube 10mn - livecoding java springboot angularjs</A>


<H1>Source Code</H1>


<H2>Java Source Code</H2>

{% highlight java %}
package com.example.demo;

import java.util.List;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

@Entity
class Todo {
	@Id @GeneratedValue public int id;
	public String msg;
	public String category;
}

interface TodoRepository extends JpaRepository<Todo,Integer> {}

@RestController
@RequestMapping(path="/api/todo")
class TodoController {
	
	@Autowired
	private TodoRepository repo;
	
	@GetMapping
	public List<Todo> list() {
		return repo.findAll();
	}
	
	@PostMapping
	public Todo create(@RequestBody Todo data) {
		return repo.save(data);
	}
	
}
{% endhighlight %}


<H2>Html Source Code</H2>

{% highlight html%}
<!doctype html>
<html ng-app="myapp">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.7/angular.min.js"></script>
    <script src="app.js"></script>
  </head>
  <body>
    <div ng-controller="TodoController as ctrl">
      <input type="text" ng-model="ctrl.category" placeholder="Enter a name here">
      <input type="text" ng-model="ctrl.msg" placeholder="Enter a name here">
      <hr>
      <h1>Todo: {{ctrl.category}}  {{ctrl.msg}}</h1>
      
      <button ng-click="ctrl.addTodo();">Add</button>
      <button ng-click="ctrl.loadTodos();">Load</button>
      
      <ul>
      <li ng-repeat="item in ctrl.todos">
      {{item.category}} {{item.msg}}
      </li>
      </ul>
    </div>
  </body>
</html>
{% endhighlight %}

<H2>AngularJS Javascript Source Code</H2>

{% highlight javascript%}
angular.module('myapp', [])
.controller("TodoController", function($http) {
	var self = this;
	self.msg = "";
	self.category = "";
	self.todos = [];

	self.loadTodos = function() {
		$http.get("/api/todo").then(function(resp, status) {
			self.todos = resp.data;
		}, function(err) {
			console.error("Error", err);
		});
	};
	
	self.addTodo = function() {
		var req = {
				msg: self.msg, category: self.category
		}
		$http.post("/api/todo", req).then(function(resp, status) {
			self.todos.push(resp.data);
		}, function(err) {
			console.error("Error", err);
		});
	};
});
{% endhighlight %}


<H2>Maven pom.xml Source Code</H2>

The maven pom.xml was not edited at all..it was generated from 
<A href="https://start.spring.io/">https://start.spring.io/</A>

{% highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.9.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
{% endhighlight %}


