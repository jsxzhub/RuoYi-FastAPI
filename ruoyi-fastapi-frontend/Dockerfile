# 构建阶段
FROM node:24-alpine AS builder
ENV TZ=UTC
WORKDIR /app

# 复制源代码
COPY . .

# 设置npm镜像源
RUN npm config set registry https://registry.npmmirror.com

# 安装依赖
RUN npm install

# 执行docker构建命令
RUN npm run build:docker

# 输出阶段 - 仅导出 dist 供宿主机 Nginx 使用
FROM alpine:latest
ENV TZ=UTC
WORKDIR /app
COPY --from=builder /app/dist ./dist-src
VOLUME ["/dist"]
CMD cp -r /app/dist-src/. /dist/
