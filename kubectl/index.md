# Kubectl常用命令



### 配置及上下文
**view**
```
kubectl config view # 显示合并后的 kubeconfig 配置
```
**current-context**
```
kubectl config current-context # 显示当前的上下文
```

### 创建对象
```
kubectl create -f ./my-manifest.yaml # 创建资源
kubectl create -f https://git.io/vPieo # 使用 url 来创建资源
```

### 显示和查找资源
```
kubectl get services # 创建资源
kubectl get pods -o wide -n namespace # 列出所有 pod 并显示详细信息
kubectl create -f https://git.io/vPieo # 使用 url 来创建资源
```

### scale资源
```
kubectl scale --replicas=3 rs/foo
```

### 删除资源
```
kubectl delete pods,services -l name=myLabel
```
