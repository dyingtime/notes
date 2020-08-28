## 给 Pod 配置服务质量等级

---

### QoS 等级

 当 Kubernetes 创建一个 Pod 时，它就会给这个 Pod 分配一个 QoS 等级 :

- `Guaranteed` 
- `Burstable`
- `BestEffort`

其中 `Requests` 用于在调度时通知调度器,目标需要多少资源才能调度，而 `Limits` 用来告诉 Linux 内核什么时候你的进程可以为了清理空间而被杀死 .

### 创建一个 Pod 并分配 QoS 等级为 Guaranteed

想要给 Pod 分配 QoS 等级为 `Guaranteed`

- Pod 里的每个容器都必须有`memory`限制(`limits`)和请求(`requests`)，而且必须是一样的。
- Pod 里的每个容器都必须有`cpu`限制(`limits`)和请求(`requests`)，而且必须是一样的。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```

>  如果一个容器配置了`limits.memory`，但是没有配置`requests.memory`，那 Kubernetes 会自动给容器分配一个符合内存限制的请求。 类似的，如果容器配置了 `limits.cpu`，但是没有 `requests.cpu`，Kubernetes 也会自动分配一个符合限制的请求。 

### 创建一个 Pod 并分配 QoS 等级为 Burstable

当出现下面的情况时，则是一个 Pod 被分配了 QoS 等级为`Burstable` :

- 该 Pod 不满足 QoS 等级 Guaranteed 的要求。
- Pod 里至少有一个容器有`memery`或者`cpu`请求。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

### 创建一个 Pod 并分配 QoS 等级为 BestEffort

 要给一个 Pod 配置 BestEffort 的 QoS 等级, Pod 里的容器必须没有任何内存或者 CPU　的限制或请求。 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-3
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-3-ctr
    image: nginx
```

### 创建一个拥有两个容器的 Pod


 这是一个含有两个容器的 Pod 的配置文件，其中一个容器指定了内存申请为 200MB ，另外一个没有任何申请或限制。 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-4
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-4-ctr-1
    image: nginx
    resources:
      requests:
        memory: "200Mi"
  - name: qos-demo-4-ctr-2
    image: redis
```

> 注意到这个 Pod 满足了 QoS 等级 Burstable 的要求. 就是说，它不满足 Guaranteed 的要求，而且其中一个容器有内存请求。 