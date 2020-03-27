# vagrant-kubernetes

この Vagrant と Ansible のコードは、学習用のマルチノードの Kubernetes 環境を自動構築するためのものです。起動の様子をYouTubeに登録しましたので、実施前にイメージを掴んて頂けると思います。 https://www.youtube.com/watch?v=ajU5iwy4-eg

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
$ git clone -b 1.17 https://github.com/takara9/vagrant-kubernetes
$ cd vagrant-Kubernetes
$ vagrant up
~~~

上記のコマンドを実行して、20分程度で、master, node1, node2 の３台の仮想サーバーからなる kubernetes クラスタが起動します。

~~~
$ vagrant status
Current machine states:

node1                     running (virtualbox)
node2                     running (virtualbox)
master                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.


$ vagrant ssh master
＜中略＞
[vagrant@master ~]$ kubectl get node
NAME     STATUS     ROLES    AGE     VERSION
master   Ready      master   6m14s   v1.18.0
node1    Ready      <none>   3m14s   v1.18.0
node2    Ready      <none>   29s     v1.18.0

vagrant@master:~$ kubectl version --short
Client Version: v1.18.0
Server Version: v1.18.0

vagrant@master:~$ kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}

vagrant@master:~$ kubectl cluster-info
Kubernetes master is running at https://172.16.20.11:6443
KubeDNS is running at https://172.16.20.11:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://172.16.20.11:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

既知の問題で以下のコマンドが実行できません。
vagrant@master:~$ kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master   196m         9%     691Mi           77%
node1    20m          2%     467Mi           52%
node2    25m          2%     468Mi           52%
~~~


## kubectl の設定方法

パソコンのOS上からkubectlコマンドを使って、master上のapiserverと連携するには、
環境変数 KUBECONFIGに、configファイルのパスを設定します。

この config ファイルは、Ansible のプレイブックから、自動生成するようになっており、
git clone したディレクトの下 kubecconfigに生成されます。

Windows10 の場合は、次のようにして、環境変数をセットすることで、kubectl コマンドが
masterと繋がるようになります。

~~~
C:\Users\MAHOTAKARA\k8s-1.18>set KUBECONFIG=%CD%\kubeconfig\config

C:\Users\MAHOTAKARA\k8s-1.17>kubectl get node
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   129m   v1.18.0
node1    Ready    <none>   128m   v1.18.0
node2    Ready    <none>   128m   v1.18.0
~~~

Linux / macOS では、git clone で作成されたディレクトリで、次のコマンドを実行します。
~~~
$ export KUBECONFIG=`pwd`/kubeconfig/config
$ kubectl get node
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   130m   v1.18.0
node1    Ready    <none>   129m   v1.18.0
node2    Ready    <none>   129m   v1.18.0
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


# Kuberntes DashBoard UI の起動方法

このコンフィグレーションでは、オプション扱いとなっている Metrics server と Dashboard を起動しています。

Dashboardにアクセスするには、パソコンのOSからアクセスできるように設定しておき、プロキシを起動してください。
ただし、このプロキシを止めるには、Ctrl-Cが必要です。

~~~
$ export KUBECONFIG=`pwd`/kubeconfig/config
$ kubectl proxy
~~~

そして、パソコンのブラウザから次のURLをアクセスします。

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

ブラウザには、Kubernetes Dashboard の認証画面が表示されます。そこでKubeconfigを選んで、kubeconfigのファイルとして、$KUBECONFIG にセットしたパスのファイルを選択します。そして、「サインイン」ボタンをクリックします。これで、Kubernetes Dashboard のトップ画面が表示されます。


# コンテナエンジンの選択

ファイル ansible-playbook/versions.yml の先頭部分に以下の記述があります。以下の例のように、デフォルトが docker-ce になっています。この設定では、Kubernetesのコンテナランタイムは、Dockerが利用されます。この値をcontainerdに変更すると、Containerdがセットアップされます。どちらも動作に変わりはありませんが、containedを選んだ方が、40MB程度メモリ消費量が抑えられます。また、K8sクラスタのノードの中で、dockerコマンドが使えなくなります。

~~~
# Container_Engine
#
# どちらか一方を選んで、有効化してください。
# Choose either one and enable it.  docker-ce or containerd
container_engine: docker-ce
#container_engine: containerd
~~~


以上です。
