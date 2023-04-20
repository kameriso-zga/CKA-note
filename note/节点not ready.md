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


journalctl -u kubelet 查看日志后，报错显示systemd 下面的kubelet 下面的文件不存在
failed to load kubelet config file, error: failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory, path: /var/lib/kubelet/config.yaml"


该错误提示指出 kubelet 无法加载配置文件 /var/lib/kubelet/config.yaml，并且系统找不到该文件或目录。

通常，kubelet 在启动时会尝试加载其默认配置文件，即 /etc/kubernetes/kubelet.conf，但是如果该文件不存在或无法访问，则 kubelet 会尝试从 /var/lib/kubelet/config.yaml 加载配置文件。

因此，解决此问题的方法是确保 /var/lib/kubelet/config.yaml 文件存在，并且 kubelet 有访问权限。

您可以使用以下命令检查 /var/lib/kubelet/config.yaml 文件是否存在：

$ ls -l /var/lib/kubelet/config.yaml
如果该文件不存在，则可以尝试使用以下命令创建该文件：

$ sudo touch /var/lib/kubelet/config.yaml
如果该文件已存在，则可能是由于 kubelet 没有访问该文件的权限。您可以使用以下命令更改该文件的权限：

$ sudo chmod 644 /var/lib/kubelet/config.yaml
如果您对 kubelet 的配置文件进行了更改，您需要重启 kubelet 服务以使更改生效。您可以使用以下命令重启 kubelet 服务：

$ sudo systemctl restart kubelet（重启）




4.重启后每个节点还是显示not ready 状态,查看kubelet 的cni 插件没有安装