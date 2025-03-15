# 概要
Docker/Kubernetes の操作を学ぶリポジトリ

# 導入手順
### Windows
1. Docker Desktop for Windows をインストールする(これだけ)<br>
Mac, Linux 向けの Docker もある(未検証)

# 学んだこと
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
