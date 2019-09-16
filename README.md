前言

  当前playbooks仅为测试hadoop生态圈组件的部署流程，并非最终实际使用场景。
playbooks中并没有做太多的异常处理，因此部署期间还需要手动接入调整。接下来介绍目录结构


目录说明:
group_vars: 作为主机组的全局变量
production：存放inventory-hosts文件
roles:
* common：作为hadoop各个组件的全局环境变量及创建base、data、log目录
* zookeeper：安装zookeeper集群的playbook
* hadoop：安装hdfs/yarn/mapred等组件playbook
* mysql：安装mysql的playbook
* hive：安装hive的playbook
* spark：安装spark的playbook，安装方式是spark on yarn

playbooks
* common.yaml
	- hosts: hadoop-cluster
  	  remote_user: root
  	  roles:
    	    - common
	
* hadoop-playbooks.yaml
	- hosts: hadoop-cluster
  	  remote_user: hadoop
  	  roles:
    	    - hadoop


	- hosts: hive
	  remote_user: hadoop
	  roles:
	    - mysql
	    - hive

	- hosts: spark
	  remote_user: hadoop
	  roles:
	    - spark

YAML文件中roles里配置的顺序具有依赖关系，依赖者要放到被依赖者之后，这样才能够实现被依赖者安装之后再安装依赖者的的正确安装顺序。在这里，由于hadoop依赖于zookeeper，所以zookeeper在最前面，而spark/hive又依赖hadoop，故spark/hive playbook放在hadoop之后。

使用方法
1.配置inventory
master1
master2
slave1

2.配置/etc/hosts
192.168.0.46 master1
192.168.0.41 master2
192.168.0.40 slave1
127.0.0.1 localhost

3.配置主机名
hostname xxx
echo xxx >> /etc/hostname

4.配置root/hadoop 免密钥登陆
ssh-keygen
ssh-copy -i xxx

5.run playbooks
ansible-playbooks common.yaml -i production/hosts
ansible-playbooks hadoop-playbooks.yaml -i production/hosts

