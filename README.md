# vagrant-kubernetes

この Vagrant と Ansible のコードは、学習用のマルチノードの Kubernetes 環境を自動構築するためのものです。

起動の様子をYouTubeに登録しましたので、実施前にイメージを掴んて頂けると思います。
https://www.youtube.com/watch?v=ajU5iwy4-eg


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

起動するためのコマンドは、どのOSでも同じです。 GitHubから、このコードをクローンして、vagrant up するだけです。このコマンドの実行中は、仮想サーバーのイメージ、コンテナイメージなど、大量のダウンロードが発生します。


~~~
git clone -b 1.14 https://github.com/takara9/vagrant-kubernetes
cd vagrant-Kubernetes
vagrant up
~~~

上記のコマンドを実行して、20分程度で、master, node1, node2 の３台の仮想サーバーからなる kubernetes クラスタが起動します。

~~~
$ vagrant ssh master
<中略>

$ kubectl get node
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   103s   v1.14.0
node1    Ready    <none>   67s    v1.14.0
node2    Ready    <none>   80s    v1.14.0

$ kubectl version --short
Client Version: v1.14.0
Server Version: v1.14.0

$ kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   

$ kubectl cluster-info
Kubernetes master is running at https://172.16.20.11:6443
KubeDNS is running at https://172.16.20.11:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
~~~


## kubectl の設定方法

パソコンのOS上からkubectlコマンドを使って、master上のapiserverと連携するには、
環境変数 KUBECONFIGに、configファイルのパスを設定します。

この config ファイルは、Ansible のプレイブックから、自動生成するようになっており、
git clone したディレクトの下 kubecconfigに生成されます。

Windows10 の場合は、次のようにして、環境変数をセットすることで、kubectl コマンドが
masterと繋がるようになります。

~~~
C:\Users\Maho\tmp\vagrant-kubernetes>set KUBECONFIG=%CD%\kubeconfig\config
C:\Users\Maho\tmp\vagrant-kubernetes>kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   35m   v1.14.3
node1    Ready    <none>   35m   v1.14.3
node2    Ready    <none>   35m   v1.14.3
~~~

Linux / macOS では、git clone で作成されたディレクトリで、次のコマンドを実行します。
~~~
imac:k8s_v1.14 maho$ export KUBECONFIG=`pwd`/kubeconfig/config
imac:k8s_v1.14 maho$ kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   98s   v1.14.3
node1    Ready    <none>   76s   v1.14.3
node2    Ready    <none>   63s   v1.14.3
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
vagrant destroy -f
~~~


## Kuberntes DashBoard UI の起動方法

このコンフィグレーションでは、オプション扱いとなっている Metrics server と Dashboard を起動しています。

Dashboardにアクセスするには、パソコンのOSからアクセスできるように設定しておき、プロキシを起動してください。
ただし、このプロキシを止めるには、Ctrl-Cが必要です。

~~~
$ export KUBECONFIG=`pwd`/kubeconfig/config
$ kubectl proxy
~~~

そして、パソコンのブラウザから次のURLをアクセスします。

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

ブラウザには、Kubernetes Dashboard の認証画面が表示されます。そこでKubeconfigを選んで、kubeconfigのファイルとして、
$KUBECONFIG にセットしたパスのファイルを選択します。そして、「サインイン」ボタンをクリックします。

これで、Kubernetes Dashboard のトップ画面が表示されます。



## 質問や不具合についての注意事項

この学習環境についての質問は、Issue https://github.com/takara9/vagrant-kubernetes/issues に投稿をお願いします。他の質問サイトなどに質問を投稿することは問題ありませんが、筆者は対応しません。


以上

