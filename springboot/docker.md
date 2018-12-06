| 命令 | 描述 |
| - | - |
| docker search (name)| 查找镜像，建议直接去docker.hub |
| docker pull (name)| 下载镜像 |
| docker images | 查看都有什么镜像 |
| docker rmi (name)| 删除镜像 |
| docker run --name myname -d -p ××××(主机端口):××××(docker端口) (name)| 运行镜像,--name 指定别名，-d 后台运行，-p 绑定外部主机端口和docker内部端口。|
| docker start container_name/id | 启动容器 |
| docker stop container_id | 停止容器 |
| docker ps (-a) | 查看已安装容器 |