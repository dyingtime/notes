## 读取日志

---

### 获取指定`Pod`的日志

`'GET /api/v1/namespaces/{namespace}/pods/{pod}/log?container={container}'`

#### Query Parameters

| Parameter    | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| container    | 要读取的容器. 当`Pod`中只有一个容器时,可以不指定.            |
| follow       | 是否实时输出. 默认为 `false`.                                |
| limitBytes   | If set, the number of bytes to read from the server before terminating the log output. This may not display a complete final line of logging, and may return slightly more or slightly less than the specified limit. |
| pretty       | 若为 `true`, 则会格式化日志.                                 |
| previous     | Return previous terminated container logs. Defaults to false. |
| sinceSeconds | A relative time in seconds before the current time from which to show logs. If this value precedes the time a pod was started, only logs since the pod start will be returned. If this value is in the future, no logs will be returned. Only one of sinceSeconds or sinceTime may be specified. |
| tailLines    | 如果设置，则显示从日志末尾开始的行数。如果未指定，则从创建容器时显示日志，或者因为sinceSeconds或sinceTime而显示 |
| timestamps   | If true, add an RFC3339 or RFC3339Nano timestamp at the beginning of every line of log output. Defaults to false. |

