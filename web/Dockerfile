FROM python:3.7.4-alpine3.10

# nginx, suprevisor, uwsgi のインストール（gcc,build-base,linux-headersはuwsgiインストール時に使うためインストールする）
#  alpine では apk を使う（add でインストール）
RUN apk update && apk add --no-cache \
  gcc \
  build-base \
  linux-headers \
  supervisor && \
  pip3 install --upgrade pip && \ 
  pip3 install uwsgi

# requirements.txt から Django などの必要なライブラリをインストール
# 不要になった gcc などをアンインストール
COPY ./requirements.txt /code/
RUN pip3 install -r /code/requirements.txt && \
    apk del gcc build-base linux-headers

# 8001番ポートを開放する
EXPOSE 8001
# 作業用フォルダを /code/ にする
WORKDIR /code/

# ユーザーを作成してカレントユーザーにする（一般ユーザーで動かすため）
# docker-compose.yml の args に指定した uid と gid を使えるように宣言する
# ユーザー名、グループ名は id と一緒にしておく
ARG DUID
ARG DGID

# ユーザーの作成
RUN addgroup -S -g $DGID $DGID && \
    adduser -S -u $DUID -g $DGID $DUID

USER $DUID
