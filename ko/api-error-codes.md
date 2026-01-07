## Data & Analytics > EasyQueue > API 오류 코드

|오류 코드|오류 메시지|설명|
|---|---|---|
|0|SUCCESS|성공|
|-1|FAIL|실패|
|4000104|Not Found Topic.|토픽을 찾을 수 없습니다.|
|4000105|topic name is already in use.|이미 사용 중인 토픽 이름입니다.|
|4000109|Invalid partition count. must be greater than existing.|잘못된 파티션 수입니다. 기존 파티션 수보다 커야 합니다.|
|4000110|Invalid partition number.|잘못된 파티션 번호입니다.|
|4000111|Topic is not in ACTIVE status. |토픽이 ACTIVE 상태가 아닙니다.|
|4000112|Invalid offset. |잘못된 오프셋입니다.|
|4000201|Not Found Cluster|클러스터를 찾을 수 없습니다.|
|4000301|Invalid time range.|잘못된 시간 범위입니다.|
|4000302|Statistics data retention period exceeded. (Maximum: 90 days)|통계 데이터 보관 기간을 초과했습니다. (최대: 90일)|
|4000303|Invalid query duration. (Minimum: 60 seconds, Maximum: 30 days)|잘못된 조회 기간입니다. (최소: 60초, 최대: 30일)|
|4000304|Invalid metrics type.|잘못된 메트릭 타입입니다.|
|4002001|Topic quota exceeded.|토픽 쿼터를 초과했습니다.|
|4002002|Topic partition quota exceeded.|토픽 파티션 쿼터를 초과했습니다.|
|4002003|Project partition quota exceeded.|프로젝트 파티션 쿼터를 초과했습니다.|
|4030001|Permission denied. |권한이 거부되었습니다.|
|4040001|Not found user. |사용자를 찾을 수 없습니다.|
|4040002|Not found topic. |토픽을 찾을 수 없습니다.|
|5000102|Topic create fail. |토픽 생성에 실패했습니다.|
|5000103|Topic update fail. |토픽 수정에 실패했습니다.|
|5000104|Topic delete fail. |토픽 삭제에 실패했습니다.|
|5000106|Get Messages fail. |메시지 조회에 실패했습니다.|
|5000107|Send Messages fail. |메시지 전송에 실패했습니다.|
|5000108|Get Consumer Groups fail. |컨슈머 그룹 조회에 실패했습니다.|
