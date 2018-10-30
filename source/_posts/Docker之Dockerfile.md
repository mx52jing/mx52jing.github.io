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
**Dockerfile 中每一个指令都会建立一层,RUN也是。每一个 RUN 的行为，就新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像**

这种写法创建了3层甚至更多镜像，不提倡。

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

```
COPY . ./app
```

可以是多个源路径，只要匹配都会选中

```
COPY xxx* /dir/
COPY aaa*.json /dir/
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