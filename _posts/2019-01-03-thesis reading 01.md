---
layout:     post
title:      "2014_Microservices: a definition of this new architectural term"
subtitle:   ""
date:       2019-01-03 16：19：00
author:     "Patrick"
header-img: ""
catalog:    true
tags:
    - 综述
    - 开山
    - 提出问题
    - 微服务
---

**基本信息：** 

- 2014|Microservices: a definition of this new architectural term
- Lewis J, Fowler M. Microservices: a definition of this new architectural term[J]. Mars, 2014.

**摘要：** 

	The term "Microservice Architecture" has sprung up over the last few years to describe a particular way of designing software applications as suites of independently deployable services. While there is no precise definition of this architectural style, there are certain common characteristics around organization around business capability, automated deployment, intelligence in the endpoints,and decentralized control of languages and data.

  
**结论：**

	One reasonable argument we've heard is that you shouldn't start with a microservices architecture. Instead begin with a monolith, keep it modular, and split it into microservices once the monolith becomes a problem. (Although this advice isn't ideal, since a good in-process interface is usually not a good service interface.)

**可参考的文献：** 
持续集成 Fowler M, Foemmel M. Continuous integration[J]. Thought-Works) http://www. thoughtworks. com/Continuous Integration. pdf, 2006, 122: 14.
持续交付 Humble J, Farley D. Continuous delivery: reliable software releases through build, test, and deployment automation[M]. Pearson Education, 2010.

**笔记：** 

首次比较明确的提出微服务的概念和具体的特征以及在实施微服务化中要注意的问题。
***微服务的概念***：

	We do not claim that the microservice style is novel or innovative, its roots go back at least to the design principles of Unix. But we do think that not enough people consider a microservice architecture and that many software developments would be better off if they used it.

微服务风格并不是新颖的或者创新的说法，他的起源可以追溯到Unix的设计原则，但是我们认为并没有足够的人意识到微服务的好处，并且通过使用微服务风格进行开发可以使软件开发变得更好。

***单体（monolithic）的问题***：

	Monolithic applications can be successful, but increasingly people are feeling frustrations with them - especially as more applications are being deployed to the cloud . Change cycles are tied together - a change made to a small part of the application, requires the entire monolith to be rebuilt and deployed. Over time it's often hard to keep a good modular structure, making it harder to keep changes that ought to only affect one module within that module. Scaling requires scaling of the entire application rather than parts of it that require greater resource.
	

也就是：扩展问题，不能局部扩展，只能整体扩展；更新周期较慢，要整体重新编译和发布；依赖问题难以选择最合适的架构。


***具体的characteristics***:
- **Componentization via Services**（通过服务实现组件）
	通过服务而不是库实现组件，We define **libraries** as components that are linked into a program and called using in memory function calls, while **services** are out-of-process components who communicate with a mechanism such as a web service request, or remote procedure call. 
	但是就会很难划分边界问题。
	
- Organized around Business Capabilities
- **Products not Projects**

	Microservice proponents tend to avoid this model, preferring instead the notion that a team should own a product over its full lifetime. A common inspiration for this is Amazon's notion of "you build, you run it" where a development team takes full responsibility for the software in production. This brings developers into day-to-day contact with how their software behaves in production and increases contact with their users, as they have to take on at least some of the support burden.
	单体也可以有这一条，但是服务的粒度越小，服务开发者和使用者建立联系就越容易。
- **Smart endpoints and dumb pipes**
	Restful 的http调用和RabbitMQ（messaging over a lightweight message bus）
- **Decentralized Governance**
	不同的系统选择不同语言等
- **Decentralized Data Management**
	多模型数据库（Polyglot Persistence），每个微服务最好有自己的数据库，要么是同一个数据库的不同instance,要么是完全不同的数据库系统。
	分布式之后不能像单体架构一样通过事物来保证一致性，只能采取最终一致性。具体的解决与业务实践有关，通常情况是允许一定程度的数据不一致为了快速响应，同时有一些逆转进程或者人工去处理不一致数据和错误，这些开销是值得的，因为修复一定程度的不一致错误比保证强一致性花费小。
- **Infrastructure Automation**
基础设施自动化，持续交付和持续集成，
- **Design for failure**
快速检测错误和自动恢复
- **Evolutionary Design**

***实施中要注意的问题***：
怎么划分，多大，边界怎么区分。

所以分为底层的功能，和上一层的功能，同层不能或者最好不要相互调用。















