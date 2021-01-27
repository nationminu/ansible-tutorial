# Simple, agentless IT automation that anyone can use
>  An ansible is a fictional device for faster-than-light communication 


# Ansible 설치
## CentOS 7
```
sudo yum install epel-release
sudo yum install ansible
```

## Ubuntu/Debian
```
sudo apt install ansible
```

## python pip
```
pip install ansible
```

## Ansible 버전 확인
```
$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.5 (default, Jul 28 2020, 12:59:40) [GCC 9.3.0]
```

# Ansible 기본 사용법
> ansible <대상호스트> -i <인벤토리파일> -m <모듈> -a <모듈 옵션> -u <SSH사용자> 
```
$ ansible --help
usage: ansible [-h] [--version] [-v] [-b] [--become-method BECOME_METHOD] [--become-user BECOME_USER] [-K] [-i INVENTORY]
               [--list-hosts] [-l SUBSET] [-P POLL_INTERVAL] [-B SECONDS] [-o] [-t TREE] [-k] [--private-key PRIVATE_KEY_FILE]
               [-u REMOTE_USER] [-c CONNECTION] [-T TIMEOUT] [--ssh-common-args SSH_COMMON_ARGS]
               [--sftp-extra-args SFTP_EXTRA_ARGS] [--scp-extra-args SCP_EXTRA_ARGS] [--ssh-extra-args SSH_EXTRA_ARGS] [-C]
               [--syntax-check] [-D] [-e EXTRA_VARS] [--vault-id VAULT_IDS]
               [--ask-vault-pass | --vault-password-file VAULT_PASSWORD_FILES] [-f FORKS] [-M MODULE_PATH] [--playbook-dir BASEDIR]
               [-a MODULE_ARGS] [-m MODULE_NAME]
               pattern
```

# Ansible 프로젝트 생성
```
git clone https://github.com/nationminu/ansible-tutorial.git
```

# 인벤토리 파일 생성
> example > inventory.txt
```
sample-1 ansible_host=172.31.25.93 ip=34.254.241.209
sample-2 ansible_host=172.31.22.153 ip=34.245.107.47
sample-3 ansible_host=172.31.20.17 ip=34.243.48.83

[web]
sample-1

[was]
sample-2

[db]
sample-3
```

# 서버 사용자 추가 
```
]$ ansible 'all' -i inventory -m user -a "name=student password={{ 'student' | password_hash('sha512') }}" -b 
```

## passlib 라이브러리 에러
> "msg": "crypt.crypt not supported on Mac OS X/Darwin, install passlib python module" 에러 발생시 python library 설치
```
]$ pip3 install passlib
```

## 1. SSH key 만들기
```
]$ mkdir key
]$ ssh-keygen -t rsa -f key/mykey
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in key/mykey.
Your public key has been saved in key/mykey.pub.
The key fingerprint is:
SHA256:eJ2mCJ3DXVoQTxkk1ep2AilCU5UN/HNSHh+hq5Q67S8 ssong@bastion.mwk8s.com
The key's randomart image is:
+---[RSA 2048]----+
|     ..o*B=+  .. |
|    o   o=o +..  |
|   . .   o++.o . |
|    .o.+o==oo..  |
|    ..*.So*+.    |
|     . + *+..    |
|      . +.oo     |
|         oE      |
|          .o.    |
+----[SHA256]-----+

]$ ls -al key/
total 8
drwxrwxr-x. 2 ssong ssong   36 Jan 26 01:47 .
drwxrwxr-x. 3 ssong ssong   91 Jan 26 01:47 ..
-rw-------. 1 ssong ssong 1675 Jan 26 01:47 mykey
-rw-r--r--. 1 ssong ssong  405 Jan 26 01:47 mykey.pub
```

## SSH KEY Copy
```
]$ ansible 'all' -i inventory -m copy -a 'src=./key/mykey.pub dest=/home/student/.ssh/authorized_keys' -b
]$ ansible 'bastion' -i inventory -m copy -a 'src=./key/mykey dest=/home/student/.ssh/id_rsa' -b

]$ssh -i key/mykey student@10.65.40.11
Last login: Tue Jan 26 01:13:53 2021 from 10.65.40.10
[student@web-0 ~]$ 
```