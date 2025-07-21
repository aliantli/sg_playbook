# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在帮助用户快速部署安全组，掌握VPC-CNI模式下原生节点非直连pod的安全组核心配置方法
# pod访问链路
外网访问pod
[<img width="4493" height="530" alt="企业微信截图_e648c6ff-2a3e-4178-942c-ec93fca0a8e0" src="https://github.com/user-attachments/assets/7851667e-1bf3-4081-8231-c474d8a965ff" />](https://github.com/aliantli/sg_playbook/blob/0e0637fcacc2bc3e788012c0b4be200231fe58de/VPC-CNI%E5%AE%89%E5%85%A8%E7%BB%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/image/1.png)
pod访问外网

# 前置条件
本次操作以ingress下clb类型为例<br>
1.TKE 集群已存在且需启用VPC-CNI网络模式<br>
参考链接:https://cloud.tencent.com/document/product/457/103981 <br>
2.确保集群有可用节点且kubectl已配置访问权限<br>
参考链接：https://kubernetes.io/docs/tasks/tools/<br> 
3.创建deployment.yaml,service.yaml,ingress.yaml文档

# 快速开始
执行
``` kubectl apply -f   ```
# 错误排查
