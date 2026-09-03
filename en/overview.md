<!-- pre-align:aligned sig=154b1ec7fbd6 -->

<a id="easyqueue-overview"></a>
## Data & Analytics > EasyQueue > Overview { #easyqueue-overview }

NHN Cloud EasyQueue is a fully managed message queue service that lets you instantly create and leverage topics via shared Kafka clusters. It eliminates the overhead of infrastructure setup and complex cluster management.
Users can easily build flexible data pipelines by asynchronously producing and consuming data between applications via Kafka topics.
Also, messages are distributed and replicated within the cluster to prevent data loss even during failures. This ensures reliable processing, as messages remain safely stored in the queue even if the receiving application is temporarily unavailable.

<a id="service-access-path"></a>
## Service Access Path { #service-access-path }

You can access the service by selecting **Data & Analytics > EasyQueue** in the NHN Cloud console.

<a id="main-features"></a>
## Main Features { #main-features }

<a id="topic-creation-and-lifecycle-management"></a>
### Topic Creation and Lifecycle Management { #topic-creation-and-lifecycle-management }

The web console allows you to easily create and delete topics with a click, no complicated commands or configuration files.
You can flexibly configure topic-specific policies, such as data retention periods and maximum message sizes, to align with your service requirements.
Furthermore, you can optimize processing performance by adjusting the number of partitions to match your traffic scale.

<a id="monitoring-dashboard"></a>
### Monitoring Dashboard { #monitoring-dashboard }

You can see metrics like data inflow by topic, number of messages, and more.

<a id="message-sendreceive-test"></a>
### Message Send/Receive Test { #message-sendreceive-test }

You can issue test messages directly from within the console, without having to write a separate client application or code.
You can view the messages loaded in a specific topic.
This can be used as a debugging tool to check communication status during the initial integration phase or to validate data formats.

<a id="consumer-group-monitoring"></a>
### Consumer Group Monitoring { #consumer-group-monitoring }

You can view consumer groups and a list of consumers per group.
You can check the processing status of consumer groups at a glance, and view lag numbers to quickly understand processing performance.

<a id="how-easyqueue-works"></a>
## How EasyQueue Works { #how-easyqueue-works }

![[Figure 1] How EasyQueue works](http://static.toastoven.net/prod_easyqueue/15_data&analytics_easyqueue_img_en.png)

➊ Message publishing: Producers send data to specific topics in EasyQueue.
➋ Message queuing: Received messages are stored distributed within the EasyQueue cluster, ensuring that they are not lost during high volumes of traffic.
➌ Message subscription: Consumers fetch queued messages and process the data according to their business logic.

<a id="service-terms"></a>
## Service Terms { #service-terms }

| Term | Description |
| --- | --- |
| Message Queue | A communication technique used to exchange data between systems in a process or program in a distributed environment |
| Broker | A server that receives messages from producers, stores them, and serves them to consumers |
| Topic | A grouping unit for messages on related topics |
| Partition | Data is split and stored across multiple partitions based on the number of partitions set in the topic |
| Producer | An entity that sends messages to a topic |
| Consumer | An entity that subscribes to a specific topic to receive and consume messages |
| Consumer Group | A group of multiple consumers subscribed to the same topic |

<a id="table-of-contents"></a>
## Table of Contents { #table-of-contents }

* [Console Guide](./console-guide/)
* [API Guide](./public-api/)
* [API Error Codes](./api-error-codes/)
* [Release Notes](./release-notes/)
