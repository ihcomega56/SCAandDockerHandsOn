`docker network rm my-network`

## Docker Hubからイメージを取得し実行する

- dockerイメージを検索する  
`docker search hello-world`
- dockerイメージを取得する  
`docker pull hello-world:latest`
- コンテナを作成する  
`docker create --name first-container hello-world:latest`
- コンテナ一覧を確認する  
`docker ps -a`  
- コンテナをフォアグラウンドで起動する  
`docker start -a first-container`　　
　　　　―　`-a`をつけないとバックグラウンドで実行される
- コンテナ一覧を確認する  
`docker ps -a`  

- 取得・作成・起動を同時に行う  
`i/bash`  
    - itの解説する(i: インタラクティブ-ホスト→コンテナ, t: TTY-コンテナ→ホストと標準出入力を使えるようにする)
    - ctrl + p -> ctrl + q
    - `docker attach [ID / name]`
    - `exit` / ctrl + d

- イメージ一覧を取得する  
`docker images`  
`docker rmi hello-world`  
`docker ps -a`  
`docker rm first-container`  
`docker rmi hello-world`  
    - `-f` について補足する
    - `docker rmi $(docker images -q)` で未使用イメージの一括削除

## Dockerfileを作成、ビルドし実行する

- サンプルプロジェクトを取得する  
https://github.com/ihcomega56/DockerHandsOn/blob/main/HandsOn.md　　
    - 簡単にプロジェクト構成を説明する　　

- Dockerfileを作成する(MySQL)

- ビルドする  
`docker build -t my-mysql-image:1.0 .`  
    - `-t` はイメージの名前・タグを指定する
    - タグは指定しなければ `latest` となる(付与する場合も使う場合も)
`docker images`  

- バックグラウンドで実行する  
`docker run -p 3333:3306 -d --name my-mysql my-mysql-image:1.0`
    - `--name` はコンテナの名前を指定する (イメージの名前とは別)

- ログを見る  
`docker logs -f my-mysql`
    - `-f` はフォロー
    - `--tail=20` とかもできる

- コンテナの中に入る  
`docker exec -it my-mysql mysql -p`  
(パスワード入力を求められる)  
`docker exec -it my-mysql /bin/bash` とかもできる  

- サンプルアプリ用Dockerfileを編集する(Springのアプリケーション)

- アーティファクトを作る`  
`cd spring-application-for-docker-handson`  
`./gradlew bootJar`  

- ビルドする
`docker build -t sample-application:0.0.1 .`

- ネットワークを作る
`docker network create my-network`

- MySQLのコンテナも同じネットワーク内で再度立ち上げる  
`docker rm -f my-mysql`  
`docker run -p 3333:3306 -d --network my-network --name my-mysql my-mysql-image:1.0`

- バックグラウンドで実行する  
`docker run -p 8888:8080 -d --network my-network --name sample-application-container sample-application:0.0.1`
`docker logs -f sample-application-container`

- ブラウザでアクセスする
http://localhost:8888

- MySQLにデータを1件入れてみる
`docker exec -it my-mysql mysql -p`
`use test;`
`insert into profiles(name) values('Yokona');`

- ブラウザでアクセスする
http://localhost:8888

- 起動中のコンテナ一覧を見る  
`docker ps`  
    - 停止中のコンテナも見る時は `-a`

- 起動中のコンテナをすべて停止する  
`docker stop my-mysql`  
`docker stop sample-application-container`
    - `docker stop $(docker ps -q);` で一括停止

## イメージをレジストリで管理する

- ローカル・リモート・バーチャルリポジトリを作成する

- イメージにタグを打つ
`docker tag my-mysql-image:1.0 ihcomegag.jfrog.io/handson-docker/my-mysql-image:1.0`  
`docker tag sample-application:0.0.1 ihcomegag.jfrog.io/handson-docker/sample-application:0.0.1`  
    - 最初からこの名前でビルドしてもよい  
`docker images`

- ログインする
`docker login ihcomegag.jfrog.io`

- バーチャルリポジトリに対しpushを行う  
`docker push ihcomegag.jfrog.io/handson-docker/my-mysql-image:1.0`
`docker push ihcomegag.jfrog.io/handson-docker/sample-application:0.0.1`

- バーチャルリポジトリ越しにpullを行う  
`docker pull ihcomegag.jfrog.io/handson-docker/hello-world:latest`

- JFrog Platformツアーをする
    - artifactsビュー(詳細含む)
    - packages
    - 他リポジトリのビルドインフォなど

## Docker Composeで複数コンテナを実行する

- docker-compose.ymlを作成する
    - volumesとentrypointはまだ書かない

- バックグラウンドで実行する  
`docker-compose up -d`

- Dockerfileで設定した内容を上書きする

- アプリをちょっと書き換えてJARを作り直しコンテナを再起動する
`./gradlew bootRun`
`docker-compose restart`

- 停止する  
`docker-compose down`
