# EasyQueue API v1.0 Guide

**Data & Analytics > EasyQueue > EasyQueue API v1.0 Guide**

## EasyQueue API Common Information

### Domain

| Region        | Domain                                         |
|-----------|-----------------------------------------------|
| Korea (Pangyo) Region | https://kr1-easyqueue.api.nhncloudservice.com |
| Korea (Pyeongchon) region | https://kr2-easyqueue.api.nhncloudservice.com |

### Authentication and Authorization

EasyQueue uses an opaque User Access Key token for authentication/authorization when making API calls.
A User Access Key token is a temporary access token of the Bearer type that is issued based on a User Access Key.
For more information about issuing and using User Access Key tokens, see [User Access Key Token](/nhncloud/ko/public-api/user-access-key-token).

The issued token must be included in the request header.

| Name | Type | Format | Required | Description |
| --- | --- | --- | --- | --- |
| X-NHN-Authorization | Header | String | O | Bearer type token issued by the Public API. <br> Enter the header value in the format 'Bearer {Access Token}'. |

Depending on your project member role, the APIs you can call are limited. You can grant permissions separately for EasyQueue ADMIN, EasyQueue VIEWER, and EasyQueue CLIENT.

* EasyQueue ADMIN permission enables full functionality.
* EasyQueue VIEWER permission only allows you to view information.
* EasyQueue CLIENT permission only enables the feature to send/receive messages and includes EasyQueue VIEWER permission.

### Request Common Information

#### Path Parameter

All APIs must specify the appkey as a path parameter.

e.g. /v1.0/appkeys/{appKey}/

| Name | Description |
| --- | --- |
| appKey | Appkey issued from the console |

### Response Common Information

The service responds with **200 OK** to all API requests. For detailed response results, refer to the header of the response body as in the following example.

<details>
  <summary><strong>Success response</strong></summary>

```
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```

</details>

<details>
  <summary><strong>Failure response</strong></summary>

```
{
    "header": {
        "isSuccessful": false,
        "resultCode": {resultCode},
        "resultMessage": "{errorMessage}"
    }
}
```

</details>


| Name | Type | Description |
| --- | --- | --- |
| header               | Object  | Header area  |
| header.isSuccessful  | Boolean | Successful or not  |
| header.resultCode    | Integer | Result code  |
| header.resultMessage | String  | Result message |


## Topic API

### View a Topic List

View a topic list.

#### Request

**[URI]**

| Method | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics |

**[QueryString Parameter]**

| Name | Type | Valid range | Required | Default | Description |
|---|---|---|---|---|---|
| sortKey | String | TOPIC_NAME, <br>CREATED_AT, <br>UPDATED_AT | Required | CREATED_AT | Sort Key <br>(TOPIC_NAME: Topic name, <br>CREATED_AT: Created at, <br>UPDATED_AT: Updated at) |
| topicIdList | List | Max. 100 items | Optional |  | Filter: Topic ID list |
| searchTopicName | String |  | Optional |  | Filter: Topic name (Prefix match) |
| sortDirection | String | DESC, ASC | Optional | DESC | Sort order (DESC: Descending, ASC: Ascending) |
| page | Integer | Min. 1 | Optional | 1 | Page No. |
| limit | Integer | Min. 1, Max. 3,000 | Optional | 50 | Number of items per page |

#### Response
 
**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "totalCount": 1,
    "topicList": [
        {
            "topicId": "topic-id-123",
            "topicName": "{appKey}.test-topic",
            "description": "test topic",
            "bootstrapServer": "kafka.example.com:9092",
            "partitionCount": 3,
            "maxRetentionTimeMs": 604800000,
            "maxRetentionBytes": 1073741824,
            "maxMessageBytes": 1048576,
            "totalMessageCount": 1000,
            "consumerGroupCount": 2,
            "createdAt": "2023-10-27T19:30:00+09:00",
            "updatedAt": "2023-10-27T19:30:00+09:00",
            "topicStatus": "ACTIVE"
        }
    ]
}
```

**[Field]**

| Name | Type | Description |
|---|---|---|
| totalCount | Long | Total number of topics |
| topicList | List | Topic list |
| topicList[0].topicId | String | Topic ID (read-only) |
| topicList[0].topicName | String | Topic name (cannot be modified) |
| topicList[0].description | String | Topic description |
| topicList[0].bootstrapServer | String | Kafka bootstrap servers (read only) |
| topicList[0].partitionCount | Integer | Number of topic partitions |
| topicList[0].maxRetentionTimeMs | Long | Maximum log retention period per partition (milliseconds) |
| topicList[0].maxRetentionBytes | Long | Maximum log retention size per partition (bytes) |
| topicList[0].maxMessageBytes | Integer | Maximum size of topic messages (bytes) |
| topicList[0].totalMessageCount | Long | Total number of messages in a topic (read-only) |
| topicList[0].consumerGroupCount | Integer | Number of consumer groups (read-only) |
| topicList[0].createdAt | DateTime | Topic created at (read only) |
| topicList[0].updatedAt | DateTime | Topic updated at (read-only) |
| topicList[0].topicStatus | String | Topic status (ACTIVE, ERROR, WARNING, and DELETING) (read-only) |

### Create a Topic

Create a topic.

#### Request

**[URI]**

| Method | URI |
|---|---|
| POST | /v1.0/appkeys/{appKey}/topics |

**[Request Body]**

| Name | Type | Valid range | Required | Default | Description |
|---|---|---|---|---|---|
| topic | Object |  | Required |  | Topic |
| topic.topicName | String | Min.1 character, Max. 50 characters<br>Alphabets, numbers, and '-' | Required |  | Topic name (cannot be modified) |
| topic.description | String | Max. 255 characters | Optional |  | Topic description |
| topic.partitionCount | Integer | Min. 1, Max. 16 | Required |  | Number of topic partitions |
| topic.maxRetentionTimeMs | Long | Min. 3,600,000 (1 hour)<br>Max. 1,209,600,000 (14 days) | Required |  | Maximum log retention period per partition (milliseconds) |
| topic.maxRetentionBytes | Long | Min. 1,024<br>Max. 26,843,545,600 | Required |  | Maximum log retention size per partition (bytes) |
| topic.maxMessageBytes | Integer | Min. 1,024<br>Max. 262,144 | Required |  | Maximum size of topic messages (bytes) |
 
#### 응답

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "topic": {
        "topicId": "topic-id-123",
        "topicName": "{appKey}.test-topic",
        "description": "test topic",
        "bootstrapServer": "kafka.example.com:9092",
        "partitionCount": 3,
        "maxRetentionTimeMs": 604800000,
        "maxRetentionBytes": 1073741824,
        "maxMessageBytes": 1048576,
        "totalMessageCount": 0,
        "consumerGroupCount": 0,
        "createdAt": "2023-10-27T19:30:00+09:00",
        "updatedAt": "2023-10-27T19:30:00+09:00",
        "topicStatus": "ACTIVE"
    }
}
```

### Single Topic Lookup

Retrieve details of a specific topic.

#### Request

**[URI]**

| Method | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| Name | Type | Required | Description |
|---|---|---|---|
| topicName | String | Required | Topic name |
 
#### 응답

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "topic": {
        "topicId": "topic-id-123",
        "topicName": "{appKey}.test-topic",
        "description": "test topic",
        "bootstrapServer": "kafka.example.com:9092",
        "partitionCount": 3,
        "maxRetentionTimeMs": 604800000,
        "maxRetentionBytes": 1073741824,
        "maxMessageBytes": 1048576,
        "totalMessageCount": 1000,
        "consumerGroupCount": 2,
        "createdAt": "2023-10-27T19:30:00+09:00",
        "updatedAt": "2023-10-27T19:30:00+09:00",
        "topicStatus": "ACTIVE"
    }
}
```

### Edit a Topic

Edit a topic.

#### Request

**[URI]**

| Method | URI |
|---|---|
| PUT | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| Name | Type | Required | Description |
|---|---|---|---|
| topicName | String | Required | Topic name |

**[Request Body]**

| Name | Type | Valid range | Required | Default | Description |
|---|---|---|---|---|---|
| topic | Object |  | Required |  | Topic |
| topic.description | String | Max. 255 characters | Optional |  | Topic description |
| topic.partitionCount | Integer | Min. 1, Max. 16 | Required |  | Number of topic partitions<br>The number of partitions can only be increased |
| topic.maxRetentionTimeMs | Long | Min. 3,600,000 (1 hour)<br>Max. 1,209,600,000 (14 days) | Required |  | Maximum log retention period per partition (milliseconds) |
| topic.maxRetentionBytes | Long | Min. 1,024<br>Max. 26,843,545,600 | Required |  | Maximum log retention size per partition (bytes) |
| topic.maxMessageBytes | Integer | Min. 1,024<br>Max. 262,144 | Required |  | Maximum size of topic messages (bytes) |

#### 응답

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "topic": {
        "topicId": "topic-id-123",
        "topicName": "{appKey}.test-topic",
        "description": "updated test topic",
        "bootstrapServer": "kafka.example.com:9092",
        "partitionCount": 5,
        "maxRetentionTimeMs": 1209600000,
        "maxRetentionBytes": 2147483648,
        "maxMessageBytes": 2097152,
        "totalMessageCount": 1000,
        "consumerGroupCount": 2,
        "createdAt": "2023-10-27T19:30:00+09:00",
        "updatedAt": "2023-10-27T20:00:00+09:00",
        "topicStatus": "ACTIVE"
    }
}
```
 
### Delete a Topic

Delete a topic.

#### Request

**[URI]**

| Method | URI |
|---|---|
| DELETE | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| Name | Type | Required | Description |
|---|---|---|---|
| topicName | String | Required | Topic name |

#### Response

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```

 
### View a Partition List

Retrieve a list of partitions for a topic.

#### Request

**[URI]**

| Method | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/partitions |

**[Path Parameter]**

| Name | Type | Required | Description |
|---|---|---|---|
| topicName | String | Required | Topic name |

#### Response

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "partitionList": [
        {
            "partition": 0,
            "replicaCount": 3,
            "startOffset": 0,
            "endOffset": 1000,
            "messageCount": 1000
        },
        {
            "partition": 1,
            "replicaCount": 3,
            "startOffset": 0,
            "endOffset": 950,
            "messageCount": 950
        }
    ]
}
```
 
**[Field]**

| Name | Type | Description |
|---|---|---|
| partitionList | List | Partition list |
| partitionList[0].partition | Integer | Partition number |
| partitionList[0].replicaCount | Integer | Number of partition replicas |
| partitionList[0].startOffset | Long | Partition start offset |
| partitionList[0].endOffset | Long | Partition end offset |
| partitionList[0].messageCount | Long | Partition-wide message count |


### Retrieve Consumer Group List

Retrieve a list of a topic's consumer groups.

#### Request

[URI]

| Method | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/consumer-groups |

**[Path Parameter]**

| Name | Type | Required | Description |
|---|---|---|---|
| topicName | String | Required | Topic name |

#### Response

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "consumerGroupList": [
        {
            "groupId": "consumer-group-1",
            "groupState": "Stable",
            "totalLag": 100,
            "memberList": [
                {
                    "memberId": "member-1",
                    "clientId": "client-1",
                    "partitionList": [
                        {
                            "partition": 0,
                            "currentOffset": 900,
                            "endOffset": 1000,
                            "lag": 100
                        }
                    ]
                }
            ]
        }
    ]
}
```
 
**[Field]**

| Name | Type | Description |
|---|---|---|
| consumerGroupList | List | Consumer group list |
| consumerGroupList[0].groupId | String | Consumer group ID |
| consumerGroupList[0].groupState | String | Consumer group status<br>• Stable: All consumers are operating normally<br>• Dead (deleted): The consumer group has been deleted.<br>• Empty (no active members): The consumer group has no active consumers<br>• Assigning: Assigning partitions to consumer groups.<br>• Reconciling (synchronizing members): Reconciling allocated partitions<br>• PreparingRebalance (rebalancing): Preparing to rebalance partition due to consumer group change.<br>• CompletingRebalance (completing rebalance): Synchronization in progress after consumer allocation.<br>• Unknown |
| consumerGroupList[0].totalLag | Long | Total consumer group lag |
| consumerGroupList[0].memberList | List | Consumer (member) ID list |
| consumerGroupList[0].memberList[0].memberId | String | Consumer (member) ID |
| consumerGroupList[0].memberList[0].clientId | String | Client ID |
| consumerGroupList[0].memberList[0].partitionList | List | Partition information list |
| consumerGroupList[0].memberList[0].partitionList[0].partition | Integer | Partition number |
| consumerGroupList[0].memberList[0].partitionList[0].currentOffset | Long | Current offset |
| consumerGroupList[0].memberList[0].partitionList[0].endOffset | Long | End offset |
| consumerGroupList[0].memberList[0].partitionList[0].lag | Long | Lag |


## Statistics API

### Query Statistics

View Kafka-related statistics.

#### Request

**[URI]**

| Method | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/statistics |

**[QueryString Parameter]**

| Name | Type | Valid range | Required | Default | Description |
|---|---|---|---|---|---|
| metricsType | String | BYTE_IN_RATE, <br>BYTE_OUT_RATE, <br>MESSAGE_COUNT, <br>CONSUMER_LAG, <br>LOG_SIZE_PER_PARTITION | Required |  | Metric type<br>• BYTE_IN_RATE: Bytes received per second<br>• BYTE_OUT_RATE: Bytes sent per second<br>• MESSAGE_COUNT: Message count<br>• CONSUMER_LAG: Consumer group lag (lag)<br>• LOG_SIZE_PER_PARTITION: Log size per partition |
| topicName | String |  | Required |  | Topic name |
| startDateTime | DateTime | ISO 8601 format | Required |  | Search start time (e.g., 2023-10-27T19:30:00+09:00) |
| endDateTime | DateTime | ISO 8601 format | Required |  | Search end time (e.g., 2023-10-27T20:30:00+09:00) |

#### Response

**[Response Body]**

```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "metricsType": "BYTE_IN_RATE",
    "step": 60,
    "data": [
        {
            "labels": {
                "topic": "test-topic"
            },
            "values": [
                {
                    "timestamp": 1698406200,
                    "value": 1024.5
                },
                {
                    "timestamp": 1698406260,
                    "value": 2048.3
                }
            ]
        }
    ]
}
```
 
**[Field]**

| Name | Type | Description |
|---|---|---|
| metricsType | String | Metric type |
| step | Integer | Metric interval (in seconds, specified by the server based on view period) |
| data | List | Metric data list |
| data[0].labels | Object | Metric Labels (BYTE_IN_RATE, BYTE_OUT_RATE, MESSAGE_COUNT: topic / CONSUMER_LAG: topic, consumergroup / LOG_SIZE_PER_PARTITION: topic, partition, broker) |
| data[0].values | List | List of metric data values |
| data[0].values[0].timestamp | Long | Metric time (Unix timestamp - epoch seconds) |
| data[0].values[0].value | Double | Metric value |
