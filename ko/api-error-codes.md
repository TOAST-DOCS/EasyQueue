# API 오류 코드

**Data & Analytics > EasyQueue > API 오류 코드**

## 오류 코드

| 오류 코드 | 오류 메시지 | 설명 |
| --- | --- | --- |
| 0 | SUCCESS | 성공 |
| -1 | FAIL | 실패 |
| 4000100 | Topic name is already in use. | 이미 사용 중인 토픽 이름입니다. |
| 4000101 | Invalid partition count. Must be greater than existing. | 잘못된 파티션 수입니다. 기존 파티션 수보다 커야 합니다. |
| 4000102 | Invalid partition number. | 잘못된 파티션 번호입니다. |
| 4000103 | Topic is not in ACTIVE status. | 토픽이 ACTIVE 상태가 아닙니다. |
| 4000104 | Invalid offset. | 잘못된 오프셋입니다. |
| 4000300 | Invalid time range. | 잘못된 시간 범위입니다. |
| 4000301 | Statistics data retention period exceeded. (Maximum: 90 days) | 통계 데이터 보관 기간을 초과했습니다. (최대: 90일) |
| 4000302 | Invalid query duration. (Minimum: 60 seconds, Maximum: 30 days) | 잘못된 조회 기간입니다. (최소: 60초, 최대: 30일) |
| 4000303 | Invalid metrics type. | 잘못된 메트릭 타입입니다. |
| 4000400 | Topic quota exceeded. | 토픽 쿼터를 초과했습니다. |
| 4000401 | Topic partition quota exceeded. | 토픽 파티션 쿼터를 초과했습니다. |
| 4000402 | Project partition quota exceeded. | 프로젝트 파티션 쿼터를 초과했습니다. |
| 4030000 | Permission denied. | 권한이 거부되었습니다. |
| 4040100 | Topic not found. | 토픽을 찾을 수 없습니다. |
| 4040200 | Cluster not found. | 클러스터를 찾을 수 없습니다. |
| 4040400 | User not found. | 사용자를 찾을 수 없습니다. |
| 5000000 | Internal API fail. | 내부 API 호출에 실패했습니다. |
| 5000100 | Topic create fail. | 토픽 생성에 실패했습니다. |
| 5000101 | Topic update fail. | 토픽 수정에 실패했습니다. |
| 5000102 | Topic delete fail. | 토픽 삭제에 실패했습니다. |
| 5000103 | Get messages fail. | 메시지 조회에 실패했습니다. |
| 5000104 | Send messages fail. | 메시지 전송에 실패했습니다. |
| 5000105 | Get consumer groups fail. | 컨슈머 그룹 조회에 실패했습니다. |
