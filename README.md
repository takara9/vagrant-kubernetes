# vagrant-kubernetes


## Kubernetesの起動方法

Ansibleが起動してクラスタのセットアップを実施します。

~~~
C:\Users\Maho\tmp\git clone https://github.com/takara9/vagrant-kubernetes
C:\Users\Maho\tmp\vagrant-kubernetes>vagrant up
~~~



## Windwos10　kubectl の設定方法

マスターノードにログインして操作

~~~
C:\Users\Maho\tmp\vagrant-kubernetes>vagrant ssh master
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-130-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

53 packages can be updated.
20 updates are security updates.

New release '18.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Sun Sep  2 07:44:26 2018 from 10.0.2.2
vagrant@master:~$ kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    10m       v1.11.2
node1     Ready     <none>    10m       v1.11.2
node2     Ready     <none>    10m       v1.11.2
~~~


環境変数 KUBECONFIGにパスを設定する方法

~~~
C:\Users\Maho\tmp\vagrant-kubernetes>dir
 ドライブ C のボリューム ラベルがありません。
 ボリューム シリアル番号は 26E6-1EC2 です

 C:\Users\Maho\tmp\vagrant-kubernetes のディレクトリ

2018/09/02  16:43    <DIR>          .
2018/09/02  16:43    <DIR>          ..
2018/09/02  12:06    <DIR>          .vagrant
2018/09/02  16:42    <DIR>          ansible-playbook
2018/09/02  12:06               207 ansible.cfg
2018/09/02  16:43    <DIR>          kubeconfig
2018/09/02  12:06                21 README.md
2018/09/02  16:35            46,603 ubuntu-xenial-16.04-cloudimg-console.log
2018/09/02  16:43            21,400 vag.log
2018/09/02  16:19             3,821 Vagrantfile
               5 個のファイル              72,052 バイト
               5 個のディレクトリ  20,505,161,728 バイトの空き領域

C:\Users\Maho\tmp\vagrant-kubernetes>set KUBECONFIG=%CD%\kubeconfig\config

C:\Users\Maho\tmp\vagrant-kubernetes>kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    2m        v1.11.2
node1     Ready     <none>    2m        v1.11.2
node2     Ready     <none>    2m        v1.11.
~~~

ホームディクレクトリの.kubeに、configをコピーして利用する方法

~~~
C:\Users\Maho\tmp\vagrant-kubernetes\kubeconfig>dir
 ドライブ C のボリューム ラベルがありません。
 ボリューム シリアル番号は 26E6-1EC2 です

 C:\Users\Maho\tmp\vagrant-kubernetes\kubeconfig のディレクトリ

2018/09/02  16:43    <DIR>          .
2018/09/02  16:43    <DIR>          ..
2018/09/02  16:43             5,448 config
               1 個のファイル               5,448 バイト
               2 個のディレクトリ  20,502,401,024 バイトの空き領域

C:\Users\Maho\tmp\vagrant-kubernetes\kubeconfig>copy config C:\Users\Maho\.kube\
        1 個のファイルをコピーしました。

C:\Users\Maho\tmp\vagrant-kubernetes\kubeconfig>kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    8m        v1.11.2
node1     Ready     <none>    8m        v1.11.2
node2     Ready     <none>    8m        v1.11.2
~~~
