## 从私有仓库拉取镜像

---

### 登陆到 `Docker`

```shell
docker login
```

出现提示符时，输入你的 Docker 用户名和密码。

登陆过程会创建或者更新 `config.json` 文件，这个文件包含了验证口令。

查看 `config.json` 文件内容:

```shell
cat ~/.docker/config.json
```

 输出会有类似这样的一段内容： 

```json
{
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "c3R...zE2"
        }
    }
}
```

### 创建一个 Secret 来保存您的验证口令

 创建一个名为 `regsecret` 的 Secret  

```shell
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

在这里:

- `<your-registry-server>` 是你的私有仓库的地址.
- `<your-name>` 是你的 Docker 用户名.
- `<your-pword>` 是你的 Docker 密码.
- `<your-email>` 是你的 Docker 邮箱.

### 理解你的 `Secret`

![](D:\我的文档\notes\kubernetes\tasks\Configure Pods and Containers\assets\images\pull-image-private-registry\魔域-暑期-活力沙滩.png)