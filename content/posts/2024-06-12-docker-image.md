+++
title = 'Docker 镜像构建优化'
date = 2024-06-13T20:38:01+08:00
draft = false

tags = ["Docker","Dockerfile"]
categories = ["Docker 镜像构建优化"]

+++
## Docker 镜像体积优化

参考链接：

* https://icloudnative.io/posts/docker-images-part1-reducing-image-size/



### 1、选择合适的基础镜像

常用的镜像有alpine，debian-slim

他们的体积都比 标准debian、ubuntu 和 rhel 系列的基础镜像小很多。



### 2、多阶段构建

要想大幅度减少镜像的体积，多阶段构建是必不可少的。多阶段构建的想法很简单：“我不想在最终的镜像中包含一堆 C 或 Go 编译器和整个编译工具链，我只要一个编译好的可执行文件！”

多阶段构建可以由多个 `FROM` 指令识别，每一个 `FROM` 语句表示一个新的构建阶段，阶段名称可以用 `AS` 参数指定，例如：

```dockerfile
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c
FROM ubuntu
COPY --from=mybuildstage  /src/hello .
CMD ["./hello"]
```

### 3、对不同语言进行相应优化

#### Go

```dockerfile
FROM golang:alpine
COPY hello.go .
RUN go build hello.go

FROM alpine
COPY --from=0 /go/hello .
CMD ["./hello"]
```

#### Java

`Java` 属于编译型语言，但运行时还是要跑在 `JVM` 中，这就意味着理论上可以使用任意的 `JVM` 来运行 Java 程序，系统标准库是 `musl libc` 还是 `glibc` 都无所谓。因此，也就可以使用任意带有 `JVM` 的基础镜像来构建 Java 程序，也可以使用任意带有 `JVM` 的镜像作为运行 Java 程序的基础镜像



#### 解释型语言

* Python

对于解释型语言来说，如果程序仅用到了标准库或者依赖项和程序本身使用的是同一种语言，且无需调用 C 库和外部依赖，那么使用 `Alpine` 作为基础镜像一般是没有啥问题的。一旦你的程序需要调用外部依赖，情况就复杂了，想继续使用 Alpine 镜像，就得安装这些依赖。或者换成slim镜像，slim 镜像一般都基于 `Debian` 和 `glibc`，删除了许多非必需的软件包，优化了体积。如果构建过程中需要编译器，那么 slim 镜像不适合，除此之外大多数情况下还是可以使用 slim 作为基础镜像的。