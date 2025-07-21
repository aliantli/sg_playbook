# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在帮助用户快速部署安全组，并掌握安全组核心配置方式
# 访问pod链路
## 外网访问pod
[<img width="3319" height="814" alt="Clipboard_Screenshot_1753079424" src="https://github.com/user-attachments/assets/3144ec71-4619-4426-8917-f0ba243226ae" />
](https://github.com/aliantli/sg_playbook/blob/8ce1a37c90a6d63067dca56699b4b6f8c587666b/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/1.png)
## pod访问外网

# 前置条件
本次操作以nginx服务为例，主机端口暴露31000端口，外网访问暴露80端口<br>
1.创建tke集群VPC-CNI网络模式<br>
参考链接:https://cloud.tencent.com/document/product/457/103981 <br>
3.创建[deployment.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/deployment.yaml),[service.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/service.yaml),[ingress.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ingress.yaml)文件<br>

# 快速开始
## 安全组配置并绑定
### 节点安全组配置
1:创建安全组
前往：私有网络-->安全-->安全组-->新建 为节点创建安全组<br>
*节点安全组创建需要放通service服务所绑定的主机端口，否则可能出现外网访问504<br>
安全组示例(以31000端口为例）<br>

2:创建原生节点
参考链接https://cloud.tencent.com/document/product/457/78198 

### 弹性网卡安全组配置
### clb安全组配置
## 服务部署，执行下列命令<br>
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
