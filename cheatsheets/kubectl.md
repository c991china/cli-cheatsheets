# kubectl 速查（由 @22178384 贡献）

## 集群 / 节点
    kubectl get nodes
    kubectl cluster-info

## 工作负载
    kubectl get pods -A
    kubectl apply -f deploy.yml
    kubectl rollout status deploy/web
    kubectl delete -f deploy.yml

## 排错
    kubectl logs <pod>
    kubectl exec -it <pod> -- sh
    kubectl describe pod <pod>
