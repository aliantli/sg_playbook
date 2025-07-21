# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在帮助用户快速部署安全组，并掌握安全组核心配置方式
# 访问pod链路
[<img width="3319" height="814" alt="Clipboard_Screenshot_1753079424" src="https://github.com/user-attachments/assets/3144ec71-4619-4426-8917-f0ba243226ae" />
](https://github.com/aliantli/sg_playbook/blob/8ce1a37c90a6d63067dca56699b4b6f8c587666b/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/1.png)

# 前置条件
本次操作以ingress下clb类型为例<br>
1.TKE 集群已存在且需启用VPC-CNI网络模式<br>
参考链接:https://cloud.tencent.com/document/product/457/103981 <br>
2.确保集群有可用节点且kubectl已配置访问权限<br>
参考链接:https://kubernetes.io/docs/tasks/tools/  <br>
3.
# 环境准备

