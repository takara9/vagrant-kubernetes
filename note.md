マスターノードへログイン

~~~
$ vagrant ssh bootnode
...
$ cd k8s
~~~

全ノードへの接続を確認

~~~
ansible -i hosts_k8s all -m ping
~~~

マスタについて確認

~~~
ansible -i hosts_k8s masters -m ping
~~~


プレイブックの適用 Ubuntu 必要なパッケージのインストール

~~~
$ ansible-playbook -i hosts_k8s playbook/install_k8s_base.yml
~~~

ロードバランサーの設定

~~~
$ ansible-playbook -i hosts_k8s playbook/install_loadbalancer.yml
~~~

マスターノードの設定

~~~
ansible-playbook -i hosts_k8s playbook/install_master_1st.yml
~~~


２番目以降のマスターノードの設定

~~~
ansible-playbook -i hosts_k8s playbook/install_master_2nd.yml
~~~


テスト用
ansible-playbook -i hosts_k8s playbook/install_test2.yml


ansible-playbook -i hosts_k8s playbook/install_all.yml
