# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在引导用户通过排查安全组故障，最终掌握eni模式下原生节点非直连pod的安全组核心配置逻辑
# 访问pod链路
[<img width="3319" height="814" alt="Clipboard_Screenshot_1753079424" src="https://github.com/user-attachments/assets/3144ec71-4619-4426-8917-f0ba243226ae" />
]()

# 前置条件
本次操作以ingress下clb类型为例
1.TKE 集群已存在且需启用VPC-CNI网络模式
参考链接：https://cloud.tencent.com/document/product/457/103981 
2.确保集群有可用节点且kubectl已配置访问权限
参考链接：https://kubernetes.io/docs/tasks/tools/  
3.集群内安装并配置好terraform
terraform参考链接:https://developer.hashicorp.com/terraform 
# 环境准备
1:创建安全组(*执行脚本后会自动生成group.txt文件为后续演练完清理资源用)
在terraform目录创建addgroup.sh并添加可执行权限
文件内容参考addgroup.sh文件
当输出安全组创建完成即可
2:创建原生节点并绑定对应安全组到节点上
