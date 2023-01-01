# dockerfile指令

## dockerfile主要组成部分

* 基础镜像信息 ：FROM centos:centos7
* 制作镜像操作指令 ： RUN yum install openssh-server -y
* 容器启动时执行初始命令 ： CMD ["/bin/bash"]

**dockerfile中不能包含交互式命令**

## **dockerfile常用指令**

```shell
FROM  指定基础镜像
MAINTAINER 指定维护者信息，可以没有
LABLE      描述，标签，可以没有
RUN 需要执行的命令
ADD 制作docker基础的系统镜像（只会自动解压tar）
WORKDIR 设置当前工作目录（进入镜像的默认目录）
VOLUME 设置卷，挂载主机目录
EXPOSE 指定对外的端口（-P 随机端口）
CMD 指定容器启动后执行的命令（可以被替换，能够被docker run启动容器命令后面的命令行参数替换，docker run -it [image] /bin/bash,"/bin/bash会替换cmd指定的命令"）
```

## dockerfile其他指令

```shell
COPY 复制文件（不会解压）
ENV 环境变量
ENTRYPOINT 指定容器启动后执行的命令（无法被替换，启动容器的时候指定的命令，会被当成参数）
```

## **RUN指令原理**

```shell
FROM centos:latest 加载镜像
RUN 启动一个临时容器，如果执行的命令产生文件变化（例如：yum install XXX），保存文件变化，提交为临时镜像，删除临时容器
RUN ...
RUN 将最后一个临时镜像 build 成镜像
```

‍
