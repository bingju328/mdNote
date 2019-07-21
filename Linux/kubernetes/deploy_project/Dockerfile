FROM alpine

WORKDIR /work

ENV KUBE_VERSION 1.14.2

ADD https://storage.googleapis.com/kubernetes-release/release/v${KUBE_VERSION}/bin/linux/amd64/kubectl /usr/local/bin/kubectl

RUN chmod +x /usr/local/bin/kubectl && \
  sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
  apk add --no-cache gettext

COPY config /root/.kube/config

COPY *.yaml.template ./

