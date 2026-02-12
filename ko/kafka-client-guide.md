# 카프카 클라이언트 가이드

**Data & Analytics > EasyQueue > 카프카 클라이언트 가이드**

EasyQueue 서비스에 Kafka 클라이언트를 사용하여 메시지를 송수신하는 방법을 설명합니다.

## 사전 준비

### 인증 정보 확인

Kafka 클라이언트를 사용하여 EasyQueue 서비스에서 메시지를 송수신하기 위해서는 NHN Cloud 사용자 인증 정보가 필요합니다. 인증 정보는 SASL/OAUTHBEARER 방식으로 사용됩니다.

1. NHN Cloud 콘솔에서 **마이 페이지 > API 보안 설정**으로 이동합니다.
2. 토큰 타입이 **JWT**인 User Access Key를 생성합니다.
3. 발급받은 인증 정보는 Kafka 클라이언트 설정에서 다음과 같이 사용됩니다.

| 항목 | 설명 | Kafka 클라이언트 설정 |
|------|------|----------------------|
| User Access Key | 사용자 인증 키 | `sasl.oauthbearer.client.id` |
| Secret Access Key | 사용자 비밀 키 | `sasl.oauthbearer.client.secret` |
| 인증 서버 도메인 | OAuth 토큰 발급 URL | `sasl.oauthbearer.token.endpoint.url` |

자세한 내용은 [Public API > API 인증 방식 > User Access Key 토큰](/nhncloud/ko/public-api/user-access-key-token/)을 참고하세요.

### 인가 정보 확인

Kafka 클라이언트를 사용하여 메시지를 송수신하려면 사용자에게 **EasyQueue CLIENT** 가 포함된 권한이 필요합니다.

### 접속 정보 확인

EasyQueue 콘솔에서 다음 정보를 확인합니다.

| 항목 | 확인 방법 | 예시 |
|------|----------|------|
| AppKey | EasyQueue 서비스 활성화 후 콘솔 상단에서 확인 | |
| Bootstrap Servers | 토픽 생성 후 클라이언트 접속 정보에서 확인 | `{region}-{cluster}-bootstrap-easyqueue.nhncloudservice.com:30000` |
| Topic | 토픽 생성 후 토픽 이름 확인 | `{APP_KEY}.{TOPIC_NAME}` |

!!! tip "알아두기: Consumer Group 규칙"
    Consumer Group ID는 콘솔에서 별도로 제공되지 않으며, `{APP_KEY}.{GROUP_NAME}` 형식으로 지정해야 합니다. AppKey로 시작하지 않는 Consumer Group ID는 사용할 수 없습니다.

### SASL/OAUTHBEARER 설정

EasyQueue는 SASL/OAUTHBEARER 인증 방식을 사용합니다. Kafka 클라이언트에서 다음 설정이 필요합니다.

| 설정 항목 | 값 | 설명 |
|----------|-----|------|
| `security.protocol` | `SASL_SSL` | SSL을 통한 SASL 인증 |
| `sasl.mechanisms` | `OAUTHBEARER` | OAuth Bearer 토큰 인증 방식 |
| `sasl.oauthbearer.token.endpoint.url` | Token Endpoint URL | OAuth 토큰 발급 엔드포인트 |
| `sasl.oauthbearer.client.id` | User Access Key | 사용자 인증 키 |
| `sasl.oauthbearer.client.secret` | Secret Access Key | 사용자 비밀 키 |
| `sasl.oauthbearer.scope` | `appKey:{APP_KEY}` | EasyQueue 서비스 AppKey |

!!! danger "주의"
    OAuth 토큰은 User Access Key에서 설정한 토큰 유효 시간이 지나면 만료됩니다. 장시간 실행되는 Producer/Consumer는 토큰 자동 갱신 설정이 필수이며, 설정하지 않으면 토큰 만료 시 인증 오류로 연결이 끊어집니다.

## 언어별 클라이언트 예제

### Java

<details>
<summary><strong>의존성 설정</strong></summary>

**Maven(pom.xml)**

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>4.1.1</version>
    </dependency>

    <!-- OAuth SASL 인증을 위한 jose4j 라이브러리 -->
    <dependency>
        <groupId>org.bitbucket.b_c</groupId>
        <artifactId>jose4j</artifactId>
        <version>0.9.6</version>
    </dependency>

    <!-- OAuth 토큰 JSON 파싱을 위한 Jackson 라이브러리 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.18.2</version>
    </dependency>
</dependencies>
```

**Gradle(build.gradle)**

```groovy
implementation 'org.apache.kafka:kafka-clients:4.1.1'

// OAuth SASL 인증을 위한 jose4j 라이브러리
implementation 'org.bitbucket.b_c:jose4j:0.9.6'

// OAuth 토큰 JSON 파싱을 위한 Jackson 라이브러리
implementation 'com.fasterxml.jackson.core:jackson-databind:2.18.2'
```

</details>

<details>
<summary><strong>Producer 예제</strong></summary>

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
<summary><strong>Consumer 예제</strong></summary>

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

### Python

<details>
<summary><strong>의존성 설치</strong></summary>

```bash
pip install confluent-kafka==2.13.0
```

</details>

<details>
<summary><strong>Producer 예제</strong></summary>

```python
from confluent_kafka import Producer

# 접속 정보 설정
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
<summary><strong>Consumer 예제</strong></summary>

```python
from confluent_kafka import Consumer

# 접속 정보 설정
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

### JavaScript(Node.js)

<details>
<summary><strong>의존성 설치</strong></summary>

```bash
npm install @confluentinc/kafka-javascript@1.8.0
```


</details>

<details>
<summary><strong>Producer 예제</strong></summary>

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
<summary><strong>Consumer 예제</strong></summary>

```javascript
const Kafka = require("node-rdkafka");

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
};

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

### Go

<details>
<summary><strong>의존성 설치</strong></summary>

**go.mod 파일:**
```
module kafka-client-go

go 1.21

require github.com/confluentinc/confluent-kafka-go/v2 v2.6.1
```

**의존성 설치:**
```bash
go mod download
```

!!! tip "알아두기"
    `confluent-kafka-go`는 librdkafka에 의존합니다. 시스템에 librdkafka가 설치되어 있어야 합니다.


</details>

<details>
<summary><strong>Producer 예제</strong></summary>

```go
package main

import (
	"fmt"
	"log"

	"github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
	// 접속 정보 설정
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
<summary><strong>Consumer 예제</strong></summary>

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
	// 접속 정보 설정
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

## 문제 해결

### 연결 오류

**증상**: `Connection refused` 또는 `Broker not available` 오류

**해결 방법**:
- Bootstrap Servers 주소가 올바른지 확인합니다.
- 네트워크 방화벽에서 브로커 포트 30000~30008이 열려 있는지 확인합니다.

### 인증 오류

**증상**: `Authentication failed` 또는 `SASL authentication failed` 오류

**해결 방법**:
- User Access Key와 Secret Access Key가 올바른지 확인합니다.
- Token Endpoint URL이 올바른지 확인합니다.
- scope 설정에 AppKey가 올바르게 포함되어 있는지 확인합니다.
- 인증 정보에 권한이 부여되어 있는지 확인합니다.

### SSL 오류

**증상**: `SSL handshake failed` 또는 `Certificate verification failed` 오류

**해결 방법**:
- `security.protocol`이 `SASL_SSL`로 설정되어 있는지 확인합니다.

### 토픽 접근 오류

**증상**: `Topic authorization failed` 또는 `Unknown topic` 오류

**해결 방법**:
- 토픽 이름이 올바른지 확인합니다(형식: `{APP_KEY}.{TOPIC_NAME}`).
- 해당 토픽에 대한 접근 권한이 있는지 확인합니다.
- EasyQueue 콘솔에서 토픽이 생성되어 있는지 확인합니다.

### 컨슈머 그룹 오류

**증상**: `Group authorization failed` 오류

#### 해결 방법
- Consumer Group ID가 올바른 형식인지 확인합니다(형식: `{APP_KEY}.{GROUP_NAME}`).
- 해당 컨슈머 그룹에 대한 접근 권한이 있는지 확인합니다.

