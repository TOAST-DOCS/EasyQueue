# Kafkaクライアントガイド

**Data & Analytics > EasyQueue > Kafkaクライアントガイド**

EasyQueueサービスでKafkaクライアントを使用して、メッセージを送受信する方法を説明します。

## 事前準備

### 認証情報の確認

Kafkaクライアントを使用してEasyQueueサービスでメッセージを送受信するには、NHN Cloudのユーザー認証情報が必要です。認証情報はSASL/OAUTHBEARER方式で使用されます。

1. NHN Cloudコンソールで**マイページ > APIセキュリティ設定**に移動します。
2. トークンタイプが**JWT**のUser Access Keyを作成します。
3. 発行された認証情報は、Kafkaクライアントの設定で次のように使用されます。

| 項目 | 説明 | Kafkaクライアント設定 |
|------|------|----------------------|
| User Access Key | ユーザー認証キー | sasl.oauthbearer.client.id |
| Secret Access Key | ユーザーシークレットキー | sasl.oauthbearer.client.secret |
| 認証サーバードメイン | OAuthトークン発行URL | sasl.oauthbearer.token.endpoint.url |

詳細は[パブリックAPI > API認証方式 > User Access Keyトークン](/nhncloud/ja/public-api/user-access-key-token/)を参照してください。

### 認可情報の確認

Kafkaクライアントを使用してメッセージを送受信するには、ユーザーに**EasyQueue CLIENT**が含まれる権限が必要です。

### 接続情報の確認

EasyQueueコンソールで以下の情報を確認します。

| 項目 | 確認方法 | 例 |
|------|----------|------|
| Appkey | EasyQueueサービスを有効化した後、コンソール右上の**URL & Appkey**ボタンをクリックして確認 | |
| Bootstrap Servers | トピック作成後、クライアント接続情報で確認 | {region}-{cluster}-bootstrap-easyqueue.nhncloudservice.com:30000 |
| Topic | トピック作成後、トピック名を確認 | {APP_KEY}.{TOPIC_NAME} |

!!! tip "参考：Consumer Group規則"
    Consumer Group IDはコンソールで別途提供されず、{APP_KEY}.{GROUP_NAME}の形式で指定する必要があります。Appkeyで始まらないConsumer Group IDは使用できません。

### SASL/OAUTHBEARER設定

EasyQueueはSASL/OAUTHBEARER認証方式を使用します。Kafkaクライアントで以下の設定が必要です。

| 設定項目 | 値 | 説明 |
|----------|-----|------|
| security.protocol | SASL_SSL | SSL経由のSASL認証 |
| sasl.mechanisms | OAUTHBEARER | OAuth Bearerトークン認証方式 |
| sasl.oauthbearer.token.endpoint.url | Token Endpoint URL | OAuthトークン発行エンドポイント |
| sasl.oauthbearer.client.id | User Access Key | ユーザー認証キー |
| sasl.oauthbearer.client.secret | Secret Access Key | ユーザーシークレットキー |
| sasl.oauthbearer.scope | appKey:{APP_KEY} | EasyQueueサービスのAppkey |

!!! danger "注意"
    OAuthトークンは、User Access Keyで設定したトークンの有効時間が経過すると期限切れになります。長時間実行されるプロデューサー/コンシューマーは、トークンの自動更新設定が必須であり、設定しない場合はトークンの期限切れ時に認証エラーで接続が切断されます。

## 言語別クライアント例

### Java

<details>
<summary><strong>依存関係の設定</strong></summary>

**Maven(pom.xml)**

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>4.1.1</version>
    </dependency>

    <!-- OAuth SASL認証のためのjose4jライブラリ -->
    <dependency>
        <groupId>org.bitbucket.b_c</groupId>
        <artifactId>jose4j</artifactId>
        <version>0.9.6</version>
    </dependency>

    <!-- OAuthトークンJSONパースのためのJacksonライブラリ -->
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

// OAuth SASL認証のためのjose4jライブラリ
implementation 'org.bitbucket.b_c:jose4j:0.9.6'

// OAuthトークンJSONパースのためのJacksonライブラリ
implementation 'com.fasterxml.jackson.core:jackson-databind:2.18.2'
```

</details>

<details>
<summary><strong>Producer例</strong></summary>

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
<summary><strong>Consumer例</strong></summary>

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
<summary><strong>依存関係のインストール</strong></summary>

```bash
pip install confluent-kafka==2.13.0
```

</details>

<details>
<summary><strong>Producer例</strong></summary>

```python
from confluent_kafka import Producer

# 接続情報の設定
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
<summary><strong>Consumer例</strong></summary>

```python
from confluent_kafka import Consumer

# 接続情報の設定
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
<summary><strong>依存関係のインストール</strong></summary>

```bash
npm install @confluentinc/kafka-javascript@1.8.0
```


</details>

<details>
<summary><strong>Producer例</strong></summary>

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
<summary><strong>Consumer例</strong></summary>

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
<summary><strong>依存関係のインストール</strong></summary>

**go.modファイル:**
```
module kafka-client-go

go 1.21

require github.com/confluentinc/confluent-kafka-go/v2 v2.6.1
```

**依存関係のインストール:**
```bash
go mod download
```

!!! tip 「ポイント」
    confluent-kafka-goはlibrdkafkaに依存します。システムにlibrdkafkaがインストールされている必要があります。


</details>

<details>
<summary><strong>Producer例</strong></summary>

```go
package main

import (
	"fmt"
	"log"

	"github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
	// 接続情報の設定
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
<summary><strong>Consumer例</strong></summary>

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
	// 接続情報の設定
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

## トラブルシューティング

### 接続エラー

#### 症状
Connection refused または Broker not available エラー

#### 解決方法
- Bootstrap Serversのアドレスが正しいか確認します。
- ネットワークのファイアウォールでブローカーのポート30000～30008が開いているか確認します。

### 認証エラー

#### 症状
Authentication failed または SASL authentication failed エラー

#### 解決方法
- User Access KeyとSecret Access Keyが正しいか確認します。
- Token Endpoint URLが正しいか確認します。
- scopeの設定にAppkeyが正しく含まれているか確認します。
- 認証情報に権限が付与されているか確認します。

### SSLエラー

#### 症状
SSL handshake failed または Certificate verification failed エラー

#### 解決方法
security.protocolがSASL_SSLに設定されているか確認します。

### トピックアクセスエラー

#### 症状
Topic authorization failed または Unknown topic エラー

#### 解決方法
- トピック名が正しいか確認します(形式: {APP_KEY}.{TOPIC_NAME})。
- 該当トピックに対するアクセス権限があるか確認します。
- EasyQueueコンソールでトピックが作成されているか確認します。

### コンシューマーグループエラー

#### 症状
Group authorization failed

#### 解決方法
- Consumer Group IDが正しい形式か確認します(形式: {APP_KEY}.{GROUP_NAME})。
- 該当コンシューマーグループに対するアクセス権限があるか確認します。
