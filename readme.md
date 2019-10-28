# Alpine3 + Python3 + Django2 + uwgsi + nginx による Docker 環境構築

## 概要

docker コンテナを使って Django 環境を構築します。<br />
ただし、DataBase は AWS RDS などのマネージドサービスを使うことを前提としているため含めていません。<br />
<br />
また、docker内で現在のユーザーを追加するため linux の id コマンドを使っています。<br />
そのため下記動作確認済み環境以外では動かない可能性があります。<br />
docker はできるだけ最新バージョンを使い、動かない場合はご容赦ください。<br />
<br />
同封の docom.sh は docker-compose を起動する際にユーザーID等をセットするためのシェルスクリプトです。<br />
```
$ cat docom.sh
DUID=$(id -u) DGID=$(id -g) docker-compose $1 $2 $3 $4 $5 $6 $7 $8 $9 
```
DUID と DGID にユーザーIDとグループIDを入れてから docker-compose を呼び出しているだけのものです。<br />
（こういう場合はエイリアスを使うほうが良いのだろうか？）<br />

### 動作確認済みの環境<br/>

- Ubuntu 18.04.3 LTS<br />
  Docker version 19.03.4, build 9013bf583a<br />
  docker-compose version 1.24.0, build 0aa59064<br />

- Windows 10 Enterprise 1903 build 18362.418<br />
  Docker for Windows を使用<br />
  Docker version 19.03.4, build 9013bf5<br />
  docker-compose version 1.24.1, build 4667896b<br />
  **コマンドの実行は必ず Git-Bash 上で行って下さい。**<br />
  **PowerShell では id コマンドが使えないため動きません。**<br />

### フォルダ構成<br/>
- フォルダ及びファイルの構成
  ```
  + nginx
    - uwsgi_params
    + conf
      - mysite_nginx.conf
  + web
    - Dockerfile
    - requirements.txt
    - uwsgi.ini
  - docker-compose.yml
  - .gitignore
  - readme.md
  - docom.sh
  + log
    + uwsgi
      - __init__.txt
  + static
    - __init__.txt
  + media
    - __init__.txt
  + src
    - __init__.txt
  ```
  log, static, media, src はマウントするために必要で実際には空フォルダです。<br />
  （gitで空フォルダを保持するためだけに \_\_init\_\_.txt を入れています）

## 手順

1. docker image を作成するためビルドします  
  下記コマンドを docker-compose.yml ファイルのあるフォルダで実行して docker Image を作成します  
  `$ ./docom.sh build web` <br />
  もしくは<br />
  `$ DUID=$(id -u) DGID=$(id -g) docker-compose build web` <br />

2. Django project を作成する<br />
    - django-admin startproject で新規作成する場合<br />
      下記コマンドを実行します<br />
      &emsp; `$ ./docom.sh run web django-admin startproject \<project name\>` <br />
      &emsp; もしくは<br />
      &emsp; `$ DUID=$(id -u) DGID=$(id -g) docker-compose run web django-admin startproject \<project name\>` <br />
      &emsp; docker環境下で /code/ フォルダ、ローカル環境では ./src/ フォルダにプロジェクトが作成されます<br />
      &emsp; テストプロジェクトは mysite という名前で作っているため、mysite で作ると以下の修正は不要です。<br />
      &emsp; もしうまく行かない場合はproject name を 「mysite」 で作り手順 3. の uwsgi.ini を src/mysite/mysite へコピーして下さい。<br />
      &emsp; 手順 7. を行えばサーバーが起動するはずです。<br />
      <br />
    - 既存のDjangoアプリを使う場合  
      ローカル環境の ./src/ フォルダ以下に \<project name\>/\<project name\>/manage.py がある構成でコピーします  
      例）mysite project で app アプリが作られている場合は以下のような構成が想定されます
      ```
      + src
        - manage.py
        + mysite
          + mysite
            - __init__.py
            - urls.py
            - wsgi.py
            - settings.py
          + app
            - apps.py
            - models.py
            - views.py
      ```
3. ./web/uwsgi.ini ファイルを `./src/<project name>/<project name>/` へコピーします  
  これはローカル環境で行います<br />
  docker環境下では `/code/<project name>/<project name>/uwsgi.ini` に配置されます。

4. 3でコピーした `./src/<project name>/<project name>/uwsgi.ini` ファイルを修正します  
  prjname=mysite となっている部分を 2. で作成もしくはコピーしたプロジェクト名に変更します  
  その他に変更すべき箇所があればそこも適宜おこないます  

5. docker-compose.yml を修正します  
  web の起動コマンドが以下となっているため /mysite/ 部分を 2. で作成もしくはコピーしたプロジェクト名に変更します<br />
    ```
    修正前  command: uwsgi --ini /code/mysite/mysite/uwsgi.ini
    修正後  command: uwsgi --ini /code/<project name>/<project name>/uwsgi.ini
    ```
6. `./nginx/conf/nginx_default.conf` を修正します  
  localhost 以外のサーバーで動かす場合はこのファイルの server_name の設定を変更します<br />
  また、nginx の設定が変更な場合はここで行えます<br />
  ファイル名は適当に変更する事も別途このフォルダに設定ファイル(*.conf)を追加する事も出来ます<br />
  （この ./nginx/conf/ フォルダを /etc/nginx/conf.d にマウントさせているため）

7. サーバーを起動します  
    ```
    $ ./docom.sh up -d
    もしくは
    $ DUID=$(id -u) DGID=$(id -g) docker-compose up -d
    ```
    webブラウザで、http://localhost:8080 にアクセスすると Django アプリが起動します<br />
    django-admin startproject でプロジェクトを作っただけなら Django のデモ画面が表示されるはずです  

8. `$ ./docom.sh down` でサーバーを終了します  

9. ログファイルはローカル環境の ./log/ 以下に集約して保存されます  
  ./log/nginx/ : nginx のログ<br />
  ./log/uwsgi/ : uwsgi のログ<br />

その他の設定等はソースファイルを参照してください<br /> 
  
以上
