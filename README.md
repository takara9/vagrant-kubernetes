# vagrant-kubernetes

この Vagrant と Ansible のコードは、学習用のマルチノードの Kubernetes 環境を自動構築するためのものです。

vagrant コマンドからクラスタを起動することで、パソコン上に仮想サーバー３台が起動して、Kuberetesの環境を
自動設定します。起動後は、kubectl コマンドで利用することができます。

1. master 172.16.20.11
1. node1  172.16.20.12
1. node2  172.16.20.13


## このクラスタを起動するために必要なソフトウェア

このコードを利用するためには、次のソフトウェアをインストールしていなければなりません。

* Vagrant (https://www.vagrantup.com/)
* VirtualBox (https://www.virtualbox.org/)
* kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* git (https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## 仮想マシンのホスト環境

Vagrant と VirtualBox が動作するOSが必要です。

* Windows10　
* MacOS
* Linux

推奨ハードウェアと言えるか、このコードの筆者の環境は以下のとおりです。

* RAM: 8GB 以上
* ストレージ: 空き領域 5GB 以上
* CPU: Intel Core i5 以上


## Kubernetesの起動方法

起動するためのコマンドは、どのOSでも同じです。 GitHubから、このコードをクローンして、vagrant up するだけです。
このコマンドの実行中は、仮想サーバーのイメージ、コンテナイメージなど、大量のダウンロードが発生します。

~~~
git clone https://github.com/takara9/vagrant-kubernetes
cd vagrant-Kubernetes
vagrant up
~~~

上記のコマンドを実行して、20分程度で、master, node1, node2 の３台の仮想サーバーからなる kubernetes クラスタが起動します。


## kubectl の設定方法

kubectlコマンドで、クラスタを操作する最も簡単な方法は、masterにログインして操作することです。

~~~
vagrant ssh master

kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    10m       v1.11.2
node1     Ready     <none>    10m       v1.11.2
node2     Ready     <none>    10m       v1.11.2
~~~

パソコンのOS上からkubectlコマンドを使って、master上のapiserverと連携するには、
環境変数 KUBECONFIGに、configファイルのパスを設定します。

この config ファイルは、Ansible のプレイブックから、自動生成するようになっており、
git clone したディレクトの下 kubecconfigに生成されます。

Windows10 の場合は、次のようにして、環境変数をセットすることで、kubectl コマンドが
masterと繋がるようになります。

~~~
C:\Users\Maho\tmp\vagrant-kubernetes>set KUBECONFIG=%CD%\kubeconfig\config
C:\Users\Maho\tmp\vagrant-kubernetes>kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    2m        v1.11.2
node1     Ready     <none>    2m        v1.11.2
node2     Ready     <none>    2m        v1.11.2
~~~

Linux / macOS では、git clone で作成されたディレクトリで、次のコマンドを実行します。
~~~
$ export KUBECONFIG=`pwd`/kubeconfig/config
$ kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    2m        v1.11.2
node1     Ready     <none>    2m        v1.11.2
node2     Ready     <none>    2m        v1.11.2
~~~


ホームディクレクトリの.kubeに、configをコピーして利用することで、環境変数 KUBECOFIG を
設定しなくても、kubectl が動作するようになります。

~~~
C:\Users\Maho\tmp\vagrant-kubernetes\kubeconfig>copy config C:\Users\Maho\.kube\
        1 個のファイルをコピーしました。
~~~

Linux / macOSでは、cp コマンドを使います。一度、kubectl コマンドを実行すれば、
ホームディレクトリに、.kube/configが作成されているので、コピーだけすれば良いです。

## Kubernetes クラスタの停止

クラスタの仮想サーバーをシャットダウンするには、次のコマンドを利用します。
再び起動するには、vagrant up で開始します。

~~~
vagrant halt
~~~


## Kubernetes クラスタの削除

仮想サーバーの環境を削除するには、次のコマンドを実行します。
これを実行した場合、仮想サーバーに加えた変更は、すべて消去されます。

~~~
vagrant destroy
~~~


以上です。
