# windows10+virtualbox+vagrant+centos7+docker 環境構築ベース

# 概要
- vagrant(+ansible)で、CentOS7上に Docker、Docker-composeが動作する仮想マシンを構築します

# 事前準備
- virtualboxのインストールが必要です : https://www.virtualbox.org/
- vagrantのインストールが必要です : https://www.vagrantup.com/
- Windows10home の環境上で動作確認を行っています

# 使い方
1. このリポジトリをClone
  - `clone https://github.com/0xmks/vagrant_centos7_docker_base.git`
2. コマンドプロンプト等で clone先ディレクトリを開いて起動
  - `vagrant up`

# `vagrant up`で起動するCentos7 VMについて
- `Vagrantfile`の内容を調整
- ゲストOSへモジュール追加等行いたい場合は SSH接続して直接編集するか、`ansible_local/`ディレクトリ内のAnsibleファイルを編集する。 

## SSHアクセス
- `vagrant ssh` でアクセス
- ホストOSの hostsへも `vagrant.localhost` で登録されるのでその他のSSHクライアントでの接続時は参考に

# Docker環境の調整
- Docker関連のファイルは `docker/`ディレクトリに集約しています
- `docker/docker-compose.yml`を `vagrant up`時に実行してDocker関連の環境を起動している。

## Docker webサーバ
- nginx をリバースProxy として、apache+php7 の名前ベースのVirtualhostが2サイト起動する。
- https://test01.localhost/
- https://test02.localhost/
- サイトを追加削除したい場合は `docker/docker_compose.yml`と、ホストOSのhostsへの追加のため `Vagrantfile`の ` node.hostsupdater.aliases` の値を編集すれば可能。
- コンテナに php モジュールの追加などを行いたい場合は `docker/php7-apache/build/Dockerfile`を調整する

## Virtualhost上のコンテンツ格納場所
- `public_html/*`としてコンテンツファイルは集約し、`docker/docker-compose.yml`の中でVirtualhostへの紐づけを行っている。

## Docker mysql
- ホストOS側から mysqlクライアントで  -h vagrant.localhost:3306 -uroot -ppassword で接続可能
- 設定を調整したい場合は `docker/docker_compose.yml` の内容、および `docker/mysql/build/Dcokerfile` を調整する
