# Module 1: Spring Essentials Overview

## What is the Spring Framework?

Spring is an Open Source, Lightweight, DI (Dependency Injection) Container and Framework for building Java enterprise applications

Spring Framework is Open Source
* Spring binary and source code are freely available
* Apache 2 license
* Documentation is available at:
    - http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle

The Spring Framework is lightweight
* Spring applications do not requier a Java EE application server
    - But they can be deployed on one
* Spring is not invasive
    - Does not require you to extend the framework classes or implement framework interfaces for most usage
    - You write your code as POJOs
* Low overhead
    - Spring jars are relatively small

The Spring Framework Provides a DI Container
* Spring serves as a Dependency Injection (DI) container for your application objects
    - Your objects do not have to worry about finding / connecting to each other
* Spring instantiates and injects dependencies into your objects
* Spring also serves as a lifecyle manager

Spring Framework: More than Just a DI Container
* Enterprise applications must deal with a wide variety of technologies / architectures / deployment-platforms
    - Containerization, Cloud, Micro-services
    - JDBC, Transactions, ORM / JPA, NoSQL
    - Events, Streaming, Reactive, Messaging, JMS, AMQP, Tasks, Scheduling
    - Security, OAuth2, OpenID Connect
    - Monitoring, Observability
    - ...
* Spring provides framework classes, interfaces, and annotations to simplify working with lower-level technlogies
* Highly extensible and customizable

## The Dependency Injection (DI) Container

Goal of the Spring Framework
* Provide comprehensive infrastructural support for developing enterprise Java applications
    - Spring deals with the plumbing
    - You focus on solving the business domain problems
* key Principles
    - Don't Repeat Yourself (DRY)
    - Separation of Concerns
    - Convention over Configuration
    - Testability