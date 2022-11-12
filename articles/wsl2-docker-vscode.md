---
title: "WSL2とDockerとVisual Studio Codeでつくる開発環境"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WSL", "WSL2", "Docker"]
published: true
---

この記事は，Windows PC上でWSL2（Windows Subsystem for Linux 2）とDockerを利用して，クリーンなLinuxでの（とりあえずPythonの）開発環境を構築する備忘録です．
なお，企業利用を想定し，Docker Desktopは使わないこととします．


# WSL2とLinux（Ubuntu）のインストール

以前はいろいろとやる必要がありましたが，いまは管理者権限で起動したPowerShellに以下のように入力するだけです．
詳しくは[こちら](https://docs.microsoft.com/ja-jp/windows/wsl/install)．

```powershell
wsl --install
```

ただし，このときインストールされるWSLはVer.1のことがあります．WSLのバージョンは以下のように確認できます．

```powershell
wsl --list -v
```

```powershell
  NAME      STATE           VERSION
* Ubuntu    Running         1
```

WSL2にするまえに，Linuxのカーネルを更新する必要があります．[こちら](https://docs.microsoft.com/ja-jp/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)から更新プログラムをダウンロードして実行します．
次に，PowerShellに以下のように入力します．Ubuntuの部分は上記インストールされているディストリビューションに合わせてください．

```powershell
wsl --set-version Ubuntu 2
```

Version 2になっていることを確認しましょう．

```powershell
wsl --list -v
```

```powershell
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

# Windowsターミナル

こちらは必須ではありませんが，Windowsターミナルを入れておくとよさそうです．入れ方は[こちら](https://docs.microsoft.com/ja-jp/windows/terminal/install)．

~~初期状態ではWindowsターミナルでLinuxを開くとWindowsのCドライブのユーザディレクトリがカレントディレクトリになります．~~
（初期状態の開始ディレクトリは\home\ユーザ名に変わったようです）
Windowsターミナルの設定＞プロファイル＞Ubuntu（Ubuntuを入れていたら）＞全般＞開始ディレクトリを
```
\\wsl$\Ubuntu\home\<ユーザ名>
```
に変えておくと起動時のカレントディレクトリがLinuxのホームディレクトリになります（注：／ではだめなようです）．

# デフォルトエディタの変更

こちらも必須ではありませんが，デフォルトでは設定を変更する際のエディタがnanoになっています．これをvim（最初から入っています）に変更する場合は，以下のように入力しvimを選択します．

```bash
$ sudo update-alternatives --config editor
```

なおデフォルトのエディタは以下で確認できます．

```bash
$ ls -lRa /etc/alternatives/editor
```

# WSL内のProxyの設定

以下の操作は会社内などProxyが必要な環境ではWSL内でProxyの設定をしておく必要があります．

まず，/etc/apt/内にapt.confというファイルを新規作成し，以下を書き込みます．

```
Acquire::http::proxy "http://<id>:<password>@proxy_address:port/";
Acquire::https::proxy "https://<id>:<password>@proxy_address:[port]/";
```

次に~/.bashrc内に以下を追記します（bashじゃない人はよしなに）．
やっかいなことに，大文字と小文字のもの，両方必要なようです．

```
export HTTP_PROXY=http://<id>:<password>@proxy_address:port/
export HTTPS_PROXY=${HTTP_PROXY}
export http_proxy=http://<id>:<password>@proxy_address:port/
export https_proxy=${http_proxy}
```

# WSL2内Linuxディストリビューション上でのDockerのインストール

基本的に[こちら](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)の指示に従っていくだけです．
以下の操作はWindows上ではなく，Ubuntu上で行います．

```bash
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```bash
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

次に，Dockerコマンドを実行できるよう，ユーザーをdockerグループに所属させます．

```bash
sudo usermod -aG docker $USER
```

Dockerはそのままだと起動しないので，~/.bashrcに以下のように追記しWSL接続時に起動するようにします．

```bash
sudo /etc/init.d/docker start
```

ただし，このままだとWSL接続時に毎回パスワードを聞かれるので，visudoコマンドを実行し，sudoersファイルを編集します．

```bash
sudo visudo
```

以下のように記載しておくとパスワードを聞かれなくなります．

```bash
%docker ALL=(ALL)  NOPASSWD: /etc/init.d/docker
```

以下のように入力し，「Hello from Docker!」と表示されればインストール成功です．

```bash
docker run hello-world
```

つぎに，[こちら](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)の指示に従って，Docker Compseをインストールします．

# Visual Studio Code連携

最初に ~~「Remote - Containers」と「Remote - WSL」~~ 「Remote Development」の拡張を入れます．

Dockerを使わずにWSL上で開発する場合，Windowsターミナル上のUbuntuで，任意のディレクトリから

```bash
code .
```

とするとWindows側のVisual Studio Codeが立ち上がり，WSL上のファイルの編集，WSL上のPythonの実行ができます．

# Docker内のProxyの設定

WSL内でProxyを通したように，Docker内でもProxyの設定が必要です．
~/.docker/config.jsonに（無ければ作って）以下のように記載します．

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://{HOST}:{port}",
      "httpsProxy": "http://{HOST}:{port}"
    }
  }
}
```

# WSL上のプロジェクトをコンテナ上で開発

WSL上のプロジェクトディレクトリ内（空のディレクトリでも大丈夫です）にDockerfileを作成します．とりあえずミニマムなPython環境をコンテナ側に用意する場合，Dockerfileの中身は以下のように記載します．

```dockerfile
FROM python:3.9-slim-buster
```

オフィシャルなPythonイメージとしてどのようなものが用意されているかは[こちら](https://hub.docker.com/_/python)を参照．slimではないPythonを入れると無駄に大きな環境が構築されてしまうので注意です．

次に，プロジェクトフォルダ内で「code .」と入力しVS Codeを立ち上げ，画面左下の接続マークのところをクリックし，「Reopen in Container」をクリックします．次にDockerfileからイメージを作成するので，「From 'Dockerfile'」をクリックするとコンテナ上での開発が可能となります．
このときWSL上のプロジェクトファイルはコンテナ上にマウントされるため，編集結果はWSL上のファイルにも反映されています．
また，.devcontainerというフォルダが作成され，このなかにdevcontainer.jsonというファイルが作成されています．

# Dockerfile

Dockerfileにはコンテナ作成時に実行する内容を記載します．
コマンドの実行はRUNコマンドで記述します．
slimなPythonを入れる場合，恐らくpipが入らないのでapt-get installでpipを入れます（別PCで環境構築したときはpipが入っていました...謎）．
このとき，インストールするものは改行して1行ごとに，アルファベット順に記載するのが作法のようです．

pipを使ったPythonライブラリのインストールもRUNコマンドで記載します．
この例ではnumpyをインストールしていますが，実際には「pip install -r requirements.txt」するのが一般的だと思います．

gitを使う場合は、ここでコンテナ側にもgitをインストールしておきます。
なお、なぜかコンテナ内の~/.gitconfigにuser.nameとuser.emailを設定しても上手くいかず、
VSCode上のbashから、git config --global user.nameとuser.emailを設定する必要がありました。

```dockerfile
FROM python:3.9.10-slim-buster

RUN apt-get update && apt-get install -y \
    python3-pip
RUN apt-get install -y git

RUN pip install numpy==1.22.2
```

# devcontainer.json

コンテナ上のVS Codeには拡張が何も入っていません．インストールする拡張はdevcontainer.jsonのextensionsに指定します．

```json
	"extensions": [
		"ms-python.python",
		"ms-python.vscode-pylance"
	]
```

記載する拡張の名前ですが，VS Codeでその拡張のページを開き，右側のMore Info>Identifierのところに書いてある名前を書けばOKです．

# Xサーバーの設定

このままでは、matplotlibで画像を出力したり、GUIアプリを開発したりすることができません。
そこで、Windows側にXサーバーをインストールし、Docker側からそれを利用する設定を行う必要があります。
（以下の設定は、Windows11でWSLg (Windows Subsystem for Linux GUI)が利用可能になると不要になると思います。）

## WindowsにXサーバー（VcXsrv）をインストール

[こちら](https://sourceforge.net/projects/vcxsrv/)からVcXsrvをダウンロードしてインストールします。

インストールが完了したらXLaunchを起動します。
最初の設定画面左下に、「Display number」があり、-1が設定されていると思いますが、これを0にします。
その次の次の画面、「Extra settings」で、Additional parameters for VcXsrvに「-ac -nowgl」を指定します。この追加パラメーター「-ac」は、localhostで接続を受ける場合は不要らしいですが、WSL2は違うIPでネットワークにつながっていて別端末扱いのため必要になります。「-nowgl」は、一部のGTKアプリを動かすときにエラーが出てしまうので、対策として指定しています。-nowglがなくても動くアプリもあると思います。

設定を完了すると、Windowsのファイアウォールから警告があります。WSL2は、通常の物理イーサネットとは別のIPアドレス範囲でWindows側とつながっていますので、「パブリックネットワーク」にもチェックを入れてアクセスを許可します。

## DockerのDISPLAY環境変数を設定する

あとは、Dockerファイルの末尾に以下のように追記すればOKといろいろなサイトにあるのですが、私の場合はこれではダメでした。。。

```dockerfile
ENV DISPLAY host.docker.internal:0.0
```

私がうまくいったやり方は以下の通りです。
~~[こちら](https://astherier.com/blog/2020/08/run-gui-apps-on-wsl2/#)で、DockerではなくWSL2のUbuntu側での設定方法を参考にしました。~~
~~shをbashに置き換えて元に戻すあたりはもっとよいやり方がある気がしますが。。。~~
[こちら](https://qiita.com/anagura0000/items/bef08bf129f1bd8529ce)のSHELLコマンドを使うほうが幾分スマートな気がします。
シングルクォーテーションでないと式が評価されてIPアドレスが直書きされてしまうのでご注意ください。

```dockerfile
SHELL ["/bin/bash", "-c"]
RUN echo 'export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '\''{print $2}'\''):0.0' >> ~/.profile
SHELL ["/bin/sh", "-c"]
```

# WSLコマンド

WSLのシャットダウン（再起動）

```powershell
> wsl --shutdown
```

# Dockerコマンド

イメージの一覧を表示

```bash
$ docker image ls
```

イメージを削除
オプションで-fとすると強制削除

```bash
$ docker image rm [オプション] イメージID
```

不要なイメージを一括削除
コンテナから参照されていないイメージを全て削除する場合は-aオプション

```bash
$ docker image prune [オプション]
```

コンテナの一覧を表示
停止中のコンテナも表示させるには-aオプション

```bash
$ docker container ls [オプション]
```

コンテナの削除
強制削除する場合は-fオプション

```bash
$ docker container rm [オプション] コンテナID
```

停止中のコンテナを一括削除
確認なしで削除するには-fオプション

```bash
$ docker container prune [オプション]
```
