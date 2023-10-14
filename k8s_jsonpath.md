# 何為要學 JsonPath
說明如何使用 jsonpath 來解析 kubeneter 中 pod、node ... 的資料，在考題中常會需要列出 node os 版本，所 pod 使用的 image 或 ip

# JsonPath
JsonPaht 是用來解析 json 格式的語法，可以滿足你想要獲取的 json 內容。
* JsonPath 的 root 稱為 $，表達式可以使用以下方式
```
$.store.book[0].title
``` 
或用括號表示
```
$[store][book][0][title]
```

**操作符號**
| Function | Description | Example | Result |
| --- | --- | --- | --- |
| text | the plain text | kind is | kind is list|
| @ | the current object || the same as input |
|.or[] | child operator | {.kind} or | List|
| .. | recursive descent || 127.0.0.1 127.0.2 myself e2e|
| * | wildcard, Get all objects || [127.0.0.1 127.0.0.2] |
| [start:end:step]| subscript operator || myself |
| [,]| union operator || 127.0.0.1 127.0.0.2 map[cpu:4] map[cpu:8]|
|?() | filter || secret |
|range,end| iterate list | {range .items[*]}[{.metadata.name}, {.status.capacity}]|127.0.0.1 127.0.0.2 map[cpu:4] map[cpu:8]|
| " | quote interpreted string | {range .items[*]}{.metadata.name}{’\t’} | 127.0.0.1 127.0.0.2 |

**函數**
函數可以在路徑的尾部調用，函數的輸出是路徑表達式的輸出，該函數的輸出是由函數本身所決定
| 函數 | 描述 | 輸出 |
| --- | --- |  --- |
| min() | 提供數組的最小值 | double|
| max() | 提供數組的最大值 | double |
| avg() | 提供數字、數組的平均值 | double |
| stddev() | 提供數字、數組的標準偏差值 | double |
| length() | 提供數組長度 | integer |

**過濾器算符**
過濾器用在篩選數組的邏輯表示式，一個典型的過濾器是 [?(@.avg > 18)]，其中 @ 表示處理當前的項目。可以使用邏輯算符 && 和 || 創建更複雜的過濾器，字串文字必須用單引號括起來 [?(@.color == 'blue')] 或 [?(@.color == "blue")]
| 操作 | 說明 |
| --- | --- |
| == | 等於，注意數字與字串不相等，ex 1 不等於 "1" |
| < ||
| <= ||
| > ||
| >= ||
| =~ | 匹配正規表達式 [?(@.name =~ /foo.*?/i)]|
| in | 左邊的存在於右邊 [?(@.size in ["S","M"])]|
| nin | 左邊的不存在右邊 |
| size | (數組或字串) 長度 |
| empty| (數組或字串) 為空 |

---

# 範例
以下用 kubernetes 的 node, pod, deploy 為範例。
使用 `kubectl get` 指令可以決定輸出的格式，參數為 `-o`。支援的格式有 `yaml`、`json`
例：
取所有 node 並輸出 json 格式
```
kubectl get nodes -o json
```

## node 的 json 結構
```json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                ...
            },
            ...
            "spec": {
                "taints": [
                    ...
                ]
            },
            "status": {
                "addresses": [
                    ...
                ],
                ...
                "images": [
                    ...
                ]
                ...
            }
        },
        {
            // node01
            ...
        },
        ...
    ]

}
```

## 取回 node 所有 IP
```yaml
...
            "status": {
                "addresses": [
                    {
                        "address": "192.168.153.101",
                        "type": "InternalIP"
                    },
                    {
                        "address": "master",
                        "type": "Hostname"
                    }
                ],
...
```
ip 存在 status 的 addresses，在 addressed 中可以定位的元素為 type，所以在 addresses 中可以加入過濾條件 tyep
```
kubectl get nodes -o jsonpath='{$.items[*].status.addresses[?(@.type=="InternalIP")].address}'
```

## 取回所有 node 的 osImage
因為在整份 json 中只有一個 osImage 所以可以用以下的語法
```
kubectl get nodes -o jsonpath='{..osImage}'
```
如需要輸出格為 <node name> <os image> 可改用以下的語法，從以下語法可看出 kubectl 的指令接收 jsonpath 的格式為一個大括號{} 表示一個處理節點
```
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.nodeInfo.osImage}{"\n"}{end}'
```

## 取回 deploy 中 pod 與 image ，格式如下
<deploy> <pod name> <image name> <count>
```
kubectl get deploy -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.containers[].image}{"\t"}{.status.replicas}{"\n"}'
```