# EasyQueue API v1.0 ガイド

**Data & Analytics > EasyQueue > EasyQueue API v1.0 ガイド**

## EasyQueue API共通情報

### ドメイン

| リージョン       | ドメイン                                        |
|-----------|-----------------------------------------------|
| 韓国(パンギョ) リージョン | https://kr1-easyqueue.api.nhncloudservice.com |
| 韓国(ピョンチョン) リージョン | https://kr2-easyqueue.api.nhncloudservice.com |

### 認証及び権限

EasyQueueはAPI呼び出し時の認証/認可のために、Opaque形式のUser Access Keyトークンを使用します。 
User Access Keyトークンは、User Access Keyに基づいて発行されるBearerタイプの一時的なアクセストークンです。 
User Access Keyトークンの発行及び使用に関する詳細は、[User Access Keyトークン](/nhncloud/ja/public-api/user-access-key-token)を参照してください。

発行されたトークンはリクエストヘッダに含める必要があります。

| 名前 | 種類 | 形式 | 必須 | 説明 |
| --- | --- | --- | --- | --- |
| X-NHN-Authorization | Header | String | O | パブリックAPIで発行されたBearerタイプのトークン。<br> ヘッダ値は「Bearer {Access Token}」形式で入力。 |

プロジェクトメンバーの役割に応じて、呼び出せるAPIが制限されます。EasyQueue ADMIN、EasyQueue VIEWER、EasyQueue CLIENTに区分して権限を付与できます。

* EasyQueue ADMIN権限は、全ての機能を使用できます。
* EasyQueue VIEWER権限は、情報を照会する機能のみを使用できます。
* EasyQueue CLIENT権限は、メッセージを送受信する機能のみを使用できます。EasyQueue VIEWER権限を含みます。

### リクエスト共通情報

#### Path Parameter

全てのAPIは、AppkeyをPath Parameterとして指定する必要があります。

例：/v1.0/appkeys/{appKey}/

| 名前 | 説明 |
| --- | --- |
| appKey | コンソールで発行されたAppkey |

### レスポンス共通情報

全てのAPIリクエストに対して**200 OK**でレスポンスします。詳細なレスポンス結果は、次の例のように応答本文のヘッダを参照してください。

<details>
  <summary><strong>成功レスポンス</strong></summary>

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
  <summary><strong>失敗レスポンス</strong></summary>

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


| 名前 | タイプ | 説明 |
| --- | --- | --- |
| header               | Object  | ヘッダ領域  |
| header.isSuccessful  | Boolean | 成功の有無  |
| header.resultCode    | Integer | 結果コード |
| header.resultMessage | String  | 結果メッセージ |


## Topic API

### トピック一覧照会

トピック一覧を照会します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics |

**[QueryString Parameter]**

| 名前 | タイプ | 有効範囲 | 必須の有無 | デフォルト値 | 説明 |
|---|---|---|---|---|---|
| sortKey | String | TOPIC_NAME, <br>CREATED_AT, <br>UPDATED_AT | 必須 | CREATED_AT | ソート基準キー値 <br>(TOPIC_NAME：トピック名、<br>CREATED_AT：作成日時、<br>UPDATED_AT：修正日時) |
| topicIdList | List | 最大100個 | 任意 |  | フィルタ：トピックID一覧 |
| searchTopicName | String |  | 任意 |  | フィルタ：トピック名(前方部分一致) |
| sortDirection | String | DESC, ASC | 任意 | DESC | ソート方向(DESC：降順、ASC：昇順) |
| page | Integer | 最小1 | 任意 | 1 | ページ番号 |
| limit | Integer | 最小1、最大3,000 | 任意 | 50 | ページあたりの項目数 |

#### レスポンス

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
            "description": "テストトピック",
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

| 名前 | タイプ | 説明 |
|---|---|---|
| totalCount | Long | 総トピック数 |
| topicList | List | トピック一覧 |
| topicList[0].topicId | String | トピックID(読み取り専用) |
| topicList[0].topicName | String | トピック名(修正不可) |
| topicList[0].description | String | トピック説明 |
| topicList[0].bootstrapServer | String | Kafkaクラスター接続サーバーアドレス(読み取り専用) |
| topicList[0].partitionCount | Integer | トピックのパーティション数 |
| topicList[0].maxRetentionTimeMs | Long | パーティション別のログ最大保存時間(milliseconds) |
| topicList[0].maxRetentionBytes | Long | パーティション別のログ最大保存サイズ(bytes) |
| topicList[0].maxMessageBytes | Integer | トピックメッセージの最大サイズ(bytes) |
| topicList[0].totalMessageCount | Long | トピックの総メッセージ数(読み取り専用) |
| topicList[0].consumerGroupCount | Integer | コンシューマーグループ数(読み取り専用) |
| topicList[0].createdAt | DateTime | トピック作成日時(読み取り専用) |
| topicList[0].updatedAt | DateTime | トピック修正日時(読み取り専用) |
| topicList[0].topicStatus | String | トピック状態(ACTIVE、ERROR、WARNING、DELETING)(読み取り専用) |

### トピック作成

トピックを作成します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| POST | /v1.0/appkeys/{appKey}/topics |

**[Request Body]**

| 名前 | タイプ | 有効範囲 | 必須の有無 | デフォルト値 | 説明 |
|---|---|---|---|---|---|
| topic | Object |  | 必須 |  | トピック |
| topic.topicName | String | 最小1文字、最大50文字<br>英字、数字、「-」 | 必須 |  | トピック名(修正不可) |
| topic.description | String | 最大255文字 | 任意 |  | トピック説明 |
| topic.partitionCount | Integer | 最小1、最大16 | 必須 |  | トピックのパーティション数 |
| topic.maxRetentionTimeMs | Long | 最小3,600,000(1時間)<br>最大1,209,600,000(14日) | 必須 |  | パーティション別のログ最大保存時間(milliseconds) |
| topic.maxRetentionBytes | Long | 最小1,073,741,824<br>最大26,843,545,600 | 必須 |  | パーティション別のログ最大保存サイズ(bytes) |
| topic.maxMessageBytes | Integer | 最小1,024<br>最大262,144 | 必須 |  | トピックメッセージの最大サイズ(bytes) |

#### レスポンス

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
        "description": "テストトピック",
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

### トピック単件照会

トピック情報を単件照会します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 名前 | タイプ | 必須の有無 | 説明 |
|---|---|---|---|
| topicName | String | 必須 | トピック名 |

#### レスポンス

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
        "description": "テストトピック",
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

### トピック修正

トピックを修正します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| PUT | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 名前 | タイプ | 必須の有無 | 説明 |
|---|---|---|---|
| topicName | String | 必須 | トピック名 |

**[Request Body]**

| 名前 | タイプ | 有効範囲 | 必須の有無 | デフォルト値 | 説明 |
|---|---|---|---|---|---|
| topic | Object |  | 必須 |  | トピック |
| topic.description | String | 最大255文字 | 任意 |  | トピック説明 |
| topic.partitionCount | Integer | 最小1、最大16 | 必須 |  | トピックのパーティション数<br>パーティション数は上方調整のみ可能 |
| topic.maxRetentionTimeMs | Long | 最小3,600,000(1時間)<br>最大1,209,600,000(14日) | 必須 |  | パーティション別のログ最大保存時間(milliseconds) |
| topic.maxRetentionBytes | Long | 最小1,073,741,824<br>最大26,843,545,600 | 必須 |  | パーティション別のログ最大保存サイズ(bytes) |
| topic.maxMessageBytes | Integer | 最小1,024<br>最大262,144 | 必須 |  | トピックメッセージの最大サイズ(bytes) |

#### レスポンス

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
        "description": "修正されたトピック説明",
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

### トピック削除

トピックを削除します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| DELETE | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 名前 | タイプ | 必須の有無 | 説明 |
|---|---|---|---|
| topicName | String | 必須 | トピック名 |

#### レスポンス

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


### パーティション一覧照会

トピックのパーティション一覧を照会します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/partitions |

**[Path Parameter]**

| 名前 | タイプ | 必須の有無 | 説明 |
|---|---|---|---|
| topicName | String | 必須 | トピック名 |

#### レスポンス

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

| 名前 | タイプ | 説明 |
|---|---|---|
| partitionList | List | パーティション一覧 |
| partitionList[0].partition | Integer | パーティション番号 |
| partitionList[0].replicaCount | Integer | パーティションのレプリカ数 |
| partitionList[0].startOffset | Long | パーティションの開始オフセット |
| partitionList[0].endOffset | Long | パーティションの最終オフセット |
| partitionList[0].messageCount | Long | パーティションの総メッセージ数 |


### コンシューマーグループ一覧照会

トピックのコンシューマーグループ一覧を照会します。

#### リクエスト

[URI]

| メソッド | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/consumer-groups |

**[Path Parameter]**

| 名前 | タイプ | 必須の有無 | 説明 |
|---|---|---|---|
| topicName | String | 必須 | トピック名 |

#### レスポンス

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

| 名前 | タイプ | 説明 |
|---|---|---|
| consumerGroupList | List | コンシューマーグループ一覧 |
| consumerGroupList[0].groupId | String | コンシューマーグループID |
| consumerGroupList[0].groupState | String | コンシューマーグループの状態<br>• Stable(正常)：全てのコンシューマーが正常に動作<br>• Dead(削除済み)：コンシューマーグループが削除された状態<br>• Empty(アクティブメンバーなし)：コンシューマーグループにアクティブなコンシューマーが存在しない<br>• Assigning(パーティション割り当て中)：コンシューマーグループにパーティションを割り当て中<br>• Reconciling(メンバー同期中)：割り当てられたパーティションを調整中<br>• PreparingRebalance(リバランス準備中)：コンシューマーグループの変更によるパーティションリバランスの準備中<br>• CompletingRebalance(リバランス完了中)：コンシューマー割り当て後の同期を進行中<br>• Unknown(不明) |
| consumerGroupList[0].totalLag | Long | コンシューマーグループの全体Lag |
| consumerGroupList[0].memberList | List | コンシューマー(メンバー)ID一覧 |
| consumerGroupList[0].memberList[0].memberId | String | コンシューマー(メンバー)ID |
| consumerGroupList[0].memberList[0].clientId | String | クライアントID |
| consumerGroupList[0].memberList[0].partitionList | List | パーティション情報一覧 |
| consumerGroupList[0].memberList[0].partitionList[0].partition | Integer | パーティション番号 |
| consumerGroupList[0].memberList[0].partitionList[0].currentOffset | Long | 現在のオフセット |
| consumerGroupList[0].memberList[0].partitionList[0].endOffset | Long | 最終オフセット |
| consumerGroupList[0].memberList[0].partitionList[0].lag | Long | Lag |


## Statistics API

### 統計照会

Kafkaに関する統計を照会します。

#### リクエスト

**[URI]**

| メソッド | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/statistics |

**[QueryString Parameter]**

| 名前 | タイプ | 有効範囲 | 必須の有無 | デフォルト値 | 説明 |
|---|---|---|---|---|---|
| metricsType | String | BYTE_IN_RATE, <br>BYTE_OUT_RATE, <br>MESSAGE_COUNT, <br>CONSUMER_LAG, <br>LOG_SIZE_PER_PARTITION | 必須 |  | メトリクスタイプ<br>• BYTE_IN_RATE：1秒あたりの受信バイト数<br>• BYTE_OUT_RATE：1秒あたりの送信バイト数<br>• MESSAGE_COUNT：メッセージ数<br>• CONSUMER_LAG：コンシューマーグループのLag(遅延量)<br>• LOG_SIZE_PER_PARTITION：パーティション別のログサイズ |
| topicName | String |  | 必須 |  | トピック名 |
| startDateTime | DateTime | ISO 8601形式、<br>90日以内 | 必須 |  | 照会開始時間(例：2023-10-27T19:30:00+09:00) |
| endDateTime | DateTime | ISO 8601形式、<br>照会期間：最小60秒、最大30日 | 必須 |  | 照会終了時間(例：2023-10-27T20:30:00+09:00) |

#### レスポンス

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

| 名前 | タイプ | 説明 |
|---|---|---|
| metricsType | String | メトリクスタイプ |
| step | Integer | 指標間隔(秒単位、照会期間に応じてサーバーで指定) |
| data | List | 指標データ一覧 |
| data[0].labels | Object | 指標のLabels(BYTE_IN_RATE、BYTE_OUT_RATE、MESSAGE_COUNT：topic / CONSUMER_LAG：topic、consumergroup / LOG_SIZE_PER_PARTITION：topic、partition、broker) |
| data[0].values | List | 指標データ値一覧 |
| data[0].values[0].timestamp | Long | 指標時間(Unix timestamp - epoch seconds) |
| data[0].values[0].value | Double | 指標値 |
