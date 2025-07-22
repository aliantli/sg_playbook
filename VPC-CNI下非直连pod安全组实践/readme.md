# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在帮助用户在VPV-CNI网络模式下快速为非直连pod部署安全组，并掌握安全组核心配置方式，本次提供service四层和ingress七层clb部署方式
# 
# 外部入站流量 
### service四层模式
[<img width="3371" height="664" alt="企业微信截图_f6c8281a-5853-4513-b146-43b0151600b5" src="https://github.com/user-attachments/assets/df1de979-b3f6-4273-be5e-a10abe27f735" />
](https://github.com/aliantli/sg_playbook/blob/bc3f100afd1a3d02aa830efd5579adbaff29f09e/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_f6c8281a-5853-4513-b146-43b0151600b5.png)
### ingress七层模式
[<img width="3428" height="664" alt="企业微信截图_29db6bf8-057b-4234-b4b7-bf2bc612c88f" src="https://github.com/user-attachments/assets/79a309d3-9c54-4356-b7fc-b75930415785" />
](https://github.com/aliantli/sg_playbook/blob/aed4d33fe8f2817c7341d4644cc0788e354bd2ff/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_29db6bf8-057b-4234-b4b7-bf2bc612c88f.png)
# 前置条件
已创建VPC-CNI网络模式集群，并配置好kubectl访问权限
# 快速开始
本次操作以nginx服务为例(服务端口80,主机端口31000,外网访问端口80)<br>
## 安全组配置
出站规则全放通或者按需配置，本此操作以出站规则全放通为例<br>
前往：私有网络-->安全-->安全组-->新建  创建安全组<br>

### 节点安全组配置
节点安全组创建需要放通service服务所绑定的主机端口(以31000端口为例)本次安全组id以sg-ephmfdsf为例,规则如下<br>

|出/入站| 来源| 协议端口| 策略|
|:--:  | :-----: | :--: | :-----: |
|  入站| all | tcp:31000 | 允许 |
  | 出站| all | all|  允许 |

### pod(辅助)网卡安全组配置
弹性网卡安全组需要放通pod上部署的服务访问端口(以80端口为例)，本次安全组id以sg-c2givfsx为例，规则如下
|出/入站| 来源| 协议端口| 策略|
 |:--:  | :-----: | :--: | :-----: |
 |  入站| all | tcp:80 |        允许 |
 | 出站| all | all|          允许 |
### clb安全组配置
clb安全组创建需要放通ingress所绑定的监听端口(以80端口为例)，本次安全组id以sg-m2bb6vu3为例，规则如下<br>
|出/入站| 来源| 协议端口| 策略|
 |:--:  | :--------------: | :--: | :-----: |
|  入站| all | tcp:80 | 允许 |
  | 出站| all | all|  允许 |

## 服务部署及验证<br>
### 部署方式1:ingress七层结构
1.创建原生节点并绑定已创建好的节点安全组<br>
2.家目录创建[ng-deploy-ingress.yaml](https://github.com/aliantli/sg_playbook/blob/adc761fcde1f23c7bf71025040df127d93dcbf50/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ng-deploy-ingress.yaml)文件(ingress下的clb安全组在此yaml文件内绑定，请执行前更改为自己的安全组id)<br>
3.执行下列命令<br>
```
[root@VM-35-196-tlinux ~]# kubectl apply -f lx.yaml  
deployment.apps/secure-web-app created
service/nodeport-svc created
ingress.networking.k8s.io/minimal-ingress created
```
4.为pod(辅助)网卡绑定pod(辅助)安全组
前往 控制台-->集群-->组件管理-->eniipamd-->更新配置 开启pod(辅助)网卡安全组(pod(辅助)网卡默认不绑定安全组需要手动开启)<br>
[<img width="908" height="197" alt="Clipboard_Screenshot_1753100854" src="https://github.com/user-attachments/assets/7cd0a352-beaf-459f-bab8-11658b5e2e2e" />
](https://github.com/aliantli/sg_playbook/blob/18ba73f4759d9368be1f6bc1c99e8c80251584bd/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/Clipboard_Screenshot_1753100854.png)
<br>到此服务及其安全组已经部署完成
### 验证
执行下面命令查看ingress所生成的供外网访问的IP
```
[root@VM-35-196-tlinux ~]# kubectl get ingress -o wide
NAME              CLASS    HOSTS   ADDRESS        PORTS   AGE
minimal-ingress   <none>   *       106.52.99.35   80      8m36s
```
执行curl命令访问，返回200即为成功
```
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
### 部署方式2:service四层结构
1.创建原生节点并绑定已创建好的节点安全组<br>
2.家目录创建[ng-deploy-service.yaml](https://github.com/aliantli/sg_playbook/blob/3ddd5fb950d25b04c7ccc071be55ebf6903066d8/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ng-deploy-service.yaml)<br>
3:执行下列命令:
```
[root@VM-35-22-tlinux ~]# kubectl apply -f ng-deploy-service.yaml 
deployment.apps/nginx created
service/nginx created
```
4.为pod(辅助)网卡绑定pod(辅助)安全组
前往 控制台-->集群-->组件管理-->eniipamd-->更新配置 开启pod(辅助)网卡安全组(pod(辅助)网卡默认不绑定安全组需要手动开启)<br>
[<img width="908" height="197" alt="Clipboard_Screenshot_1753100854" src="https://github.com/user-attachments/assets/7cd0a352-beaf-459f-bab8-11658b5e2e2e" />
](https://github.com/aliantli/sg_playbook/blob/18ba73f4759d9368be1f6bc1c99e8c80251584bd/VPC-CNI%E4%B8%8B%E9%9D%9E%E7%9B%B4%E8%BF%9Epod%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/Clipboard_Screenshot_1753100854.png)
<br>到此服务及其安全组已经部署完成
### 验证
执行下面命令查看ingress所生成的供外网访问的IP
```
[root@VM-35-22-tlinux ~]# kubectl get svc 
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      172.16.0.1    <none>         443/TCP        34m
nginx        LoadBalancer   172.16.91.7   106.52.99.35   80:30844/TCP   100s
```
执行curl命令访问,返回200即为成功
```
[root@VM-35-22-tlinux ~]# curl -I http://106.52.99.35:80
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
# 问题快速排查
   | 状态码 | 排查点| 排查项目|
   | :-----: | :--: | :-----: |
   | 502  | clb层 |  是否放通服务端口 |
   | 504 | pod(辅助)网卡层 |  是否放通服务暴露的端口 |
   | | 节点层 |  是否放通service所绑定的主机端口 |

