# 背景
安全组在容器层面是基础设施级的流量守门员，通过节点边界的粗粒度过滤，为容器环境提供基础网络隔离，本playbook旨在帮助用户快速部署安全组，并掌握安全组核心配置方式
# pod访问链路
## 外网访问pod
[<img width="3319" height="814" alt="Clipboard_Screenshot_1753079424" src="https://github.com/user-attachments/assets/3144ec71-4619-4426-8917-f0ba243226ae" />
](https://github.com/aliantli/sg_playbook/blob/8ce1a37c90a6d63067dca56699b4b6f8c587666b/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/1.png)
## pod访问外网
[<img width="3401" height="796" alt="企业微信截图_6527d6cd-5396-4160-b1d9-d21ffd48ed8b" src="https://github.com/user-attachments/assets/25846d8d-114c-4033-a1d5-d2759f664ead" />
](https://github.com/aliantli/sg_playbook/blob/e3eba8f7eb06d00ba23e741ec69049d0873d13ab/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/6.png)
# 前置条件
本次操作以nginx服务为例，主机端口暴露31000端口，外网访问暴露80端口<br>
1.创建tke集群VPC-CNI网络模式<br>
参考链接:https://cloud.tencent.com/document/product/457/103981 <br>
2.确保集群有可用节点且kubectl已配置访问权限<br>
参考链接:https://kubernetes.io/docs/tasks/tools/ <br>

# 快速开始
## 安全组配置
出站规则全放通或者与入站规则一致<br>
前往：私有网络-->安全-->安全组-->新建  创建安全组<br>
### 节点安全组配置
*节点安全组创建需要放通service服务所绑定的主机端口，否则可能出现外网访问504<br>
安全组示例(以31000端口为例）<br>
[<img width="1442" height="308" alt="Clipboard_Screenshot_1753082934" src="https://github.com/user-attachments/assets/09b5476d-3617-4758-a903-7ac9c1b10f5f" />
](https://github.com/aliantli/sg_playbook/blob/691cd5763b0d7db549267d30ab2aa2f53e96f580/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/2.png)

### 弹性网卡安全组配置
*弹性网卡安全组需要放通pod上部署的服务访问端口否则会出现外网访问504<br>
安全组示例(以80端口为例)<br>
[<img width="1031" height="308" alt="Clipboard_Screenshot_1753084104" src="https://github.com/user-attachments/assets/37b3249f-c93e-42f2-bf0d-54e7475f47b7" />
](https://github.com/aliantli/sg_playbook/blob/7ecbe1fea4a7fce2e86a92cc16d4d3c7be2eefe1/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/3.png)
### clb安全组配置
*clb安全组创建需要放通ingress所绑定的监听端口，否则会出现外网访问502<br>
安全组示例(以80端口为例)<br>
[<img width="1031" height="308" alt="Clipboard_Screenshot_1753084104" src="https://github.com/user-attachments/assets/37b3249f-c93e-42f2-bf0d-54e7475f47b7" />
](https://github.com/aliantli/sg_playbook/blob/7ecbe1fea4a7fce2e86a92cc16d4d3c7be2eefe1/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/3.png)
## 服务部署<br>
1.创建原生节点并绑定已创建好的安全组<br>
参考链接:https://cloud.tencent.com/document/product/457/78198<br> 
2.家目录创建[deployment.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/deployment.yaml), [service.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/service.yaml), [ingress.yaml](https://github.com/aliantli/sg_playbook/blob/6b6b9cfb132f7a3a261bcfe9fe93607bbda99a2c/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/ingress.yaml)文件<br>
3.执行下列命令<br>
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
到此服务及其安全组已经部署完成
# 验证
前往 控制台-->集群-->集群name-->服务与路由-->ingress<br>
点击下面内容进行访问
[<img width="1007" height="322" alt="Clipboard_Screenshot_1753084765" src="https://github.com/user-attachments/assets/cddcbe22-2c2f-4318-917e-eb66fcce1fdd" />
](https://github.com/aliantli/sg_playbook/blob/523b58ea6cae9fff44eb136ad1fe740798ffea68/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/4.png)
出现以下内容即为成功<br>
[<img width="1055" height="285" alt="Clipboard_Screenshot_1753085070" src="https://github.com/user-attachments/assets/2cd1b0db-37f4-46db-9e0a-7b2be9e9fc60" />
](https://github.com/aliantli/sg_playbook/blob/d4bc87a84ab8baa36198368791f1d094ec462a60/VPC-CNI%E4%B8%8B%E5%AE%89%E5%85%A8%E7%BB%84%E5%AE%9E%E8%B7%B5/image/5.png)
# 问题快速排查
   | 状态码 | 排查点| 排查项目|
   | :-----: | :--: | :-----: |
   | 502  | clb层 |  是否放通服务端口 |
   | 503 | 弹性网卡层 |  是否放通服务暴露的端口 |
   | | 节点层 |  是否放通service所绑定的主机端口 |

