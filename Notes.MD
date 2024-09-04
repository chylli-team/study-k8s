1. all pods can be communicated with each other by ip address in one cluster. 
2. if one container restared, you can check previous container logs by `--previous` option.
3. liveness probe: http get, tcp socket, command 
4. check why container restarted: check logs, describe pods
5. a container exit code greater than 128, like 137, that means the container get external signal X, here 128 + X = 137
6. if you created a pod direclty, without deployment or replication controller. if the node of that pod is down, the pod cannot be recreated on otther node, because it is managed by kubelet which is also on same pod and down.
7. when you don't specify a label in a ReplacationController yaml, it will extract label from template. So, for simple, don't specify it.
8. RC only care about the pods that match labels. So if you change the label to a value that dont match the RC, the RC will not manage it, and it will create a new pod with previous label. Of course if you create a pod directly with matched label, RC will manage it (like delete one extra pod)
9. By default, when deleting an RC, the pods will also be deleted. But if you delete the RC by `kubectl delete rc my-rc --cascade=false`, the pods will not be deleted.
10. RC will be deprecated in future, use ReplicaSet instead.
11. In general you needn't create ReplicaSet directly, because Deployment will create it for you. But you still can create it manuall if you want.
12. DaemonSet use label select to match nodes. If you changed a label of a node to mismatch, the existing pod on that node will be destroyed. 

# Job
1. Job can specify completions, parallelism.

# service
1. use port name in srv and pod, so that you can change port in pod, and it will be updated in srv automatically.
2. You can specify sessionAffinity in srv, so that the request from same client will be sent to same pod.
2. you can get current namespace's service with env var.
3. you can access service by service-name.namepsace.svc.cluster.local. and you can omit cluster.local. if in same namespace, you can access with service-name directly.
4. you can't ping a service because the srv is a virtual ip which bind to special port of pods. So if that ping port is not open on that svc or pod, you can't ping it.
5. there is a resource `endpoints` sit between srv and pod. when we create a srv, the endpoints will be created automatically according to svc selector. you can see it by `kubectl get endpoints my-srv`.
6. if you create a svc without selector, then endpoints will not be created, you must/can create it manually.
7. you should create an endpoints resource with exactly same name as svc.
8. you can create a service without selector and an endpoints to point external service. So that you can use the service name to access the external service.
9. if you add a selector to a svc, the endpoints will be managed automatically. If you remove the selector, the endpoints will be not be managed, and will be constant.
10. You can create a svc with externalName, so that you can access the external service by service name. Which will create a DNS record, and bypass the kube-proxy. NO cluster IP.
11. Service type: ClusterIP, ExternalName : consumed by pods in cluster. NodePort, LoadBalancer, Ingress: consumed by external clients.
12. NodePort: open a port on every node, and also forward to the pod. So you can access it with both service cluster ip and node ip & port.
13. LoadBalancer: create a LB in cloud provider, and forward to the nodeport. So you can access it with LB service ip and LB ip. LoadBalancer is an extension of NodePort.
14. Ingress: HTTP level.
15. NodePort. if you connect to any node's port, it will be forwarded to one of the pod, not only that pod service. both cluster ip and node ip will be routed to a random pod.
16. LoadBalancer: will have its own public IP, beside pod ips.
17. If cloud provider have no LB, there will be public ip, but there will be Node Port opened like NodePort.
18. Even you set service affinity to none, the request from same client will still be sent to same pod, because browser use keep-alive connections..
19. with LoadBalancer, lb will redirect the request to a random node port. But maybe that node isn't running pod service. that means another hop is increased. You can use `externalTrafficPolicy: Local` to only forward to the local running pod on that node. But if there is no pod on that node, the request will hang there. And with it, maybe the load is unbalanced between all pods -- LB forward to nodes evenly, and nodes forward to local pods evenly, suppose 3 pods on 2 nodes, then 1 pod wil get 50%, and 2 pods will get 25%.
20. Local option mentioned above can help to get client IP, since there is no SNAT there.
21. why we need Ingress even we have LB ? every LB service need a IP. but all ingress need only one.
22. Ingress is a new kind, not service kind
23. on cloud Ingress will point to a nodeport service, but that's not k8s requirement.
24. Ingress can support https
25. liveness probe: will kill & restart pod if failed. readiness probe: will not kill pod, but will not send request to it -- it will be removed from endpoints.
26. headless service: ClusterIP: None. It will not have cluster ip, but will have dns record. It will return all pod ips.
27. by default dns will return all ready pods. if you want to get all pods including not ready, you can add an annotation.
# Volumes