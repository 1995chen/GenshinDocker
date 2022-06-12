# 克隆Grasscutter
FROM bitnami/git:2.36.1-debian-11-r4 as grasscutter_source

WORKDIR /workspace
# 这里只克隆development分支
RUN git clone -b development https://github.com/Grasscutters/Grasscutter.git

# 构建jar文件
FROM openjdk:18-slim-buster as build

WORKDIR /workspace

# 拷贝源码
COPY --from=grasscutter_source /workspace/Grasscutter ./
# 编译jar文件
RUN chmod +x gradlew && ./gradlew jar

# 构建最终镜像
FROM openjdk:18-slim-buster

# 安装依赖
RUN apt-get update -y && \
  apt-get install -y tini locales git vim procps npm && \
  rm -rf /var/lib/apt/lists/*
RUN npm install --unsafe-perm -g json

# https://stackoverflow.com/a/28406007
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen

WORKDIR /home/Grasscutter

ARG VERSION=2.7

# 写入版本文件
RUN echo $VERSION > VERSION
# 拷贝jar文件
COPY --from=build /workspace/grasscutter-*.jar ./grasscutter.jar
# 拷贝p12文件
COPY build/keystore.p12 .
# 创建日志文件夹
RUN mkdir logs
# 创建资源文件夹
RUN mkdir resources
# 创建插件文件夹
RUN mkdir plugins
# 拷贝数据文件夹
COPY build/data ./data

# 回到home目录
WORKDIR /home

# 拷贝entrypoint.sh
COPY build/entrypoint.sh .

ENV LANG="en_US.UTF-8"
ENV LANGUAGE="en_US:en"
ENV LC_ALL="en_US.UTF-8"

EXPOSE 443/tcp 22102/udp
ENTRYPOINT ["sh", "entrypoint.sh"]
