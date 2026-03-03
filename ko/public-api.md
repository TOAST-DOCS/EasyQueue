# EasyQueue API v1.0 가이드

**Data & Analytics > EasyQueue > EasyQueue API v1.0 가이드**

## EasyQueue API 공통 정보

### 도메인

| 리전        | 도메인                                         |
|-----------|-----------------------------------------------|
| 한국(판교) 리전 | https://kr1-easyqueue.api.nhncloudservice.com |
| 한국(평촌) 리전 | https://kr2-easyqueue.api.nhncloudservice.com |

### 인증 및 권한

EasyQueue는 API 호출 시 인증/인가를 위해 Opaque형식의 User Access Key 토큰을 사용합니다. 
User Access Key 토큰은 User Access Key를 기반으로 발급되는 Bearer 타입의 일시적 액세스 토큰입니다. 
User Access Key 토큰 발급 및 사용에 대한 자세한 내용은 [User Access Key 토큰](/nhncloud/ko/public-api/user-access-key-token)을 참고하세요.

발급받은 토큰은 요청 Header에 포함해야 합니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-NHN-Authorization | Header | String | O | Public API로 발급 받은 Bearer 유형 토큰. <br> 헤더 값은 'Bearer {Access Token}' 형식으로 입력. |

프로젝트 멤버 역할에 따라 호출할 수 있는 API가 제한됩니다. EasyQueue ADMIN, EasyQueue VIEWER, EasyQueue CLIENT로 구분하여 권한을 부여할 수 있습니다.

* EasyQueue ADMIN 권한은 모든 기능을 사용 가능합니다.
* EasyQueue VIEWER 권한은 정보를 조회하는 기능만 사용 가능합니다.
* EasyQueue CLIENT 권한은 메시지를 송/수신하는 기능만 사용 가능합니다. EasyQueue VIEWER 권한을 포함합니다.

### 요청 공통 정보

#### Path Parameter

모든 API는 앱키를 Path Parameter로 지정해야 합니다.

예: /v1.0/appkeys/{appKey}/

| 이름 | 설명 |
| --- | --- |
| appKey | 콘솔에서 발급받은 앱키(Appkey) |

### 응답 공통 정보

모든 API 요청에 대해서 **200 OK**로 응답합니다. 자세한 응답 결과는 다음의 예와 같이 응답 본문의 헤더를 참고합니다.

<details>
  <summary><strong>성공 응답</strong></summary>

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
  <summary><strong>실패 응답</strong></summary>

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


| 이름 | 타입 | 설명 |
| --- | --- | --- |
| header               | Object  | 헤더 영역  |
| header.isSuccessful  | Boolean | 성공 여부  |
| header.resultCode    | Integer | 결과 코드  |
| header.resultMessage | String  | 결과 메시지 |


## Topic API

### 토픽 목록 조회

토픽 목록을 조회합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics |

**[QueryString Parameter]**

| 이름 | 타입 | 유효 범위 | 필수 여부 | 기본값 | 설명 |
|---|---|---|---|---|---|
| sortKey | String | TOPIC_NAME, <br>CREATED_AT, <br>UPDATED_AT | 필수 | CREATED_AT | 정렬 기준 키 값 <br>(TOPIC_NAME: 토픽 이름, <br>CREATED_AT: 생성 일시, <br>UPDATED_AT: 수정 일시) |
| topicIdList | List | 최대 100개 | 선택 |  | 필터: 토픽 ID 목록 |
| searchTopicName | String |  | 선택 |  | 필터: 토픽 이름(부분 일치) |
| sortDirection | String | DESC, ASC | 선택 | DESC | 정렬 방향(DESC: 내림차순, ASC: 오름차순) |
| page | Integer | 최소 1 | 선택 | 1 | 페이지 번호 |
| limit | Integer | 최소 1, 최대 3,000 | 선택 | 50 | 페이지당 항목 수 |

#### 응답

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
            "description": "테스트 토픽",
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

| 이름 | 타입 | 설명 |
|---|---|---|
| totalCount | Long | 전체 토픽 개수 |
| topicList | List | 토픽 목록 |
| topicList[0].topicId | String | 토픽 ID(읽기 전용) |
| topicList[0].topicName | String | 토픽 이름(수정 불가) |
| topicList[0].description | String | 토픽 설명 |
| topicList[0].bootstrapServer | String | Kafka 클러스터 접속 서버 주소(읽기 전용) |
| topicList[0].partitionCount | Integer | 토픽 파티션 수 |
| topicList[0].maxRetentionTimeMs | Long | 파티션별 로그 최대 저장 시간(milliseconds) |
| topicList[0].maxRetentionBytes | Long | 파티션별 로그 최대 저장 크기(bytes) |
| topicList[0].maxMessageBytes | Integer | 토픽 메시지의 최대 크기(bytes) |
| topicList[0].totalMessageCount | Long | 토픽 전체 메시지 수(읽기 전용) |
| topicList[0].consumerGroupCount | Integer | 컨슈머 그룹 수(읽기 전용) |
| topicList[0].createdAt | DateTime | 토픽 생성 일시(읽기 전용) |
| topicList[0].updatedAt | DateTime | 토픽 수정 일시(읽기 전용) |
| topicList[0].topicStatus | String | 토픽 상태(ACTIVE, ERROR, WARNING, DELETING)(읽기 전용) |

### 토픽 생성

토픽을 생성합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| POST | /v1.0/appkeys/{appKey}/topics |

**[Request Body]**

| 이름 | 타입 | 유효 범위 | 필수 여부 | 기본값 | 설명 |
|---|---|---|---|---|---|
| topic | Object |  | 필수 |  | 토픽 |
| topic.topicName | String | 최소 1자, 최대 50자<br>영문자, 숫자, '-' | 필수 |  | 토픽 이름(수정 불가) |
| topic.description | String | 최대 255자 | 선택 |  | 토픽 설명 |
| topic.partitionCount | Integer | 최소 1, 최대 16 | 필수 |  | 토픽 파티션 수 |
| topic.maxRetentionTimeMs | Long | 최소 3,600,000(1시간)<br>최대 1,209,600,000(14일) | 필수 |  | 파티션별 로그 최대 저장 시간(milliseconds) |
| topic.maxRetentionBytes | Long | 최소 1,073,741,824<br>최대 26,843,545,600 | 필수 |  | 파티션별 로그 최대 저장 크기(bytes) |
| topic.maxMessageBytes | Integer | 최소 1,024<br>최대 262,144 | 필수 |  | 토픽 메시지의 최대 크기(bytes) |

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
        "description": "테스트 토픽",
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

### 토픽 단건 조회

토픽 정보를 단건 조회합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| topicName | String | 필수 | 토픽 이름 |

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
        "description": "테스트 토픽",
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

### 토픽 수정

토픽을 수정합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| PUT | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| topicName | String | 필수 | 토픽 이름 |

**[Request Body]**

| 이름 | 타입 | 유효 범위 | 필수 여부 | 기본값 | 설명 |
|---|---|---|---|---|---|
| topic | Object |  | 필수 |  | 토픽 |
| topic.description | String | 최대 255자 | 선택 |  | 토픽 설명 |
| topic.partitionCount | Integer | 최소 1, 최대 16 | 필수 |  | 토픽 파티션 수<br>파티션 수는 상향 조정만 가능 |
| topic.maxRetentionTimeMs | Long | 최소 3,600,000(1시간)<br>최대 1,209,600,000(14일) | 필수 |  | 파티션별 로그 최대 저장 시간(milliseconds) |
| topic.maxRetentionBytes | Long | 최소 1,073,741,824<br>최대 26,843,545,600 | 필수 |  | 파티션별 로그 최대 저장 크기(bytes) |
| topic.maxMessageBytes | Integer | 최소 1,024<br>최대 262,144 | 필수 |  | 토픽 메시지의 최대 크기(bytes) |

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
        "description": "수정된 토픽 설명",
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

### 토픽 삭제

토픽을 삭제합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| DELETE | /v1.0/appkeys/{appKey}/topics/{topicName} |

**[Path Parameter]**

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| topicName | String | 필수 | 토픽 이름 |

#### 응답

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


### 파티션 목록 조회

토픽의 파티션 목록을 조회합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/partitions |

**[Path Parameter]**

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| topicName | String | 필수 | 토픽 이름 |

#### 응답

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

| 이름 | 타입 | 설명 |
|---|---|---|
| partitionList | List | 파티션 목록 |
| partitionList[0].partition | Integer | 파티션 번호 |
| partitionList[0].replicaCount | Integer | 파티션 레플리카 수 |
| partitionList[0].startOffset | Long | 파티션 시작 오프셋 |
| partitionList[0].endOffset | Long | 파티션 마지막 오프셋 |
| partitionList[0].messageCount | Long | 파티션 전체 메시지 수 |


### 컨슈머 그룹 목록 조회

토픽의 컨슈머 그룹 목록을 조회합니다.

#### 요청

[URI]

| 메서드 | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/topics/{topicName}/consumer-groups |

**[Path Parameter]**

| 이름 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| topicName | String | 필수 | 토픽 이름 |

#### 응답

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

| 이름 | 타입 | 설명 |
|---|---|---|
| consumerGroupList | List | 컨슈머 그룹 목록 |
| consumerGroupList[0].groupId | String | 컨슈머 그룹 ID |
| consumerGroupList[0].groupState | String | 컨슈머 그룹 상태<br>• Stable(정상): 모든 컨슈머가 정상적으로 동작<br>• Dead(삭제됨): 컨슈머 그룹이 삭제된 상태<br>• Empty(활성 멤버 없음): 컨슈머 그룹에 활성 컨슈머가 없음<br>• Assigning(파티션 할당 중): 컨슈머 그룹에 파티션 할당 중<br>• Reconciling(멤버 동기화 중): 할당된 파티션 조정 중<br>• PreparingRebalance(리밸런스 준비 중): 컨슈머 그룹 변경으로 인한 파티션 리밸런싱 준비 중<br>• CompletingRebalance(리밸런스 완료 중): 컨슈머 할당 후 동기화 진행 중<br>• Unknown(알 수 없음) |
| consumerGroupList[0].totalLag | Long | 컨슈머 그룹 전체 Lag |
| consumerGroupList[0].memberList | List | 컨슈머(멤버) ID 목록 |
| consumerGroupList[0].memberList[0].memberId | String | 컨슈머(멤버) ID |
| consumerGroupList[0].memberList[0].clientId | String | 클라이언트 ID |
| consumerGroupList[0].memberList[0].partitionList | List | 파티션 정보 목록 |
| consumerGroupList[0].memberList[0].partitionList[0].partition | Integer | 파티션 번호 |
| consumerGroupList[0].memberList[0].partitionList[0].currentOffset | Long | 현재 오프셋 |
| consumerGroupList[0].memberList[0].partitionList[0].endOffset | Long | 마지막 오프셋 |
| consumerGroupList[0].memberList[0].partitionList[0].lag | Long | Lag |


## Statistics API

### 통계 조회

Kafka 관련 통계를 조회합니다.

#### 요청

**[URI]**

| 메서드 | URI |
|---|---|
| GET | /v1.0/appkeys/{appKey}/statistics |

**[QueryString Parameter]**

| 이름 | 타입 | 유효 범위 | 필수 여부 | 기본값 | 설명 |
|---|---|---|---|---|---|
| metricsType | String | BYTE_IN_RATE, <br>BYTE_OUT_RATE, <br>MESSAGE_COUNT, <br>CONSUMER_LAG, <br>LOG_SIZE_PER_PARTITION | 필수 |  | 메트릭 타입<br>• BYTE_IN_RATE: 초당 수신 바이트 수<br>• BYTE_OUT_RATE: 초당 송신 바이트 수<br>• MESSAGE_COUNT: 메시지 수<br>• CONSUMER_LAG: 컨슈머 그룹 Lag(지연량)<br>• LOG_SIZE_PER_PARTITION: 파티션별 로그 크기 |
| topicName | String |  | 필수 |  | 토픽 이름 |
| startDateTime | DateTime | ISO 8601 형식, <br>90일 이내 | 필수 |  | 조회 시작 시간(예: 2023-10-27T19:30:00+09:00) |
| endDateTime | DateTime | ISO 8601 형식, <br>조회 기간 최소 60초, 최대 30일 | 필수 |  | 조회 마지막 시간(예: 2023-10-27T20:30:00+09:00) |

#### 응답

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

| 이름 | 타입 | 설명 |
|---|---|---|
| metricsType | String | 메트릭 타입 |
| step | Integer | 지표 간격(초 단위, 조회 기간에 따라 서버에서 지정) |
| data | List | 지표 데이터 목록 |
| data[0].labels | Object | 지표 Labels(BYTE_IN_RATE, BYTE_OUT_RATE, MESSAGE_COUNT: topic / CONSUMER_LAG: topic, consumergroup / LOG_SIZE_PER_PARTITION: topic, partition, broker) |
| data[0].values | List | 지표 데이터 값 목록 |
| data[0].values[0].timestamp | Long | 지표 시간(Unix timestamp - epoch seconds) |
| data[0].values[0].value | Double | 지표 값 |
