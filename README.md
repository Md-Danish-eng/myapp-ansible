
# Project Title

 Run Ansible playbook from Jenkins pipeline job

 #prerequisite

 * setup one ansible-controller and nodes on instance.
 * Install jenkins on ansible-controller

 #Install ansible-controller & nodes

 CONTROLLER:

sudo su
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
ls
yum install < paste the link >
yum update -y
yum install git python python-pip python-level openssl ansible -y
ansible --version
vi /etc/ansible/hosts
[demo] #name group which you want
paste the pvt.ip of target nodes
:wq
vi /etc/ansible/ansible.cfg
uncommented inventory and sudo_user
adduser ansible
passwd ansible
< give password >
su - ansible

-------------------------------------------
go to the node1 

adduser ansible
passwd ansible

< give the password >

su - ansible

exit
---------------------------------------------

controller:

 visudo

Allow root to run any commands anywhere
ansible ALL=(ALL) NOPASSWD: ALL
:wq

<same work on the target nodes>

controller:

vi /etc/ssh/sshd_config

uncommented permitrootlogin yes
uncommented passwordauthentication yes
commented passwordauthentication no
:wq
service sshd restart

< same work on the target nodes >

contoller:

su - ansible 

ssh-keygen
enter three times
ls -a
cd .ssh/
ls
ssh-copy-id ansible@<pvt.ip of node1>

#similarly paste the other pvt.ip of nodes.

#setup jenkins 

sudo su

yum install java * -y

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

 yum install epel-release # repository that provides 'daemonize'

  yum install jenkins -y

  sudo systemctl start jenkins

  sudo systemctl enable jenkins

  sudo systemctl status jenkins

  make sure status is running

Hit the ip:8080 in the browser and check 

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

you will get password  paste in the jenkins and go with 

install suggested plugins.

Enter username & password and do next next your jenkins setup is ready.

#create a github repo and clone the repo in your ansible-controller vm

make a file apache.yml

vi apache.yml

---
- hosts: Demo #the group name you create 
  become: True
  tasks:
    - name: Install packages
      yum:
        name: "httpd"
        state: "present"
    - name: Start Apache server
      service:
        name: httpd
        state: started
        enabled: True
    - name: Deploy static website
      copy:
        src: index.html
        dest: /var/www/html/
...

:wq!

make a file dev.inv

vi dev.inv

[Demo]
172.31.30.6 <pvt.ip of nodes> ansible_user=ec2-user 

:wq!

make a file index.html

vi index.html

<h1> Hello! world from Md Danish!!</h1>

:wq!

push it to the github 

git init

git add .

git commit -m "ansiblefile"

git push origin master

Name:
Personal-access-token:

#Go to manage-jenkins > manage-plugins > available > serch for ansible plugins 

Install plugins and restart your jenkins page.

Go with manage-jenkins > Global-tool-configuration > Add ansible 

Name : ansible

path the ansible executables directory: go to vm and type which ansible and copy the path and paste here.

then save and apply.

create a new job name ansible-palybook-pipeline

choose pipeline 

#pipeline syntax

pipeline{
    agent any
    stages{
        stage("scm checkout"){
            steps{
                git 'https://github.com/Md-Danish-eng/myapp-ansible.git'
            }
        }
        stage("execute ansible playbook"){
            steps{
                ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'dev.inv', playbook: 'apache.yml'
            }
        }
    }
}

#To help for pipeline:

go to pipeline syntax and choose git #use credentials if git repo is pvt.

paste github repo and branch

and generate pipeline syntax.

go to pipeline syntax and choose ansiblePlaybook: invoke an ansible playbook

playbook file path in workspace: apache.yml

Inventory file path in workspace: dev.inv

SSH connection credentials: click on add 

kind: ssh username with pvt.key

ID: private-key

Description: ansible

username: ec2-user

select eneter directly and paste the pem file which you use for instance.

click on add .

Disable the host SSH key check: check

click the generate pipeline syntax.

#Now Build 

check console output and make sure it successfull.

#Hit the public ip of Nodes the check 

Hello! world from Md Danish!!

#you can choose GitHub hook trigger for GITScm polling

and apply and save

Go to your github repo and add webhook then change in your index.html file

go to jenkins build automatically and check the Nodes browser changes reflect.

THANKS!
