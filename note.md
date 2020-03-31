マスターノードへログイン

~~~
$ vagrant ssh master1
...
$ cd /vagrant
~~~

全ノードへの接続を確認

~~~
ansible -i /vagrant/hosts_ssh all -m ping
~~~

マスタについて確認

~~~
ansible -i /vagrant/hosts_ssh masters -m ping
~~~


プレイブックの適用 Ubuntu 必要なパッケージのインストール

~~~
$ ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_k8s_base.yml
~~~

ロードバランサーの設定

~~~
$ ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_loadbalancer.yml
~~~

マスターノードの設定

~~~
ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_master_1st.yml
~~~


２番目以降のマスターノードの設定

~~~
ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_master_2nd.yml
~~~


テスト用

ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_test2.yml

ansible-playbook -i /vagrant/hosts_ssh /vagrant/playbook/install_all.yml
