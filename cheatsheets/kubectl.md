# kubectl cheatsheet

kubectl 1.29. `-n <ns>` = namespace; `-A` = all namespaces.

## Get

```bash
kubectl get pods                     # current namespace
kubectl get pods -A                  # all namespaces
kubectl get pods -o wide             # + node and pod IP
kubectl get pods -w                  # watch
kubectl get pods --show-labels
kubectl get pods -l app=web,tier=fe  # label selector
kubectl get deploy,svc,ingress
kubectl get all -n myapp
kubectl get events --sort-by=.lastTimestamp
kubectl get pod <p> -o yaml
kubectl get pod <p> -o jsonpath='{.status.podIP}'
kubectl get nodes -o wide
kubectl api-resources                # what kinds exist
```

## Describe (read the Events at the bottom)

```bash
kubectl describe pod <p>
kubectl describe deploy <d>
kubectl describe svc <s>             # check Endpoints
kubectl describe node <n>            # capacity, allocatable, taints
```

## Logs

```bash
kubectl logs <p>
kubectl logs -f <p>                  # follow
kubectl logs --tail=100 <p>
kubectl logs --since=10m <p>
kubectl logs <p> -c <container>      # multi-container pod
kubectl logs <p> --previous          # logs from the last crashed container!
kubectl logs -l app=web --all-containers=true
kubectl logs -f deploy/web           # one pod from the deployment
```

## Exec / port-forward / cp

```bash
kubectl exec -it <p> -- sh
kubectl exec -it <p> -c app -- bash
kubectl exec <p> -- env
kubectl exec <p> -- cat /etc/config
kubectl port-forward pod/<p> 8080:80
kubectl port-forward svc/web 8080:80
kubectl port-forward deploy/web 8080:80
kubectl cp <p>:/var/log/app.log ./app.log
kubectl cp ./conf <p>:/etc/conf
```

## Apply / create / edit / delete

```bash
kubectl apply -f manifest.yaml
kubectl apply -f dir/
kubectl apply -k overlays/prod       # kustomize
kubectl create -f x.yaml             # fails if exists; prefer apply
kubectl edit deploy/web              # opens editor, applies on save
kubectl delete -f manifest.yaml
kubectl delete pod <p>
kubectl delete pod <p> --grace-period=0 --force   # last resort
kubectl scale deploy/web --replicas=5
kubectl set image deploy/web app=myapp:v2
```

## Rollout

```bash
kubectl rollout status deploy/web
kubectl rollout history deploy/web
kubectl rollout undo deploy/web              # previous revision
kubectl rollout undo deploy/web --to-revision=3
kubectl rollout restart deploy/web           # rolling restart
kubectl rollout pause deploy/web
kubectl rollout resume deploy/web
```

## Debugging

```bash
kubectl get pod <p> -o jsonpath='{.status.containerStatuses[*].restartCount}'
kubectl get pod <p> -o jsonpath='{.status.containerStatuses[*].lastState}'
kubectl run tmp --rm -it --image=nicolaka/netshoot -- bash   # debug pod
kubectl debug <p> -it --image=busybox --target=<container>   # ephemeral container
kubectl get pod <p> -o yaml | kubectl neat 2>/dev/null || kubectl get pod <p> -o yaml
kubectl top pod / kubectl top node
```

## Namespaces / context

```bash
kubectl get ns
kubectl create ns myapp
kubectl config get-contexts
kubectl config current-context
kubectl config use-context prod
kubectl config set-context --current --namespace=myapp
```

## Taints / labels / cordon

```bash
kubectl label node n1 disktype=ssd
kubectl label pod <p> env=prod
kubectl taint node n1 key=value:NoSchedule
kubectl taint node n1 key=value:NoSchedule-    # remove
kubectl cordon <n>                             # stop scheduling here
kubectl drain <n> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <n>
```

## Notes

- `kubectl logs` shows the CURRENT container; `--previous` shows the crashed one.
- Empty `Endpoints` on a Service = selector doesn't match any Ready pods.
- `kubectl delete pod` on a Deployment-managed pod just recreates it.
- Verify `current-context` before any destructive command. Prompt showing the
  context is worth the setup.
