# 便利コマンド
止まっているコンテナを全て削除
```
docker system prune
```

コンテナ名を付ける
```
docker run --name <name> <image>
```
- 常に起動させるようなコンテナは名前をつけておく
- 共有サーバーを使うとき
- 他のプログラムを使用するとき

detached mode
コンテナ起動後にdetachする(バックグランドで動かす)
```
docker run -d <image>
```

foreground mode
コンテナをExit後に排除する(一回きりのコンテナ)
```
docker run --rm <image>
```
--rmをつけておくと停止後にすぐにコンテナを消してくれる

dockerfileをbuildするときにイメージに名前を付ける
```
docker build -t <name><directory>
```

-f でフィルタリングが可能<br>
danglingはdangling imageなのかの指定をする
```
docker images -f dangling=true
```

Dockerfileがbuildコンテキストに入っていない場合は-fを使用してディレクトリを指定する
```
docker build -f ../Dockerfile.dev .
```
開発用で使うDockerfileは.devのサフィックスとする

-xでどのようなオプションがshで取れるか確認できる
```
sh -x
```

shのバッチ処理でインタラクティブな操作をしないようできる
```
sh <sh file> -b
```

Linuxコマンドマニュアル表示
```
man <cmd>
```

Docker イメージ未使用イメージ削除
```
docker rmi `docker images -q`
```

未使用ボリューム一括削除
```
docker volume prune
```

未使用ネットワーク削除
```
docker network prune
```

# Dockerfile instraction
FROM, RUN, CMDが基本的に使用される

## FROM
ベースのイメージをレイヤーとして呼び出す

## RUN
好きなようなカスタマイズをしていく<br>
RUN毎にイメージレイヤーが作られる<br>
RUNを多様するとイメージが肥大化しやすい<br>

## Layerについて
RUN, COPY, ADDでレイヤーが生成される<br>
特にRUNは多様するため気を付ける必要がある<br>
&&でコマンドをつなげていくことで複数のコマンドを一つのRUNにまとめる(\で改行する)<br>
cacheで使うことで追加分のみbuildすることで効率的に構築可能<br>
追加分のコマンドは一旦分けて構築できてからRUNをまとめる

## CMD
コンテナのデフォのコマンドを使用する<br>
Dockerfileの最後に記述される<br>
デフォで起動させるコマンドを入れると良い<br>

## RUNとCMDの違い
- RUNはレイヤーは作る
- CMDはレイヤーを作らない

イメージレイヤーに含めたいものはRUNを使う

## COPY
ホストからファイルやフォルダをコンテナに渡す

## COPYとADDの違い
- 単純なコピーはCOPYを使う
- tarの圧縮ファイルをコピーして解凍したいときはADDを使う
  - ホストで渡したいものが容量や階層が深いときに圧縮されたものをコンテナに送りたいときとか

## CMD vs ENTRYPOINT
- ENTRYPOINTはrun時に上書きされない
- ENTRYPOINTがあるとCMDはオプションを記述する形を取る
- run時に上書きできるCMD部分のみ
- コンテナをコマンドのように使いたいとき

## ENV
環境変数の設定を行う<br>
```
ENV <key> <value>
ENV <key>=<value>
ENV <key>=<value> <key>=<value>
```

## WORKDIR
RUNでcdしても必ずRootDirで実行されるためdirを変更したい場合はWORKDIRを使う<br>
WORKDIR以降のコマンドは指定したdirで実行される

# Volume
-v で指定のホストディレクトリをコンテナにマウントできる<br>
デフォルトのコンテナはrootユーザーのためルート権限でマウントしたフォルダを操作できてしまうため気を付ける必要あり<br>

```
docker run -it -v <host>:<continer dir> <image> bash
```
<continer dir>事前に作成していなくても上記コマンドで生成される

# UserID, GroupIDを指定する
```
docker run -it -u $(id -u):$(id -g) -v <hostdir>:<continer dir> <image id> bash
```
-uでユーザー名とグループ名を指定できる

# Portの指定
ホストとコンテナportを繋げる(パブリッシュする)<br>

# コンピュータリソース設定
共有コンピュータ等を扱うときにコンピューターリソースを制限しないとすぐ枯渇する<br>
- --cpus: CPU数
- --memory: メモリ容量(g->GBの単位として扱う)

```
docker inspect <conter id>
```
コンテナの情報を取得できる

```
docker inspect <conter id> | grep -i cpu
```
cpuの情報を取得できる

# データ解析用環境構築

## AWSで環境構築する
1. Dockerレジストリを使う
2. Dockerfileを送る
3. Docker imageをtarにして送る

Docker imageをtarにして送るとインスタンスが外部ネットワークにつなぎに行ってビルドする必要がないためセキュアに環境を構築できる

Docker imageをtarファイルに圧縮して出力する
```
docker save <image id> > myimage.tar
```

SFTPを使ってファイルをAWSインスタンスに転送する<br>
以下コマンドでインスタンスにsshで接続しsftpを開始する
```
sftp -i ~.pem username@host
```

get, putでファイル転送を行っていく<br>
sftpで入った直前のローカルディレクトリからの位置となる
```
put local-file
```

```
get aws-file
```

tarで圧縮したdockerimageの解凍
```
docker load < myimage.tar
```

## GPUを使った環境
- nvidiaドライバーインストール
  - nvidia Tesla等のエンタープライズ向けドライバ
    - https://docs.nvidia.com/datacenter/tesla/index.html
  - nvidia GTXクライアント向けドライバ
    - https://www.geforce.com/drivers
- nvidiaのドライバーインストール完了後以下コマンドでGPUにアクセスできるか確認する
```
nvidia-smi
```
- nvidia container toolkitをインストールする以下gitの手順で入れる
  - https://github.com/NVIDIA/nvidia-docker
  - Dockerは19.03以降のものが必要
- nvidia container toolkitインストール後に以下コマンドが実行できるか確認する
```
docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
```
- GPU用にDockerfileを書き換える
  - ベースは[nvidia/cuda](https://hub.docker.com/r/nvidia/cuda)を使用する
  - keras, scipy, tensorflow-gpuを入れる
- ```docker run --gpus all -v $PWD:/work -p 8888:8888 <image id>```で実行する
- [kerasのmnistサンプル](https://github.com/keras-team/keras/blob/master/examples/mnist_cnn.py)をコピーする
- コピーしたコードをJupyter notebookに貼り付け実行する
- GPUリソースを使っていることを確認するため```nvidia-smi -l```を別ターミナルで実行する

nvidiaドライバーとdockerfileのベースとするバージョンが合わないとdocker runでエラーが起こる可能性がある
# nvidiaドライバーの更新
1. nvidiaドライババージョン確認```cat /proc/driver/nvidia/version```
2. 古いドライバ削除```sudo apt --purge remove nvidia*```
3. アップデート```sudo apt update```
4. 推奨ドライバ検索```ubuntu-drivers devices```
5. ドライバインストール```sudo apt install nvidia-driver-<version>```
6. ドライバ更新後に以下コマンドを実行してnvidia container toolkitを再度入れる
```
# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```
7. ```docker run --gpus all nvidia/cuda:9.0-base nvidia-smi```を実行してGPUリソースが繋がっている事を確認
8. GPUを使用するDockerfileをビルドし再度runする