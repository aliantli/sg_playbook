# 背景
安全组作为容器基础设施级的流量管控组件，在节点边界执行粗粒度过滤，为容器环境提供基础网络隔离。本Playbook旨在指导用户在VPC-CNI网络模式下，为非直连Pod快速部署安全组并掌握核心配置方法，同时以Service（四层）和Ingress（七层）的CLB配置为例进行说明。​​
# 
# 外部入站流量 
### service四层
[<img width="3371" height="664" alt="企业微信截图_f6c8281a-5853-4513-b146-43b0151600b5" src="https://github.com/user-attachments/assets/df1de979-b3f6-4273-be5e-a10abe27f735" />
](https://github.com/aliantli/sg_playbook/blob/bc3f100afd1a3d02aa830efd5579adbaff29f09e/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_f6c8281a-5853-4513-b146-43b0151600b5.png)
### ingress七层
[<img width="3428" height="664" alt="企业微信截图_29db6bf8-057b-4234-b4b7-bf2bc612c88f" src="https://github.com/user-attachments/assets/79a309d3-9c54-4356-b7fc-b75930415785" />
](https://github.com/aliantli/sg_playbook/blob/aed4d33fe8f2817c7341d4644cc0788e354bd2ff/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_29db6bf8-057b-4234-b4b7-bf2bc612c88f.png)
# 前置条件
已创建VPC-CNI网络模式集群，并配置好kubectl访问权限
# 快速开始
本次操作中配置的服务以nginx为例,在四层Service配置中,容器端口80通过节点端口31234暴露,外网经80端口以TCP协议进行访问;七层Ingress配置中,容器端口80经节点端口31000转发,外网通过80端口以HTTP(S)协议进行访问。
## 安全组配置
service四层安全组配置：
```
   节点安全组​
      入站：放通所有源 IP 的 TCP:31234 端口<br>
      出站：放通所有流量
   ​Pod（辅助网卡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量
​   CLB（四层负载均衡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量
```
​Ingress七层安全组配置​
  ```
 ​节点安全组​
      入站：放通所有源 IP 的 TCP:31000 端口
      出站：放通所有流量
   ​Pod（辅助网卡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量
   ​CLB（七层负载均衡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量

```
## 服务部署及验证<br>
### 集群层面
1.创建原生节点并绑定已创建好的节点安全组<br>
2.为pod(辅助)网卡绑定pod(辅助)安全组
前往 控制台-->集群-->组件管理-->eniipamd-->更新配置 开启pod(辅助)网卡安全组(pod(辅助)网卡默认不绑定安全组需要手动开启)<br>
[<img width="908" height="197" alt="Clipboard_Screenshot_1753100854" src="https://github.com/user-attachments/assets/7cd0a352-beaf-459f-bab8-11658b5e2e2e" />
](https://github.com/aliantli/sg_playbook/blob/18ba73f4759d9368be1f6bc1c99e8c80251584bd/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/Clipboard_Screenshot_1753100854.png)<br>
3，将clb安全组通过注解方式进行绑定
### 节点层面
1:service四层<br>
yaml参考文件：[ng-deployment-service.yaml](https://github.com/aliantli/sg_playbook/blob/cdc31e8b133e95d6a17ef605bc316d0a9425526a/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ng-deploy-service.yaml)
```
1.创建ng-deployment-service.yaml文件
2.执行下列命令
[root@VM-35-196-tlinux ~]# kubectl apply -f ng-deployment-service.yaml  
deployment.apps/secure-web-app created
service/nodeport-svc created
ingress.networking.k8s.io/minimal-ingress created
到此服务及其安全组已经部署完成
```
2,ingress七层<br>
yaml参考文件：[ng-deployment-ingress.yaml](https://github.com/aliantli/sg_playbook/blob/8a5105bd1a1d581fef7a4e5dc95cab3968d4134c/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ng-deploy-ingress.yaml)
```
1创建ng-deployment-ingress.yaml
2:执行一下命令
[root@VM-35-22-tlinux ~]# kubectl apply -f ng-deploy-service.yaml 
deployment.apps/nginx created
service/nginx created
到此服务及其安全组已经部署完成
```
## 验证
service四层
```
#执行下面命令查看ingress所生成的供外网访问的IP
[root@VM-35-244-tlinux ~]# kubectl get service -o wide
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE     SELECTOR
kubernetes   ClusterIP      172.16.0.1      <none>           443/TCP        4h22m   <none>
nginx        LoadBalancer   172.16.60.200   119.91.244.213   80:30713/TCP   156m    app=nginx
#执行curl命令访问,返回200即为成功
[root@VM-35-22-tlinux ~]# curl -I http://119.91.244.213:80
HTTP/1.1 200 OK
Server: nginx/1.29.0
Date: Tue, 22 Jul 2025 02:44:12 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Jun 2025 17:22:41 GMT
Connection: keep-alive
ETag: "685adee1-267"
Accept-Ranges: bytes
```
ingress七层
```
#执行下面命令查看ingress所生成的供外网访问的IP
[root@VM-35-196-tlinux ~]# kubectl get ingress -o wide
NAME              CLASS    HOSTS   ADDRESS        PORTS   AGE
minimal-ingress   <none>   *       106.52.99.35   80      8m36s
#执行curl命令访问，返回200即为成功
[root@VM-35-196-tlinux ~]# curl -I 106.52.99.35
HTTP/1.1 200 OK
Date: Mon, 21 Jul 2025 10:02:10 GMT
Content-Type: text/html
Content-Length: 615
Connection: keep-alive
Server: nginx/1.25.5
Last-Modified: Tue, 16 Apr 2024 14:29:59 GMT
ETag: "661e8b67-267"
Accept-Ranges: bytes
```


# 问题快速排查

ingress七层：
```
502排查方式:clb层是否放通服务端口
504排查方式:pod(辅助)网卡层是否放通服务暴露的端口
```
service四层：<br>
```
访问出现time out 排查方式
clb层:检查clb安全组是否放通service所绑定的服务端口
节点层:查看节点安全组规则是否放通service所绑定的主机端口
pod(辅助)网卡层:查看pod(辅助)网卡安全组是否放通pod暴露的服务端口
```
