---
author: volodymyr
comments: true
date: 2017-02-08 00:03:00+00:00
layout: post
slug: benefits-of-aws-lambda
title: Benefits of the AWS Lambda and the serverless approach
categories:
- AWS
tags:
- AWS
- Lambda
- Serverless
- Cloud Computing
---

Thinking about switching to a serverless architecture? Or creating a new
mobile application and don't want to deal with server configuration and upgrades?
AWS Lambda is your friend then. But is it a silver bullet? I don't think you
believe in those. Read more to check some pros and cons I
discovered while working with it on a daily basis.

<!-- more -->

# Who is who?

AWS Lambda is a **pay-per-execution** service which allows to run arbitrary
code in the AWS environment and interact with the rest of the AWS
infrastructure.

You can think of it as a piece of code that will be executed when some
event occurs. The code itself can be written in: Java, NodeJS, Python,
and C#. The support of any other language can be achieved by using a transpiler,
though obviously, it may affect the performance.

There are many different event sources available that can trigger the
execution of the Lambda, including:

* **AWS API Gateway** &mdash; allows building REST APIs backed by Lambda functions, resulting in an almost perfect microservices architecture implementation
* **S3** &mdash; for processing changes of objects in S3 buckets
* **AWS IoT** &mdash; for handling events, coming from connected devices
* **AWS Alexa Skills API** &mdash; for implementing Alexa skills
* and others

The idea behind the AWS Lambda is that it's a super-efficient code that
handles the request and quickly returns the result. Pretty much like a UNIX tools
philosophy applied to the cloud computing.

# Areas in which AWS Lambda Just Rocks

Let's say you need to create a REST API for your mobile application or a SPA,
written in Angular/Vue.js/React.js. Nowadays, you don't even need to create
any boilerplate with Express/Spring Boot or Flask. Just use something like

[serverless](https://serverless.com/){:target="_blank"} or [Claudia.js](https://claudiajs.com/){:target="_blank"}
and you will get a highly-available, easy-to-use and easy-to-develop backend
in several seconds. Furthermore, you won't be paying for the idle time but only
for actual executions!

Another possible use-case is event handling for various AWS services, like S3.
A long time ago one would need to periodically poll the S3 to check whether a
new object appeared. Now it's enough to write a simple Lambda function and
configure an event source for it to easily handle any object modifications inside
some S3 bucket.

Possibilities are almost endless, but there are things to be considered:

* Lambda is good if you need a **scalable architecture** that serves many small requests. But remember there is a default limit of 100 concurrent executions per each Lambda. It can be increased by contacting the support.
* It's great for handling events from **S3** uploads, **SNS** and **IoT** notifications and other sources.
* Recently released [AWS Step Functions](https://aws.amazon.com/step-functions/){:target="_blank"} service also makes AWS Lambda a great tool for creating **complex**  multi-step **workflows**.
Right now the process of creation of new step functions is rather time-consuming, but the result is rewarding. Still, there are some things that can be improved in it.  


# Some stuff that can't be done with Lambda efficiently
Right tool for the right job they say. There are things that can't be
implemented efficiently using AWS Lambda because of it's very nature.

Let's take a look on the list of such things:

* **Long polling**. You can't actually create one,
taking into account AWS Lambda limit is 300 seconds per execution
and API Gateway limit is 30 seconds per execution.
* **API  with infrequent calls**. In the end, Lambda is just an EC2 instance. It's there for some time and "hot" calls are fast.
But if it's not being triggered for several minutes, the instance is disposed. Add a time needed for JVM to start and most likely you will want
to create some kind of a heartbeat to keep your function "warm".  Things are even worse if your Lambda is inside a VPC.
A known issue caused by the fact an additional network interface is initialized. Again, everything is good if you call the API at least once every several minutes
* No **data streaming** to the front-end. Websockets isn't something that can be implemented using AWS Lambda. For the same reason as the long-polling. But there are far better options for websockets than the Lambda, for example a Socket.io running on an BeanStalk instance.


# Does it worth trying?

Definitely yes, especially if your problem belongs to areas in which Lambda rocks!
We at [AgileVision](http://agilevision.pl/) have developed several completely serverless
solutions and Lambda saved a lot of time for us and our customers.

# Useful links

* [AWS Lambda](https://aws.amazon.com/lambda/) page
* [AWS StepFunctions](https://aws.amazon.com/step-functions/) page
