# Summary

##  如何安装 Nexus

* 拉取 Nexus 镜像
    ```bash
    docker pull sonatype/nexus3
    ```
* 启动 Nexus 容器
    ```bash
    mkdir /home/nexus && chown -R 200 /home/nexus
    docker run -d -p 8081:8081 -p 8082:8082 \
    --name nexus \
    -v /home/nexus:/nexus-data \
    --restart always \
    sonatype/nexus3

    firewall-cmd --zone=public --add-port=8081/tcp --permanent
    firewall-cmd --zone=public --add-port=8082/tcp --permanent
    ```
  

## 如何创建 Docker 镜像库

* 登陆 Nexus 网页 http://192.168.1.42:8081
* 点击设置 => Repository => Repositories => Create Repository => 选择 Docker(hosted) => 配置 Select Recipe
* 填写库名称 => 勾选 HTTP 端口 8082 => Allow anonymous docker pull(允许未登录拉取镜像) => Docker Registry API Support 开启 Docker V1 版本 api(兼容用)
* 私服创建后，客户端配置私服地址（IP + 8082 端口号）
    ```bash
    vi /etc/docker/daemon.json
    {
        "insecure-registries": ["192.168.1.42:8082"]
    }
    ```
* 重启 Docker
    ```bash
    docker restart docker
    ```
* 登陆 Nexus
    ```bash
    docker login 192.168.1.42:8082
    Username: admin
    Password: 1
    ```
* 推送镜像库，需要将 Docker 镜像打标签为固定格式后，才能推送


## 三种制品库类型 概念&分工

* proxy: 只下载，不允许用户推送，有缓存功能
* hosted: 只允许用户推送，不允许缓存。私有库核心，只存放自己的私有镜像或制品
* group: 上两种集合，集合为一个使用。

## 多台机器怎么实现部署

* 项目代码推送到 GitLab 远程仓库，更目录下创建好 Dockerfile 构建镜像的文件
    ```bash
    FROM nginx:1.15-alpine
    COPY dist /usr/share/nginx/html
    WORKDIR /usr/share/nginx/html
    ```
* Jenkins 拉取 GitLab 仓库代码，进行测试、构建（Jenkins 服务上生成密钥对，将公钥放在 GitLab SSH 上，实现免密登录）
* Jenkins 执行脚本之后，通过根目录下的 Dockerfile 文件构建并发布镜像到 Nexus
* 远程登陆 Nginx 服务，脚本拉取上一步推送的 Docker 镜像
* Nginx 服务在此镜像基础上，生成 Nginx 容器，即可实现将 dist 目录部署在 nginx/html 没目录下，实现项目部署

## 如何在不需要登陆镜像库又保证安全的情况下推送拉取镜像


