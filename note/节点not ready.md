现象：

kubectl get node 后每个节点显示
root@master:~# kubectl get node
NAME     STATUS     ROLES           AGE   VERSION
master   NotReady   control-plane   20h   v1.27.1
node1    NotReady   <none>          13m   v1.27.1
node2    NotReady   <none>          11m   v1.27.1


原因：
1.每个节点没有加入集群
测试办法： 是否可以ssh 登录其他集群且不需要密码

2.kubelet 服务没有启动
测试办法： systemctl status kubelet  或者 journalctl -u kubelet


3.日志打印错误：
root@master:~# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
Drop-In: /etc/systemd/system/kubelet.service.d
└─10-kubeadm.conf
Active: active (running) since Tue 2023-04-18 18:02:43 CST; 20h ago
Docs: https://kubernetes.io/docs/home/
Main PID: 82209 (kubelet)
Tasks: 11 (limit: 4556)
Memory: 38.8M
CGroup: /system.slice/kubelet.service
└─82209 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.i>

4月 19 14:59:10 master kubelet[82209]: E0419 14:59:10.458910   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:15 master kubelet[82209]: E0419 14:59:15.460035   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:20 master kubelet[82209]: E0419 14:59:20.461775   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:25 master kubelet[82209]: E0419 14:59:25.463023   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:30 master kubelet[82209]: E0419 14:59:30.467704   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:35 master kubelet[82209]: E0419 14:59:35.469821   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:40 master kubelet[82209]: E0419 14:59:40.472133   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:45 master kubelet[82209]: E0419 14:59:45.474005   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:50 master kubelet[82209]: E0419 14:59:50.475996   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
4月 19 14:59:55 master kubelet[82209]: E0419 14:59:55.477176   82209 kubelet.go:2760] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"


4.查看cni插件是否安装
