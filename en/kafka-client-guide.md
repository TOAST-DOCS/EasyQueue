<!-- pre-align:aligned sig=fbd2c85fde15 -->

<a id="kafka-client-guide"></a>
## Kafka Client Guide { #kafka-client-guide }

**Data & Analytics > EasyQueue > Kafka Client Guide**

This guide explains how to use the Kafka client to send and receive messages to and from the EasyQueue service.

<a id="prerequisites"></a>
## Prerequisites { #prerequisites }

<a id="verify-credentials"></a>
### Verify Credentials { #verify-credentials }

NHN Cloud user credentials are required to send and receive messages from the EasyQueue service using the Kafka client. The credential is used in the SASL/OAUTHBEARER method.

1. In the NHN Cloud console, go to **My Page > API Security Settings**.
2. Generate a User Access Key with **JWT** token type.
3. The issued credentials are used in the Kafka client settings as follows:

| Item | Description | Kafka Client Setup |
|------|------|----------------------|
| User Access Key | User authentication key | sasl.oauthbearer.client.id |
| Secret Access Key | User secret key | sasl.oauthbearer.client.secret |
| Authentication Server Domain | OAuth token issuance URL | sasl.oauthbearer.token.endpoint.url |

For more information, see [NHN Cloud > Public API > API Authentication Methods > User Access Key Token](/en/nhncloud/en/public-api/user-access-key-token/).

<a id="check-the-authorization-information"></a>
### Check the Authorization Information { #check-the-authorization-information }

To send and receive messages using the Kafka client, users need permissions that include **EasyQueue CLIENT**.

<a id="check-the-connection-information"></a>
### Check the Connection Information { #check-the-connection-information }

In the EasyQueue console, view the following information:

| Item | How to view | Example |
|------|----------|------|
| Appkey | Enable EasyQueue service and click **URL & Appkey** button on the top right of the console to confirm | |
| Bootstrap Servers | View client access information after creating a topic | {region}-{cluster}-bootstrap-easyqueue.nhncloudservice.com:30000 |
| Topic | Confirm topic name after topic creation | {APP_KEY}.{TOPIC_NAME} |

!!! tip "Note: Consumer Group Rules"
Consumer Group IDs are not provided separately in the console and must be specified in the format: {APP_KEY}.{GROUP_NAME}. Any Consumer Group ID that does not start with the AppKey cannot be used.

<a id="configure-sasloauthbearer"></a>
### Configure SASL/OAUTHBEARER { #configure-sasloauthbearer }

EasyQueue uses the SASL/OAUTHBEARER authentication method. The following settings are required on the Kafka client:

| Settings Item | Value | Description |
|----------|-----|------|
| security.protocol | SASL_SSL | SASL authentication over SSL |
| sasl.mechanisms | OAUTHBEARER | OAuth Bearer token authentication method |
| sasl.oauthbearer.token.endpoint.url | Token Endpoint URL | OAuth token issuing endpoints |
| sasl.oauthbearer.client.id | User Access Key | User authentication key |
| sasl.oauthbearer.client.secret | Secret Access Key | User secret key |
| sasl.oauthbearer.scope | appKey:{APP_KEY} | EasyQueue service app key |

!!! danger "Caution"
OAuth tokens expire based on the token lifetime configured in the User Access Key settings. For long-running producers and consumers, auto-refresh must be enabled. Otherwise, the connection will be terminated due to authentication errors upon token expiration.

<a id="client-examples-by-language"></a>
## Client Examples by Language { #client-examples-by-language }

<a id="java"></a>
### Java { #java }

<details>
<summary><strong>dependency configuration</strong></summary>

Maven(pom.xml)

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>4.1.1</version>
    </dependency>

    <!-- jose4j library for OAuth SASL authentication -->
    <dependency>
        <groupId>org.bitbucket.b_c</groupId>
        <artifactId>jose4j</artifactId>
        <version>0.9.6</version>
    </dependency>

    <!-- Jackson library for OAuth token JSON parsing  -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.18.2</version>
    </dependency>
</dependencies>
```

Gradle(build.gradle)

```groovy
implementation 'org.apache.kafka:kafka-clients:4.1.1'

// jose4j library for OAuth SASL authentication
implementation 'org.bitbucket.b_c:jose4j:0.9.6'

// Jackson library for OAuth token JSON parsing
implementation 'com.fasterxml.jackson.core:jackson-databind:2.18.2'
```

</details>

<details>
<summary><strong>Producer example</strong></summary>

```java
package com.example.easyqueue;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class EasyQueueProducer {

    public static void main(String[] args) {
        // Connection configuration
        String bootstrapServers = "{BOOTSTRAP_SERVERS}";
        String tokenEndpointUrl = "{TOKEN_ENDPOINT_URL}";
        String userAccessKey = "{USER_ACCESS_KEY}";
        String secretAccessKey = "{SECRET_ACCESS_KEY}";
        String appKey = "{APP_KEY}";
        String topic = appKey + ".{TOPIC_NAME}";

        // Kafka 4.x security: Allow OAuth token endpoint URL
        System.setProperty("org.apache.kafka.sasl.oauthbearer.allowed.urls", tokenEndpointUrl);

        // Producer configuration
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.CLIENT_ID_CONFIG, "java-producer");

        // SASL/OAUTHBEARER authentication configuration
        props.put("security.protocol", "SASL_SSL");
        props.put("sasl.mechanism", "OAUTHBEARER");
        props.put("sasl.login.callback.handler.class", 
            "org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler");
        props.put("sasl.oauthbearer.token.endpoint.url", tokenEndpointUrl);
        props.put("sasl.jaas.config", 
            "org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required " +
            "clientId=\"" + userAccessKey + "\" " +
            "clientSecret=\"" + secretAccessKey + "\" " +
            "scope=\"appKey:" + appKey + "\";");

        try (Producer<String, String> producer = new KafkaProducer<>(props)) {
            // Send messages
            for (int i = 0; i < 10; i++) {
                String key = "key-" + i;
                String value = "Hello EasyQueue! Message " + i;

                ProducerRecord<String, String> record = new ProducerRecord<>(topic, key, value);
                
                producer.send(record, (metadata, exception) -> {
                    if (exception != null) {
                        System.err.println("Failed to send message: " + exception.getMessage());
                    } else {
                        System.out.printf("Message sent successfully - Topic: %s, Partition: %d, Offset: %d%n",
                            metadata.topic(), metadata.partition(), metadata.offset());
                    }
                });
            }
            
            producer.flush();
            System.out.println("All messages sent successfully");
        } catch (Exception e) {
            System.err.println("Producer error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary><strong>Consumer example</strong></summary>

```java
package com.example.easyqueue;

import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class EasyQueueConsumer {

    public static void main(String[] args) {
        // Connection configuration
        String bootstrapServers = "{BOOTSTRAP_SERVERS}";
        String tokenEndpointUrl = "{TOKEN_ENDPOINT_URL}";
        String userAccessKey = "{USER_ACCESS_KEY}";
        String secretAccessKey = "{SECRET_ACCESS_KEY}";
        String appKey = "{APP_KEY}";
        String topic = appKey + ".{TOPIC_NAME}";
        String groupId = appKey + ".{GROUP_NAME}";

        // Kafka 4.x security: Allow OAuth token endpoint URL
        System.setProperty("org.apache.kafka.sasl.oauthbearer.allowed.urls", tokenEndpointUrl);

        // Consumer configuration
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // SASL/OAUTHBEARER authentication configuration
        props.put("security.protocol", "SASL_SSL");
        props.put("sasl.mechanism", "OAUTHBEARER");
        props.put("sasl.login.callback.handler.class", 
            "org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginCallbackHandler");
        props.put("sasl.oauthbearer.token.endpoint.url", tokenEndpointUrl);
        props.put("sasl.jaas.config", 
            "org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required " +
            "clientId=\"" + userAccessKey + "\" " +
            "clientSecret=\"" + secretAccessKey + "\" " +
            "scope=\"appKey:" + appKey + "\";");

        try (Consumer<String, String> consumer = new KafkaConsumer<>(props)) {
            consumer.subscribe(Collections.singletonList(topic));
            System.out.println("Started subscribing to topic: " + topic);

            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(10));
                
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Message received - Topic: %s, Partition: %d, Offset: %d, Key: %s, Value: %s%n",
                        record.topic(), record.partition(), record.offset(), record.key(), record.value());
                }
                
                consumer.commitSync();
            }
        } catch (Exception e) {
            System.err.println("Consumer error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

</details>

<a id="python"></a>
### Python { #python }

<details>
<summary><strong>Install dependencies</strong></summary>

```bash
pip install confluent-kafka==2.13.0
```

</details>

<details>
<summary><strong>Producer example</strong></summary>

```python
from confluent_kafka import Producer

# Connection settings
bootstrap_servers = "{BOOTSTRAP_SERVERS}"
token_endpoint_url = "{TOKEN_ENDPOINT_URL}"
user_access_key = "{USER_ACCESS_KEY}"
secret_access_key = "{SECRET_ACCESS_KEY}"
app_key = "{APP_KEY}"
topic = f"{app_key}.{{TOPIC_NAME}}"

# Producer configuration
config = {
    "bootstrap.servers": bootstrap_servers,
    "security.protocol": "SASL_SSL",
    "sasl.mechanisms": "OAUTHBEARER",
    "sasl.oauthbearer.method": "oidc",
    "sasl.oauthbearer.token.endpoint.url": token_endpoint_url,
    "sasl.oauthbearer.client.id": user_access_key,
    "sasl.oauthbearer.client.secret": secret_access_key,
    "sasl.oauthbearer.scope": f"appKey:{app_key}",
    "client.id": "py-producer"
}

def delivery_callback(err, msg):
    if err:
        print(f"Failed to send message: {err}")
    else:
        print(f"Message sent successfully - Topic: {msg.topic()}, Partition: {msg.partition()}, Offset: {msg.offset()}")

# Create producer and send messages
producer = Producer(config)

for i in range(10):
    key = f"key-{i}"
    value = f"Hello EasyQueue! Message {i}"
    producer.produce(topic, key=key, value=value, callback=delivery_callback)

producer.flush()
print("All messages sent successfully")
```

</details>

<details>
<summary><strong>Consumer example</strong></summary>

```python
from confluent_kafka import Consumer

# Connection settings
bootstrap_servers = "{BOOTSTRAP_SERVERS}"
token_endpoint_url = "{TOKEN_ENDPOINT_URL}"
user_access_key = "{USER_ACCESS_KEY}"
secret_access_key = "{SECRET_ACCESS_KEY}"
app_key = "{APP_KEY}"
topic = f"{app_key}.{{TOPIC_NAME}}"
group_id = f"{app_key}.{{GROUP_NAME}}"

# Consumer configuration
config = {
    "bootstrap.servers": bootstrap_servers,
    "security.protocol": "SASL_SSL",
    "sasl.mechanisms": "OAUTHBEARER",
    "sasl.oauthbearer.method": "oidc",
    "sasl.oauthbearer.token.endpoint.url": token_endpoint_url,
    "sasl.oauthbearer.client.id": user_access_key,
    "sasl.oauthbearer.client.secret": secret_access_key,
    "sasl.oauthbearer.scope": f"appKey:{app_key}",
    "group.id": group_id,
    "auto.offset.reset": "earliest"
}

# Create consumer and receive messages
consumer = Consumer(config)
consumer.subscribe([topic])
print(f"Started subscribing to topic: {topic}")

try:
    while True:
        msg = consumer.poll(timeout=10.0)
        
        if msg is None:
            continue
        if msg.error():
            print(f"Consumer error: {msg.error()}")
            continue
            
        print(f"Message received - Topic: {msg.topic()}, Partition: {msg.partition()}, "
              f"Offset: {msg.offset()}, Key: {msg.key()}, Value: {msg.value().decode('utf-8')}")
              
except KeyboardInterrupt:
    print("Consumer terminated")
finally:
    consumer.close()
```

</details>

<a id="javascriptnodejs"></a>
### JavaScript(Node.js) { #javascriptnodejs }

<details>
<summary><strong>Install dependencies</strong></summary>

```bash
npm install @confluentinc/kafka-javascript@1.8.0
```


</details>

<details>
<summary><strong>Producer example</strong></summary>

```javascript
const { Kafka } = require("@confluentinc/kafka-javascript").KafkaJS;

const bootstrapServers = "{BOOTSTRAP_SERVERS}";
const tokenEndpointUrl = "{TOKEN_ENDPOINT_URL}";
const userAccessKey = "{USER_ACCESS_KEY}";
const secretAccessKey = "{SECRET_ACCESS_KEY}";
const appKey = "{APP_KEY}";
const topic = `${appKey}.{TOPIC_NAME}`;

const kafka = new Kafka();
const producer = kafka.producer({
  "bootstrap.servers": bootstrapServers,
  "security.protocol": "sasl_ssl",
  "sasl.mechanisms": "OAUTHBEARER",
  "sasl.oauthbearer.method": "oidc",
  "sasl.oauthbearer.token.endpoint.url": tokenEndpointUrl,
  "sasl.oauthbearer.client.id": userAccessKey,
  "sasl.oauthbearer.client.secret": secretAccessKey,
  "sasl.oauthbearer.scope": `appKey:${appKey}`,
  "client.id": "js-producer",
});

async function run() {
  try {
    console.log("Connecting producer...");
    await producer.connect();
    console.log("Producer ready");

    // Send messages
    for (let i = 0; i < 10; i++) {
      const key = `key-${i}`;
      const value = `Hello EasyQueue! Message ${i}`;
      
      await producer.send({
        topic: topic,
        messages: [{ key, value }],
      });
      console.log(`Sent: ${value}`);
    }

    console.log("All messages sent successfully!");

  } catch (err) {
    console.error("Producer error:", err);
  } finally {
    console.log("Disconnecting producer...");
    await producer.disconnect();
  }
}

run();
```

</details>

<details>
<summary><strong>Consumer example</strong></summary>

```javascript
const { Kafka } = require("@confluentinc/kafka-javascript").KafkaJS;

const bootstrapServers = "{BOOTSTRAP_SERVERS}";
const tokenEndpointUrl = "{TOKEN_ENDPOINT_URL}";
const userAccessKey = "{USER_ACCESS_KEY}";
const secretAccessKey = "{SECRET_ACCESS_KEY}";
const appKey = "{APP_KEY}";
const topic = `${appKey}.{TOPIC_NAME}`;
const groupId = `${appKey}.{GROUP_NAME}`;

const kafka = new Kafka();
const consumer = kafka.consumer({
  "bootstrap.servers": bootstrapServers,
  "security.protocol": "sasl_ssl",
  "sasl.mechanisms": "OAUTHBEARER",
  "sasl.oauthbearer.method": "oidc",
  "sasl.oauthbearer.token.endpoint.url": tokenEndpointUrl,
  "sasl.oauthbearer.client.id": userAccessKey,
  "sasl.oauthbearer.client.secret": secretAccessKey,
  "sasl.oauthbearer.scope": `appKey:${appKey}`,
  "group.id": groupId,
  "auto.offset.reset": "earliest"
});

async function run() {
  try {
    console.log("Connecting consumer...");
    await consumer.connect();
    console.log("Consumer ready");

    await consumer.subscribe({ topics: [topic] });
    console.log(`Started subscribing to topic: ${topic}`);

    // Graceful shutdown
    process.on("SIGINT", async () => {
      console.log("\nDisconnecting consumer...");
      await consumer.disconnect();
      process.exit(0);
    });

    await consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        console.log(
          `Message received - Topic: ${topic}, Partition: ${partition}, ` +
          `Offset: ${message.offset}, Key: ${message.key?.toString()}, ` +
          `Value: ${message.value?.toString()}`
        );
      },
    });

  } catch (err) {
    console.error("Consumer error:", err);
    await consumer.disconnect();
  }
}

run();
```

</details>

<a id="go"></a>
### Go { #go }

<details>
<summary><strong>Install dependencies</strong></summary>

go.mod file
```
module kafka-client-go

go 1.21

require github.com/confluentinc/confluent-kafka-go/v2 v2.6.1
```

install dependencies
```bash
go mod download
```

`confluent-kafka-go` depends on `librdkafka`. You must have `librdkafka` installed on your system.


</details>

<details>
<summary><strong>Producer example</strong></summary>

```go
package main

import (
	"fmt"
	"log"

	"github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
	// Connection settings
	bootstrapServers := "{BOOTSTRAP_SERVERS}"
	tokenEndpointUrl := "{TOKEN_ENDPOINT_URL}"
	userAccessKey := "{USER_ACCESS_KEY}"
	secretAccessKey := "{SECRET_ACCESS_KEY}"
	appKey := "{APP_KEY}"
	topic := appKey + ".{TOPIC_NAME}"

	// Producer configuration
	config := kafka.ConfigMap{
		"bootstrap.servers":                   bootstrapServers,
		"security.protocol":                   "SASL_SSL",
		"sasl.mechanisms":                     "OAUTHBEARER",
		"sasl.oauthbearer.method":             "oidc",
		"sasl.oauthbearer.token.endpoint.url": tokenEndpointUrl,
		"sasl.oauthbearer.client.id":          userAccessKey,
		"sasl.oauthbearer.client.secret":      secretAccessKey,
		"sasl.oauthbearer.scope":              "appKey:" + appKey,
		"client.id": "go-producer",
	}

	producer, err := kafka.NewProducer(&config)
	if err != nil {
		log.Fatalf("Failed to create producer: %v", err)
	}
	defer producer.Close()

	// Handle delivery reports in a goroutine
	go func() {
		for e := range producer.Events() {
			switch ev := e.(type) {
			case *kafka.Message:
				if ev.TopicPartition.Error != nil {
					fmt.Printf("Failed to send message: %v\n", ev.TopicPartition.Error)
				} else {
					fmt.Printf("Message sent successfully - Topic: %s, Partition: %d, Offset: %d\n",
						*ev.TopicPartition.Topic, ev.TopicPartition.Partition, ev.TopicPartition.Offset)
				}
			}
		}
	}()

	// Send messages
	for i := 0; i < 10; i++ {
		key := fmt.Sprintf("key-%d", i)
		value := fmt.Sprintf("Hello EasyQueue! Message %d", i)

		err := producer.Produce(&kafka.Message{
			TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
			Key:            []byte(key),
			Value:          []byte(value),
		}, nil)

		if err != nil {
			fmt.Printf("Failed to send message request: %v\n", err)
		}
	}

	// Wait for all messages to be delivered
	producer.Flush(5000)
	fmt.Println("All messages sent successfully")
}
```

</details>

<details>
<summary><strong>Consumer example</strong></summary>

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"

	"github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
	// Connection settings
	bootstrapServers := "{BOOTSTRAP_SERVERS}"
	tokenEndpointUrl := "{TOKEN_ENDPOINT_URL}"
	userAccessKey := "{USER_ACCESS_KEY}"
	secretAccessKey := "{SECRET_ACCESS_KEY}"
	appKey := "{APP_KEY}"
	topic := appKey + ".{TOPIC_NAME}"
	groupId := appKey + ".{GROUP_NAME}"

	// Consumer configuration
	config := kafka.ConfigMap{
		"bootstrap.servers":                   bootstrapServers,
		"security.protocol":                   "SASL_SSL",
		"sasl.mechanisms":                     "OAUTHBEARER",
		"sasl.oauthbearer.method":             "oidc",
		"sasl.oauthbearer.token.endpoint.url": tokenEndpointUrl,
		"sasl.oauthbearer.client.id":          userAccessKey,
		"sasl.oauthbearer.client.secret":      secretAccessKey,
		"sasl.oauthbearer.scope":              "appKey:" + appKey,
		"group.id":                            groupId,
		"auto.offset.reset":                   "earliest",
	}

	consumer, err := kafka.NewConsumer(&config)
	if err != nil {
		log.Fatalf("Failed to create consumer: %v", err)
	}
	defer consumer.Close()

	err = consumer.SubscribeTopics([]string{topic}, nil)
	if err != nil {
		log.Fatalf("Failed to subscribe to topic: %v", err)
	}
	fmt.Printf("Started subscribing to topic: %s\n", topic)

	// Handle termination signals
	sigchan := make(chan os.Signal, 1)
	signal.Notify(sigchan, syscall.SIGINT, syscall.SIGTERM)

	run := true
	for run {
		select {
		case sig := <-sigchan:
			fmt.Printf("Received termination signal: %v\n", sig)
			run = false
		default:
			ev := consumer.Poll(10000)
			if ev == nil {
				continue
			}

			switch e := ev.(type) {
			case *kafka.Message:
				fmt.Printf("Message received - Topic: %s, Partition: %d, Offset: %d, Key: %s, Value: %s\n",
					*e.TopicPartition.Topic, e.TopicPartition.Partition, e.TopicPartition.Offset,
					string(e.Key), string(e.Value))
			case kafka.Error:
				fmt.Printf("Consumer error: %v\n", e)
				if e.Code() == kafka.ErrAllBrokersDown {
					run = false
				}
			}
		}
	}

	fmt.Println("Consumer terminated")
}
```

</details>

<a id="transaction-support"></a>
## Transaction Support { #transaction-support }
Kafka transactions process multiple messages as a single unit, ensuring that all messages either succeed or fail together.
Since consumers cannot read messages until a transaction is committed, incomplete data processing is prevented.

<a id="producer-settings"></a>
### Producer Settings { #producer-settings }

| Setting | Description |
|-----------|--------------------------------------------------|
| transactional.id | Identifier for the transactional producer. Must start with {APP_KEY}. |
| transaction.timeout.ms | Maximum duration for a transaction. Must be set to **300,000 ms (5 minutes)** or less. |


!!! tip "Note"
    `transactional.id` must start with {APP_KEY}. If configured without the appkey prefix, a permission error will occur on the broker.
    If the same `transactional.id` is used across multiple producer instances, the producer that starts later will forcibly terminate (fence) the existing producer.
    `transaction.timeout.ms` can be set to a maximum of 300,000 ms (5 minutes). If exceeded, an `InvalidTxnTimeoutException` error will occur.
    If a commit or abort is not completed within the configured time, the broker will automatically abort the transaction.


<a id="consumer-settings"></a>
### Consumer Settings { #consumer-settings }

| Setting | Description |
|-----------|---------------------------------------------------------|
| isolation.level | Set to read_committed to read only committed transaction messages. |



<a id="troubleshooting"></a>
## Troubleshooting { #troubleshooting }

<a id="connection-errors"></a>
### Connection Errors { #connection-errors }

<a id="connection-errors-symptoms"></a>
#### Symptoms


<a id="connection-errors-solution"></a>
#### Solution
- Verify that the Bootstrap Servers address is correct.
- Ensure that broker ports 30000 through 30008 are open on your network firewall.

<a id="authentication-errors"></a>
### Authentication Errors { #authentication-errors }

<a id="authentication-errors-symptoms"></a>
#### Symptoms


<a id="authentication-errors-solution"></a>
#### Solution
- Verify that the User Access Key and Secret Access Key are correct.
- Verify that the Token Endpoint URL is correct.
- Verify that the app key is included correctly in the scope settings.
- Verify that the credentials are authorized.

<a id="ssl-errors"></a>
### SSL Errors { #ssl-errors }

<a id="ssl-errors-symptoms"></a>
#### Symptoms


<a id="ssl-errors-solution"></a>
#### Solution
Verify that security.protocol is set to SASL_SSL.

<a id="topic-access-errors"></a>
### Topic Access Errors { #topic-access-errors }

<a id="topic-access-errors-symptoms"></a>
#### Symptoms


<a id="topic-access-errors-solution"></a>
#### Solution
- Verify that the topic name is correct (format: {APP_KEY}.{TOPIC_NAME}).
- Verify that you have access to the topic.
- In the EasyQueue console, verify that the topic has been created.

<a id="consumer-group-errors"></a>
### Consumer Group Errors { #consumer-group-errors }

<a id="consumer-group-errors-symptoms"></a>
#### Symptoms


<a id="consumer-group-errors-solution"></a>
#### Solution
- Verify that the consumer group ID is in the correct format (format: {APP_KEY}.{GROUP_NAME}).
- Verify that you have access to the consumer group.

<a id="transaction-timeout-error"></a>
### Transaction Timeout Error { #transaction-timeout-error }

<a id="transaction-timeout-error-symptom"></a>
#### Symptom
An InvalidTxnTimeoutException error occurs and the transaction cannot be started.

<a id="transaction-timeout-error-resolution"></a>
#### Resolution
- Verify that the `transaction.timeout.ms` value does not exceed 300,000 ms (5 minutes).
- Set the value to 300,000 or less.

<a id="transactional-id-permission-error"></a>
### Transactional ID Permission Error { #transactional-id-permission-error }

<a id="transactional-id-permission-error-symptom"></a>
#### Symptom
A TransactionalIdAuthorizationFailed error occurs and the transaction cannot be started.

<a id="transactional-id-permission-error-resolution"></a>
#### Resolution
- Verify that `transactional.id` starts with the appkey prefix (format: {APP_KEY}.{identifier}).
- If configured without the appkey prefix, the broker will reject the request.

<a id="producer-fencing-error"></a>
### Producer Fencing Error { #producer-fencing-error }

<a id="producer-fencing-error-symptom"></a>
#### Symptom
A ProducerFencedException error occurs and message transmission or commit fails.

<a id="producer-fencing-error-resolution"></a>
#### Resolution
- Check whether another producer instance using the same `transactional.id` is running.
- Use a unique `transactional.id` for each producer instance.

<a id="concurrent-transaction-conflict-error"></a>
### Concurrent Transaction Conflict Error { #concurrent-transaction-conflict-error }

<a id="concurrent-transaction-conflict-error-symptom"></a>
#### Symptom
A ConcurrentTransactionsException error occurs and a new transaction cannot be started.

<a id="concurrent-transaction-conflict-error-resolution"></a>
#### Resolution
- Start the next transaction only after the commit or abort of the previous transaction is complete.
- Multiple transactions cannot be opened simultaneously with the same `transactional.id`.

<a id="transaction-messages-not-being-read"></a>
### Transaction Messages Not Being Read { #transaction-messages-not-being-read }

<a id="transaction-messages-not-being-read-symptom"></a>
#### Symptom
Messages committed by the producer are not being read by the consumer.

<a id="transaction-messages-not-being-read-resolution"></a>
#### Resolution
- Verify that `isolation.level=read_committed` is configured on the consumer.


<a id="message-timestamp-error"></a>
### Message timestamp error { #message-timestamp-error }

<!-- TODO: translate body -->

<a id="message-timestamp-error-symptoms"></a>
#### Symptoms

<!-- TODO: translate body -->

<a id="message-timestamp-error-solution"></a>
#### Solution

<!-- TODO: translate body -->

<a id="transaction-delay-during-broker-maintenance"></a>
### Transaction Delay During Broker Maintenance { #transaction-delay-during-broker-maintenance }

<a id="transaction-delay-during-broker-maintenance-symptom"></a>
#### Symptom
COORDINATOR_LOAD_IN_PROGRESS or COORDINATOR_NOT_AVAILABLE errors occur temporarily during broker maintenance, causing a delay in starting transactions.

<a id="transaction-delay-during-broker-maintenance-resolution"></a>
#### Resolution
- Transactions may be temporarily delayed during broker maintenance. Recovery usually occurs automatically within a few seconds.
- Verify that retry settings (`retries`, `retry.backoff.ms`) are configured on the producer client.