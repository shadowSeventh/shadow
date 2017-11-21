---
title: spring中scope概述
date: 2017-11-21 10:34:22
categories: 框架
tags: [Java,spring]
description: spring中scope是一个非常关键的概念，简单说就是对象在spring容器（IOC容器）中的生命周期，也可以理解为对象在spring容器中的创建方式。
---

## singleton （单一实例）
此取值时表明容器中创建时只存在一个实例，所有引用此bean都是单一实例。如同每个国家都有一个总统，国家的所有人共用此总统，而这个国家就是一个spring容器，总统就是spring创建的类的bean，国家中的人就是其它调用者，总统是一个表明其在spring中的scope为singleton，也就是单例模型。
此外，singleton类型的bean定义从容器启动到第一次被请求而实例化开始，只要容器不销毁或退出，该类型的bean的单一实例就会一直存活，典型单例模式，如同servlet在web容器中的生命周期。

## prototype
spring容器在进行输出prototype的bean对象时，会每次都重新生成一个新的对象给请求方，虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不在拥有当前对象的引用，请求方需要自己负责当前对象后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回请求方该对象的一个新的实例之后，就由这个对象“自生自灭”，最典型的体现就是spring与struts2进行整合时，要把action的scope改为prototype。
如同分苹果，将苹果的bean的scope属性声明为prototype，在每个人领取苹果的时候，我们都是发一个新的苹果给他，发完之后，别人爱怎么吃就怎么吃，爱什么时候吃什么时候吃，但是注意吃完要把苹果核扔到垃圾箱！对于那些不能共享使用的对象类型，应该将其定义的scope设为prototype。

## request
再次说明request，session和global session类型只实用于web程序，通常是和XmlWebApplicationContext共同使用。
`<bean id ="requestPrecessor" class="...RequestPrecessor"   scope="request" />`
Spring容器，即XmlWebApplicationContext 会为每个HTTP请求创建一个全新的RequestPrecessor对象，当请求结束后，该对象的生命周期即告结束，如同java web中request的生命周期。当同时有10个HTTP请求进来的时候，容器会分别针对这10个请求创建10个全新的RequestPrecessor实例，且他们相互之间互不干扰，简单来讲，request可以看做prototype的一种特例，除了场景更加具体之外，语意上差不多

## session
对于web应用来说，放到session中最普遍的就是用户的登录信息，对于这种放到session中的信息，我们可以使用如下形式的制定scope为session：
`<bean id ="userPreferences" class="...UserPreferences"   scope="session" />`
Spring容器会为每个独立的session创建属于自己的全新的UserPreferences实例，比request scope的bean会存活更长的时间，其他的方面没区别，如java web中session的生命周期。

## global session
`<bean id ="userPreferences" class="...UserPreferences"   scope="globalsession" />`
global session只有应用在基于porlet的web应用程序中才有意义，它映射到porlet的global范围的session，如果普通的servlet的web 应用中使用了这个scope，容器会把它作为普通的session的scope对待。