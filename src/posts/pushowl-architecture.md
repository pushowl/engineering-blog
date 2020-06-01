---
title: PushOwl Architecture in a Nutshell
date: '2020-06-01'
tags:
  - architecture
  - scaling
author:
  name: Abhishek Kumar
  link: https://www.linkedin.com/in/abhishek-kumar-a9b66418/
---

## About PushOwl

PushOwl is a B2B SaaS solution for ecommerce marketing. We integrate the functionality to collect push subscribers and send push notifications to the browser experience when visiting an online store. Currently, PushOwl can be installed as a plugin on the Shopify ecommerce platform.

We strive for a stellar user experience to make life as easy as possible for our clients, who are busy ecommerce merchants with a billion things to handle on their plate. A self-serve user interface, no code changes or manual integration steps required, meaningful default settings, and a dashboard to quickly analyze and optimize performance are some of the user-facing technical features that differentiate PushOwl in the market.

Web push and its use cases open up new ways of communications. It is highly recommended that you watch [this video](https://www.youtube.com/watch?v=ggUY0Q4f5ok) on the Google Chrome Developers channel, to understand the interaction between web push service and PushOwl system. Together with Shopify, we have also created a merchant-facing [course about using web push notifications](https://www.shopifycompass.com/learn/what-are-push-notifications-grow-your-business-using-this-marketing-strategy) and its impact on e-commerce marketing.

## The need for Scale and Elasticity

We send close to 7 million web push notifications every day. During holiday sales, such as BFCM (Black Friday Cyber Monday), the number of notifications sent can go as high as 40 million notifications per day. Our service workers, running in the browser of our subscribers, send the delivery and click pings back to our API servers. Though, not all notifications are delivered successfully (reasons are: device might be off, the subscriber might have re-installed their browser, etc.), our API servers get delivery and click pings for a significant portion of dispatched notifications. Hence, if we end up sending 1 million notifications within 15 minutes, that is almost 300 thousand delivery pings and 15 thousand click pings sent to our API servers, spread over 30 minutes.

![Push dispatch](/images/pushowl-architecture/push_dispatch.png 'An illustration of how push dispatch works')

This leads to a bursty load on our API servers, which needs to elastically scale up to serve those bursts and scale down to save cost and resources. The workers dispatching the notifications should also scale up to dispatch notifications at a higher rate when a merchant with a huge subscriber base sends a campaign, and then scale down to save cost. We use Kubernetes APIs to scale our API servers (Python Django) and async worker (Celery with RabbitMQ) resources.

![Kubernetes scaling PushOwl workers](/images/pushowl-architecture/kubernetes.png 'We use Kubernetes to autoscale our workers')

## Event Streaming

We also need to store the notification payload that we send to subscribers for audit and debug. We can not store the notifications in our primary database, since it would mean increasing the size of the database table by almost 10 GB every day and the index size to increase by 1 GB every day. We have a multi-tenant PostgreSQL database (Citus Cloud) as our primary transactional database. Our merchants vary in terms of the number of subscribers. Some merchants have subscribers in the order of millions, while some of them have subscribers in the order of hundreds. If we store the push notifications in our database, this might potentially lead to hot spots in our cluster during campaign dispatch. Hence, we use event streaming to publish all these events to Apache Kafka, and then use Kafka Connect to store all those payloads to S3. We then use AWS Athena (Presto) to make SQL queries on top of S3 data for our audit and debugging purposes.

![Publishing events to Kafka](/images/pushowl-architecture/kafka.png 'PushOwl uses Apache Kafka to publish events and Kafka Connect to save those events to S3')

## Reporting and Analytics

Reporting and analytics are the beating heart of any marketing tool. We put the power in the hands of our merchants to slice and dice into how their subscriber growth, their campaigns, and their automations are performing. While our pricing plans are based on "impressions" (successfully delivered push notifications = those notifications we record a delivery ping for), our merchants are mainly interested in the revenue that PushOwl generates for their online stores. Our revenue attribution model requires that we store windowed aggregation of impressions (delivery) and clicks, which we receive from the service workers. We use KSQL (Kafka Streams) to do the aggregation. Kafka Streams takes data from Kafka topic in micro-batches, does the aggregations, and stores the result back to a Kafka topic. It provides near real-time aggregations for any grouping that is needed. KSQL makes it very easy to create streaming applications using a declarative interface. Watch [this excellent video by Tim Berglund from Confluent](https://www.youtube.com/watch?v=C-rUyWmRJSQ) explaining how KSQL works.

![kSQL aggregating events](/images/pushowl-architecture/ksql.png 'kSQL aggregates events and stores back to Kafka. Kafka connect saves those aggregations to Citus')

Below diagram sums up the entire backend stack of PushOwl, and shows various interaction between sub systems of PushOwl with third party cloud providers.

![PushOwl backend stack](/images/pushowl-architecture/po_stack.jpg 'PushOwl backend stack showing interactions between various sub systems')

Do you want to see how it all comes together? You can easily test PushOwl by setting up a free [Shopify developer account](https://partners.shopify.com/signup/developer) and add [PushOwl](https://apps.shopify.com/pushowl) to your development store.

You can read more about development stores [here](https://help.shopify.com/en/partners/dashboard/development-stores).

By the way, we are hiring! [Join our team of misfits](https://angel.co/company/pushowl/jobs)!
