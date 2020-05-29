---
title: PushOwl Architecture in a Nutshell
date: '2020-05-29'
tags:
  - architecture
  - scaling
---

## About PushOwl

PushOwl is a Business to Business (B2B) Software as a Service, e-Commerce marketing tool. B2B, since our users are online store owners. Our business model is recurring revenue model, where our clients pay us every month. We integrate the functionality of "Web Push Notification" in their online store. Currently PushOwl can be installed as a plugin on Shopify e-Commerce platform. We have done a good job of making that integration a self serve model and it takes less than 3 minutes to start getting their first web push subscribers. We provide our end customers a dashboard, where they can schedule their campaigns, see how their campaigns are performing and customise their automations. It is highly recommended that you watch [this video](https://www.youtube.com/watch?v=ggUY0Q4f5ok) by Google Chrome Developers, to understand the interaction between web push service and PushOwl system.
![PushOwl Dashboard](/images/pushowl-architecture/dashboard.png 'A view of PushOwl dashboard')

## The need for Scale and Elasticity

We send close to 7 Million web push notification every day. During holiday sales, such as BFCM, the number can go as high as 40 Million notifications per day. Our service workers, running in the browser of our subscribers, send delivery and click ping back to our API servers. Though, not all notifications are delivered (device might be off, user might have re-installed their browser etc.), our API server gets delivery and click ping for a significant portion of notifications dispatched. Hence, if we end up sending 1 Million notification in a span of 15 minutes, that is almost 300 thousand delivery pings and 15 thousand click pings sent to our API server, spread over 30 minutes.

![Push dispatch](/images/pushowl-architecture/push_dispatch.png 'An illustration of how push dispatch works')

This leads to bursty load on our API servers, which needs to elastically scale up to serve those bursts and scale down to save cost and resource. The workers dispatching the notifications should also scale up to dispatch notifications at a higher rate, when a merchant with huge subscriber base sends a campaign, and then scale down to save cost. We use Kubernetes APIs to scale our API server (Python Django) and async worker (Celery with RabbitMQ) resources.

![Kubernetes scaling PushOwl workers](/images/pushowl-architecture/kubernetes.png 'We use Kubernetes to autoscale our workers')

## Event Streaming

We also need to store the notification payload that we send to the subscribers for audit and debug. We can not store the notifications in our primary database, since it would mean increasing the size of the database table by almost 10 GB every day, and the index size to increase by 1 GB every day. We have a multi tenant PostgreSQL database (Citus Cloud) as our primary transactional database. Our merchants vary in terms of number of subscribers. Some merchants have subscribers in the order of millions, while some of them have subscribers in the order of hundreds. If we store the push notifications in our database, this might potentially lead to hot spots in our cluster during campaign dispatch. Hence, we use event streaming to publish all these events to Apache Kafka, and then use Kafka Connect to store all those Payload to S3. We then use AWS Athena (Presto) to make SQL queries on top of S3 data for our audit and debugging purposes.

![Publishing events to Kafka](/images/pushowl-architecture/kafka.png 'PushOwl uses Apache Kafka to publish events and Kafka Connect to save those events to S3')

## Reporting and Analytics

Reporting and analytics is the beating heart of any marketing SAAS tool. We put the power in the hands of our customers to slice and dice into how their subscriber growth, their campaigns, and their automations are performing. We also have a very fair revenue model, which is based on number of impressions consumed in any given calendar month. So, it is necessary that we store windowed aggregation of impressions (delivery) and clicks, which we receive from the service workers. We use kSQL (Kafka Streams) to do the aggregation. Kafka Streams take data from Kafka topic in micro batches, does the aggregations and stores the result back to a Kafka topic. It provides near real time aggregations, by any grouping that is needed by the application. kSQL provides makes it very easy to create streaming applications using a declarative interface. Watch [this excellent](https://www.youtube.com/watch?v=C-rUyWmRJSQ) video by Tim from Confluent explaining how kSQL works.

![kSQL aggregating events](/images/pushowl-architecture/ksql.png 'kSQL aggregates events and stores back to Kafka. Kafka connect saves those aggregations to Citus')

Below diagram sums up the entire backend stack of PushOwl, and shows various interaction between sub systems of PushOwl with third party cloud providers.

![PushOwl backend stack](/images/pushowl-architecture/po_stack.jpg 'PushOwl backend stack showing interactions between various sub systems')

Try PushOwl in one of your development stores.
