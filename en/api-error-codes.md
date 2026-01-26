# API Error Code

**Data & Analytics > EasyQueue > API Error Code**

## Error Code

| Error code | Error message | Description |
| --- | --- | --- |
| 0 | SUCCESS | Success |
| -1 | FAIL | Failure |
| 4000100 | Topic name is already in use. | Topic name already in use. |
| 4000101 | Invalid partition count. Must be greater than existing. | Invalid partition count. New value must exceed the current count. |
| 4000102 | Invalid partition number. | Invalid partition number. |
| 4000103 | Topic is not in ACTIVE status. | Topic status is not ACTIVE. |
| 4000104 | Invalid offset. | Invalid offset. |
| 4000300 | Invalid time range. | Invalid time range. |
| 4000301 | Statistics data retention period exceeded. (Maximum: 90 days) | Data retention period exceeded. (max: 90days) |
| 4000302 | Invalid query duration. (Minimum: 60 seconds, Maximum: 30 days) | Invalid view period. (min: 60 seconds, max: 30 days) |
| 4000303 | Invalid metrics type. | Invalid metric type. |
| 4000400 | Topic quota exceeded. | Toic quota exceeded. |
| 4000401 | Topic partition quota exceeded. | Topic partition quota exceeded. |
| 4000402 | Project partition quota exceeded. | Project partition quota exceeded. |
| 4030000 | Permission denied. | Permission denied. |
| 4040100 | Topic not found. | Could not find topics. |
| 4040200 | Cluster not found. | Could not find clusters. |
| 4040400 | AppKey not found. | Could not find Appkeys. |
| 5000000 | Internal API fail. | Failed to call internal APIs. |
| 5000100 | Topic create fail. | Failed to create topics. |
| 5000101 | Topic update fail. | Failed to modify topics. |
| 5000102 | Topic delete fail. | Failed to delete topics. |
| 5000103 | Get messages fail. | Failed to view messages. |
| 5000104 | Send messages fail. | Failed to send messages. |
| 5000105 | Get consumer groups fail. | Failed to get consumer groups. |