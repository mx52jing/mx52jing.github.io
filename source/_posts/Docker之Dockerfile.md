---
title: Docker之Dockerfile
date: 2018-10-30 11:21:52
categories:
- Docker
tags:
- Docker
---

## Dockerfile命令以及解释

```
Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。
```

<!-- more -->

#### FROM 指定基础镜像
```
FROM basicImage
```

定制镜像都要以一个镜像为基础，FROM作用就是指定基础镜像。在Dockerfile中，FROM是必须的，必须是第一条指令。

**特殊空白镜像**

Docker存在一个特殊的scratch镜像。这个镜像是虚拟的概念，不实际存在，表示一个空白的镜像。

scratch 为基础镜像的，表示不以任何镜像为基础，所写的指令将作为镜像第一层开始存在。

#### RUN 执行命令

两种格式

1、shell格式： RUN <命令>,如下：

```
RUN echo '<h1>hello</h1>' > /usr/share/nginx/html/index.html
```

2、exec格式：RUN ["可执行文件", "参数1", "参数2"]

**注意：不要用下面这种写法**

```
FROM basicImage

RUN operation1
RUN operation2
RUN operation3
...
```

**执行RUN命令的时候，会重写hosts文件，因此在RUN之前修改hosts文件是无效的**

**Dockerfile 中每一个指令都会建立一层,RUN也是。每一个 RUN 的行为，就新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像**

这种写法创建了3层甚至更多镜像，导致镜像层数过多，而我们并不需要这么多层，所以不提倡,

正确写法：

```
FROM basicImage

RUN  yum install -y gcc gcc-c++ make perl-devel \
     && mkdir -p /opt/app/ /opt/source/ \
     && cd /opt/source \
     && 其他操作
```
#### COPY 复制文件

```
COPY 源路径 目标路径

COPY ['源路径1','源路径2', ..., '目标路径']
```
和 RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用


COPY指令是从**构建上下文**目录中的`源路径`的文件或者目录复制到新一层的镜像内的`目标路径`。

```
COPY . ./app
```

可以是多个源路径，也可以是通配符,只要匹配都会选中

```
COPY ['xxx*', 'aaa/' '/dir/']
```

* **<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。**

* **使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。**

#### ADD 更高级的复制文件

ADD和COPY格式以及性质基本一致，在COPY基础上加了一些功能。

**ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢**

**ADD具有自动解压缩功能，如果需要自动解压缩功能就使用ADD，否则如果只是单纯复制文件，COPY将是更好的选择。**

#### CMD容器启动命令

和RUN相似，也是两种格式

shell格式：

```
CMD <命令>
```

exec格式：

```
CMD ["可执行文件", "参数1", "参数2"...]
```

使用shell命令，会被包装为 sh -c 的参数形式进行执行。

例如：
```
CMD echo $HOME
```

实际执行中会变为：

```
CMD ["sh", "-c", "scho $HOME"]  // 一定要用双引号
```

#### ENTRYPOINT

和RUN相似，也是两种格式 `exec`格式和 `shell`格式

ENTRYPOINT和CMD一样，都是配置容器启动后执行的命令，ENTRYPOINT可以在docker run的时候通过--ectrypoint传入。

**当指定了 ENTRYPOINT 后，CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令。**

例如：

```
FROM centos

RUN  yum install ...

...

ENTRYPOINT ["xxx.sh"]

CMD ["aaa"]
```
aaa就会作为参数传到执行的xxx.sh内

或者是

```
FROM centos

RUN  yum install ...

...

ENTRYPOINT ["xxx.sh"]
```

比如将这个修改build成demo:v1
```
docker build -t demo:v1 .
```
然后在命令行执行
```
docker run --name demo_server demo:v1 CMD
```
此时的CMD也会作为参数传递给`ENTRYPOINT`运行的命令


#### ENV 设置环境变量

`指定一个环境变量，会被后续RUN指令使用，可以在容器内被脚本或者程序调用`

有两种格式

```
ENV key value

或者

ENV key1=value1 key2=value2 ...
```

```
...
ENV VERSON 2.2
...

或者

...
ENV VERSON=2.2 MY_ENV=hello
...
```
**注意**

> 如果设置的环境变量中间有空格，那么环境变量记得用""括起来

例如：

```
ENV XXX "Hello World"
```

#### EXPOSE 暴露端口

```
EXPOSE 7100
```
EXPOSE是声明容器运行时提供的服务端口，只是一个声明，不会因为这个声明就开启这个端口的服务。

EXPOSE 和 docker run -p <宿主端口>:<容器端口> 区别：

`docker run -p <宿主端口>:<容器端口>：映射宿主端口和容器端口，就是将容器的对应端口服务公开给外界访问`

`EXPOSE：仅仅是决定容器使用什么端口，不会自动在宿主端口进行端口映射`

#### WORKDIR 指定工作目录

**这个指定的目录很重要，会影响很多命令执行时所操作的目录，一定要写对**

指定工作目录，为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录，以后各层的当前目录就是被指定的目录，如果目录不存在，WORKDIR会帮助我们建立目录。


#### build 构建镜像

```
docker build [OPTIONS] PATH | URL | -

例如：

docker build -t nginx:v1 .
```

##### 镜像构建上下文

docker build命令末尾会指定一个路径，通常为 . ,这个.的作用就是来**指定上下文路径**。

**docker build这一构建命令的执行，实际上是使用远程调用形式在服务端（Docker 引擎）完成。在服务端构建时，有可能需要将本地的一些文件复制到镜像里面去，服务器要获取本地需要复制的文件就是要通过这个上下文路径，而这个路径是我们自己在build时指定的路径，docker build命令获取到这个路径之后，会将路径下内容打包，传给Docker引擎。这样Docker引擎收到这个上下文包之后，展开就会得到构建镜像所需文件。**

```
如果所给出的 URL 不是个仓库地址，而是个 tar 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

如果标准输入传入的是文本文件，则将其视为 Dockerfile，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 COPY 进镜像之类的事情。
```
