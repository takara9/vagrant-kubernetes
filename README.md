# Kubernetes HA

学習用のマルチマスター ＆ マルチノードの Kubernetes 環境を構築するためのものです。

注意：動作確認が出来ていません。少々改善点するべき点が残っています。


## 起動方法

~~~
git clone https://xxx  yyy
cd yyy
vagrant up
vagrant ssh bootnode
ansible -i hosts_k8s all -m ping
ansible-playbook -i hosts_k8s playbook/install_all.yml
~~~

## この必要なソフトウェア

次のソフトウェアをインストールしていなければなりません。

* Vagrant (https://www.vagrantup.com/)
* VirtualBox (https://www.virtualbox.org/)
* kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* git (https://kubernetes.io/docs/tasks/tools/install-kubectl/)




~~~
$ vagrant ssh master

vagrant@bootnode:~/k8s$ kubectl get node
NAME      STATUS   ROLES    AGE     VERSION
master1   Ready    master   10m     v1.16.8
master2   Ready    <none>   9m43s   v1.16.8
master3   Ready    <none>   9m43s   v1.16.8
node1     Ready    <none>   9m19s   v1.16.8
node2     Ready    <none>   9m19s   v1.16.8
node3     Ready    <none>   9m18s   v1.16.8

vagrant@bootnode:~/k8s$ kubectl version --short
Client Version: v1.16.8
Server Version: v1.16.8


vagrant@master:~$ kubectl get componentstatus
NAME                 AGE
vagrant@bootnode:~/k8s$ kubectl get componentstatus
NAME                 AGE
controller-manager   <unknown>
scheduler            <unknown>
etcd-0               <unknown>
etcd-1               <unknown>

vagrant@bootnode:~/k8s$ kubectl cluster-info
Kubernetes master is running at https://172.16.2.21:6443
KubeDNS is running at https://172.16.2.21:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://172.16.2.21:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
~~~


1.16 の kubectlは次のコマンドでは componentstatus が動かない様です。バージョン 1.15 のkubeclt を利用すると表示されます。参考URL https://discuss.kubernetes.io/t/component-status-showing-unknown-in-a-multi-master-cluster/8034



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
master   Ready    master   25m   v1.16.0
node1    Ready    <none>   24m   v1.16.0
node2    Ready    <none>   24m   v1.16.0
~~~


Linux / macOS では、git clone で作成されたディレクトリで、次のコマンドを実行します。
~~~
imac:k8s_v1.16 maho$ export KUBECONFIG=`pwd`/kubeconfig/config
imac:k8s_v1.16 maho$ kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   25m   v1.16.0
node1    Ready    <none>   24m   v1.16.0
node2    Ready    <none>   24m   v1.16.0
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
