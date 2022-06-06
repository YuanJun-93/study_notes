## k3s安装

### Server端安装

**一键无脑安装**

```bash
curl -sfL https://get.k3s.io | sh -     
# 如需安装指定版本请使用：curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=**K3s版本号** sh -
```

**查看Token，安装Agent时需要使用**

```bash
cat /var/lib/rancher/k3s/server/node-token     # 记录（复制）这个 token
```

**配置文件**

```bash
/etc/rancher/k3s/k3s.yaml     # 配置文件
```

**The connection to the server localhost:8080 was refused - did you specify the right host or port?**

```bash
mkdir ~/.kube/  && cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

**测试是否安装成功**

```bash
sudo k3s kubectl get nodes
```

### Agent安装

**下载**

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://**Server IP**:6443 K3S_TOKEN=**token** sh -    
# 我们需要将 Server IP 和刚才获得的 token 填入命令并运行脚本
# 如果 Server 安装了指定版本，Agent 安装时请加上相同的版本号参数以防出错：curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=**K3s版本号** K3S_URL=https://**Server IP**:6443 K3S_TOKEN=**token** sh -
```

**The connection to the server localhost:8080 was refused - did you specify the right host or port?**

将Server上的k3s.yaml文件复制到``/etc/rancher``中

```bash
cd /etc/rancher     # 定位到 /etc/rancher 路径
mkdir k3s     # 创建 k3s 目录
cd k3s     # 进入 k3s 目录
```

然后我们需要先修改一下配置文件，需要将 k3s.yaml 中“server”的“127.0.0.1”修改为 Server 的真实 IP

然后我们尝试在两台 Agent 上传这个 k3s.yaml 到 Agent，可以发现还是存在权限不足的问题，所以我们如法炮制，依旧修改其默认权限，待上传完毕后再改回

```bash
ls -la     # 查看其默认权限为 755
chmod 777 /etc/rancher/k3s     # 将其权限改为 777
put k3s.yaml     # 上传这个文件
chmod 755 /etc/rancher/k3s     # 将其权限改回 755
ls -la     # 查看其权限是否为 755
```

### Reference

```
https://b.hui.ke/posts/install-k3s/
https://blog.csdn.net/qq_36120342/article/details/117220400
```

