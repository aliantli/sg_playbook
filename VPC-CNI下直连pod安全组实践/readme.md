# 背景
安全组作为容器基础设施级的流量管控组件，在节点边界执行粗粒度过滤，为容器环境提供基础网络隔离。本Playbook旨在指导用户在VPC-CNI网络模式下，为直连Pod快速部署安全组并掌握核心配置方法，同时以Service（四层）和Ingress（七层）的CLB配置为例进行说明。​​
# 
# 外部入站流量 
### service四层

### ingress七层

# 前置条件
已创建VPC-CNI网络模式集群，并在集群内创建一个原生节点且配置好kubectl访问权限
# 快速开始

## 安全组配置
service四层安全组配置：
```
   
   ​Pod（辅助网卡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量
​   CLB（四层负载均衡）安全组​
      入站：放通所有源 IP 的 TCP:81 端口
      出站：放通所有流量
```
​Ingress七层安全组配置​
  ```
   ​Pod（辅助网卡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量
   ​CLB（七层负载均衡）安全组​
      入站：放通所有源 IP 的 TCP:80 端口
      出站：放通所有流量

```
## 服务部署及验证<br>
### 集群层面

1.为pod(辅助)网卡绑定pod(辅助)安全组<br>
前往 控制台-->集群-->组件管理-->eniipamd-->更新配置 开启pod(辅助)网卡安全组并绑定自己所创建的pod(辅助)网卡安全组的id此处以sg-m2bb6vu3为例<br>
[<img width="908" height="197" alt="Clipboard_Screenshot_1753100854" src="https://github.com/user-attachments/assets/7cd0a352-beaf-459f-bab8-11658b5e2e2e" />
](https://github.com/aliantli/sg_playbook/blob/18ba73f4759d9368be1f6bc1c99e8c80251584bd/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/Clipboard_Screenshot_1753100854.png)<br>
2，将clb安全组通过注解方式进行绑定<br>
service四层
```
#为yaml文件里创建service部分添加如下内容
metadata:
  name: nginx
  annotations:
    service.cloud.tencent.com/security-groups: 'sg-xxxxxxx'  #改为自己所创建的安全组id
```
ingress七层
```
#为yaml文件里创建ingress部分添加如下内容
metadata:
  annotations:
    ingress.cloud.tencent.com/security-groups: 'sg-xxxxx'  #改为自己所创建的安全组id
  name: nginx
```
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

```
ingress七层
```

```


# 问题快速排查

ingress七层：
```
502排查方式:clb层是否放通服务端口
          pod(辅助)网卡层是否放通服务暴露的端口
```
service四层：<br>
```
访问出现time out 排查方式
clb层:检查clb安全组是否放通service所绑定的服务端口
pod(辅助)网卡层:查看pod(辅助)网卡安全组是否放通pod暴露的服务端口
```

