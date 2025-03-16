# 概要
Docker/Kubernetes の操作を学ぶリポジトリ

# 導入手順
### Windows
1. Docker Desktop for Windows をインストールする(これだけ)<br>
Mac, Linux 向けの Docker もある(未検証)

# 学んだこと

### Docker は Linux で動く前提

Docker の管理ソフト(Docker Desktop for Windows など)をインストーすれば Docker は使える。<br>
その管理ソフトの実体は、各種 OS 上で Linux を動かしている。<br>
<br>
Docker コンテナの中も Linux が動く。<br>
任意の Linux ディストリビューションを入れたコンテナを使うことになる。

### Docker コンテナのライフサイクル

Docker コンテナは、作ってすぐ捨てる想定で構築する。<br>
Docker では 1 コンテナ・ 1 アプリで管理すると効果的。<br>
(ex) MySQL8.0 のコンテナを捨てて MySQL8.1 のコンテナを使うようなバージョン変更作業と相性がよい

### データの残し方

Docker コンテナを作ってすぐ捨てるので、データはコンテナの外に保存する。<br>
ホストマシン上のファイルや DB にデータを保存する。

### コンテナはイメージから作る

Docker コンテナはイメージから作る。<br>
Web サーバー(Apache, nginx) や DB(MySQL, MariaDB) などの主要なソフトウェアは、各社から公式の Docker イメージが配布されている<br>
<br>
公式の Docker イメージは Docker Hub で探せば出てくる。<br>
<br>
自分でもコンテナに追加でソフトを追加可能。<br>
また、自分で育てたコンテナを再度イメージとして出力可能。<br>
これにより、 1 人が作った Docker イメージをチームに共有して、開発環境を統一する使い方ができる。

# 使用例
## よく使うコマンド

コンテナ操作は `docker container ***` の形式を取る

+ コンテナの確認（稼働中のみ）
```
docker container ls
```
旧バージョンは次のコマンド
```
docker ps
```
+ コンテナの確認（停止中を含む）
```
docker container ls -a
```

+ コンテナの作成
```
docker container create [コンテナ名]
```

+ コンテナの削除
```
docker container rm [コンテナ名]
```
削除対象は複数指定可能
```
docker container rm [コンテナ名1] [コンテナ名2] [コンテナ名3] ...
```

コンテナ以外にも同様のコマンドがある<br>
`docker network ***`<br>
`docker image ***`<br>
`docker volume ***`

## Webサーバー + ソフトウェア + DB
### MySQL(5.7) + Wordpress
```
docker network create wordpress000net1
docker run --name mysql000ex11 -dit --net=wordpress000net1 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=wordpress000db -e MYSQL_USER=wordpress000aloekun -e MYSQL_PASSWORD=waloekunpass mysql:5 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
docker run --name wordpress000ex12 -dit --net=wordpress000net1 -p 8085:80 -e WORDPRESS_DB_HOST=mysql000ex11 -e MYSQL_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000aloekun -e WORDPRESS_DB_PASSWORD=waloekunpass wordpress
```

### MySQL(8.0.41) + Redmine
```
docker network create redmine000net2
docker run --name mysql000ex13 -dit --net=redmine000net2 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=redmine000db -e MYSQL_USER=redmine000aloekun -e MYSQL_PASSWORD=raloekunpass mysql:8.0.41 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
docker run -dit --name redmine000ex14 --network redmine000net2 -p 8086:3000 -e REDMINE_DB_MYSQL=mysql000ex13 -e REDMINE_DB_DATABASE=redmine000db -e REDMINE_DB_USERNAME=redmine000aloekun -e REDMINE_DB_PASSWORD=raloekunpass redmine
```

### MariaDB + Redmine
```
docker network create redmine000net3
docker run --name mariadb000ex15 -dit --net=redmine000net3 -e MYSQL_ROOT_PASSWORD=mariarootpass -e MYSQL_DATABASE=redmine000db -e MYSQL_USER=redmine000aloekun -e MYSQL_PASSWORD=raloekunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
docker run -dit --name redmine000ex16 --network redmine000net3 -p 8087:3000 -e REDMINE_DB_MYSQL=mariadb000ex15 -e REDMINE_DB_DATABASE=redmine000db -e REDMINE_DB_USERNAME=redmine000aloekun -e REDMINE_DB_PASSWORD=raloekunpass redmine
```

### MariaDB + Wordpress
```
docker network create wordpress000net4
docker run --name mariadb000ex17 -dit --net=wordpress000net4 -e MYSQL_ROOT_PASSWORD=mariarootpass -e MYSQL_DATABASE=wordpress000db -e MYSQL_USER=wordpress000aloekun -e MYSQL_PASSWORD=waloekunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password
docker run --name wordpress000ex18 -dit --net=wordpress000net4 -p 8088:80 -e WORDPRESS_DB_HOST=mariadb000ex17 -e WORDPRESS_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000aloekun -e WORDPRESS_DB_PASSWORD=waloekunpass wordpress
```

### ホストマシンから Docker コンテナにファイルをコピー
※ ホストマシン上にパスに該当するファイルが必要
```
docker run --name apa000ex19 -d -p 8089:80 httpd
docker cp E:\work\docker-kubernetes-practice\index.html apa000ex19:/usr/local/apache2/htdocs/
```

### Docker コンテナからホストマシンにファイルをコピー
※ ホストマシン上にパスに該当するファイルが必要
```
docker run --name apa000ex19 -d -p 8089:80 httpd
docker cp apa000ex19:/usr/local/apache2/htdocs/index.html E:\work\docker-kubernetes-practice
```

### Docker にホストマシンのフォルダをマウント
個別ファイルのマウントも可能。<br>
フォルダをマウントしたら、Dockerからもホストマシンからもファイルを出し入れできる。<br>
※ ホストマシン上にパスに該当するフォルダが必要
```
docker run --name apa000ex20 -d -p 8090:80 -v E:\work\docker-kubernetes-practice\apa_folder:/usr/local/apache2/htdocs httpd
```

### Docker のボリュームとDockerのディレクトリをマウント
```
docker volume create apa000vol1
docker run --name apa000ex21 -d -p 8091:80 -v apa000vol1:/usr/local/apache2/htdocs httpd
```
ボリュームは Docker からしかアクセスできないため、マシン上での確認方法は限られる。
+ ボリュームの確認
```
docker volume inspect apa000vol1
```
次のような結果が得られればOK
```text
[
    {
        "CreatedAt": "2025-03-15T07:20:01Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/apa000vol1/_data",
        "Name": "apa000vol1",
        "Options": null,
        "Scope": "local"
    }
]
```

+ コンテナからの確認
```
docker container inspect apa000ex21
```
次のような結果が得られればOK
```text
    ...
    "Mounts": [
        {
            "Type": "volume",
            "Name": "apa000vol1",
            "Source": "/var/lib/docker/volumes/apa000vol1/_data",
            "Destination": "/usr/local/apache2/htdocs",
            "Driver": "local",
            "Mode": "z",
            "RW": true,
            "Propagation": ""
        }
    ],
    ...
```

### Docker コンテナからイメージを作成
```
docker run --name apa000ex22 -d -p 8092:80 httpd
docker commit apa000ex22 ex22_original1
```
次のコマンドでイメージができたことを確認
```
docker image ls
```

### Dockerfile からイメージを作成
フォルダ内に Dockerfile を置いて、`docker build ***` コマンドでイメージを作成できる。<br>
+ Dockerfile の中身(ApacheのみのWebサーバ)
```
FROM httpd
COPY index.html /usr/local/apache2/htdocs/
```
次のコマンドでフォルダからイメージを作成する<br>
※ ホストマシン上にパスに該当するフォルダが必要
```
docker build -t ex22_original2 E:\work\docker-kubernetes-practice\apa_folder\
```

### Docker コンテナをカスタマイズ
Docker コンテナは Linux 環境になっている。<br>
ディストリビューション(Debian系, RedHat系など)に合わせたコマンドが使用可能。
```
docker exec -it apa000ex23 /bin/bash
```
+ Ubuntu で Apache 環境に mysql をインストールする場合<br>
※ Ubuntu イメージそのままだと apt のバージョンが古いことがある。<br>
そのときは `apt update` を実行すると上手くいくかもしれない。
```
apt install mysql-server
```

### Docker イメージを公開する
ローカルレジストリを使う場合と Docker Hub を使う場合がある。
+ ローカルレジストリを使う
1. ローカルレジストリを作る<br>
ローカルレジストリを Docker コンテナで作る。<br>
```
docker run --name registry -d -p 5000:5000 registry
```
2. Docker イメージに公開用のタグを付ける。<br>
```
docker tag [イメージ名] [レジストリ名]/[リポジトリ名]:[バージョン]
docker tag ubuntu_with_apache localhost:5000/ubuntu_aloekun:1
```
3. ローカルレジストリにイメージを公開する
```
docker push localhost:5000/ubuntu_aloekun:1
```

+ ローカルレジストリに登録されたイメージを確認する
```
curl http://localhost:5000/v2/_catalog
```
次のような結果で、`Content` の `repositories` にリポジトリ名が出ていれば OK
```
StatusCode        : 200
StatusDescription : OK
Content           : {"repositories":["ubuntu_aloekun"]}

RawContent        : HTTP/1.1 200 OK
                    Docker-Distribution-Api-Version: registry/2.0
                    X-Content-Type-Options: nosniff
                    Content-Length: 36
                    Content-Type: application/json; charset=utf-8
                    Date: Sun, 16 Mar 2025 14:16:15 GMT...
Forms             : {}
Headers           : {[Docker-Distribution-Api-Version, registry/2.0], [X-Content-Type-Options, nosniff], [Content-Length, 36], [Content-Type, application/json; charset=utf-8]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : System.__ComObject
RawContentLength  : 36
```

+ ローカルレジストリから登録したイメージを削除する<br>
公式で公開されている手順でひと手間かかる。<br>
1. ローカルレジストリのコンテナにアクセスする
```
docker exec -it registry /bin/sh
```

2. (コンテナ内)リポジトリのデータがあることを確認する
```
ls /var/lib/registry/docker/registry/v2/repositories/
```
次のようにアップロードしたリポジトリ名が表示されれば、OK
```
ubuntu_aloekun
```

3. (コンテナ内)削除するファイル一覧を確認する<br>
ここで表示された内容で消したくないものがあれば、結果をフィルタする
```
find /var/lib/registry/docker/registry/v2/repositories/
```

4. (コンテナ内)リポジトリのデータを削除する
```
rm -rf /var/lib/registry/docker/registry/v2/repositories/*
```

5. コンテナから出る
```
exit
```

6. コンテナの外でガーベジコレクションを実行する
```
docker exec registry registry garbage-collect /etc/docker/registry/config.yml
```

7. リポジトリのデータが消えたことを確認する
```
curl http://localhost:5000/v2/_catalog
```

+ Docker Hub を使う<br>
1. Docker Hub の GUI でリポジトリを作る

2. Docker Hub へログイン<br>
コンソールから Docker Hub へ `docker push` するには、
コンソールから Docker Hub へログインが必要
```
docker login
```

3. Docker Hub へ登録<br>
tag に Docker Hub の `[ユーザー名]/[リポジトリ名]` を付けて、作った tag を登録する。
```
docker push aloekun/test:1
```