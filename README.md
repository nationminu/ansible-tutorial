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

## Ansible 기본 사용법
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
> usage : <BR>
> -b : become 명령으로 root 권한 획득 <BR>
> -k : 비밀번호 입력  <BR>
> --private-key : 키파일 위치 지정 <br>
> -f : 병렬쓰레드 수 지정 <br>


## Ansible 프로젝트 생성

> git 에서 tutorial 프로젝트를 다운로드 받고 Key 파일을 복사한다.

```
$ git clone https://github.com/nationminu/ansible-tutorial.git
$ cd ansible-tutorial

$ cp -r ~/terraform-tutorial-aws/ssh .
```

## 인벤토리 파일 생성
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

## Ansible Ad-hoc 명령

## 전체 호스트 확인
---
```
$ ansible all -i inventory --list-hosts
  hosts (3):
    sample-1
    sample-2
    sample-3
```

## ping : OS PING 모듈
---
```
$ ansible all -i inventory -m ping 
sample-1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Warning: Permanently added '172.31.25.93' (ECDSA) to the list of known hosts.\r\nubuntu@172.31.25.93: Permission denied (publickey).",
    "unreachable": true
} 
...
```
> Permission denied (publickey) 권한이 없다는 메시가 발생한다. --private-key 옵션을 사용하여 key 위치를 지정한다. <BR>
```
$ ansible all -i inventory -m ping --private-key ./ssh/key.pem
sample-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
...
```
> ansible.cfg 설정으로 해결가능. <br>
> 더이상 키를 입력하지 않기 위해 설정파일을 사용한다.
```
$ vi ansible.cfg
[defaults]
private_key_file = ./ssh/key.pem
```

### setup : OS 정보 확인 모듈  
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html

> 모든 서버에 OS 정보를 배열로 저장
```
]$ ansible 'all' -i inventory -m setup
```
> 모든 서버에 OS 정보 확인
```
]$ ansible 'all' -i inventory -m setup -a 'filter=ansible_distribution'
sample-1 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu" 
    },
    "changed": false
}
...

# For a Ubuntu Bionic Host the distribution facts look like this:
#        "ansible_distribution": "Ubuntu", 
#        "ansible_distribution_file_parsed": true, 
#        "ansible_distribution_file_path": "/etc/os-release", 
#        "ansible_distribution_file_variety": "Debian", 
#        "ansible_distribution_major_version": "18", 
#        "ansible_distribution_release": "bionic", 
#        "ansible_distribution_version": "18.04"
``` 

## user : OS 사용자 관리 모듈 <BR>
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
> example > 모든 서버에 "student" 사용자 추가 
```
$ ansible 'all' -i inventory -m user -a "name=student password={{ 'student' | password_hash('sha512') }} uid=1500" -b 
```
> authorized_key 모듈을 사용하여 키파일로 로그인을 위한 인증

https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html#examples
```
$ ansible 'all' -i inventory -m authorized_key -a "user=student state=present key={{lookup('file', '~/.ssh/authorized_keys')}}" -b
```
> 서버에 로그인해서 사용자 id 확인
```
$ ssh student@172.31.25.93
$ id
uid=1500(student) gid=1001(student) groups=1001(student)
```

## apt : 패키지 설치/삭제
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
```
$ ansible web -i inventory -m apt -a 'name=nginx state=present' -b 
$ ansible was -i inventory -m apt -a 'name=tomcat9 state=present' -b
$ ansible db  -i inventory -m apt -a 'name=mariadb-server state=present' -b

$ ansible web -i inventory -m apt -a 'name=nginx state=absent' -b
$ ansible was -i inventory -m apt -a 'name=tomcat9 state=absent' -b
$ ansible db  -i inventory -m apt -a 'name=mariadb-server state=absent' -b
```

## service : 패키지 실행/종료
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
```
ansible web -i inventory -m service -a 'name=nginx state=started' -b
ansible web -i inventory -m service -a 'name=nginx state=stopped' -b

ansible was -i inventory -m service -a 'name=tomcat9 state=started' -b
ansible was -i inventory -m service -a 'name=tomcat9 state=stopped' -b

ansible db -i inventory -m service -a 'name=mariadb state=started' -b
ansible db -i inventory -m service -a 'name=mariadb state=stopped' -b
```

## copy : HTML 컨텐츠 복사하기
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
```
ansible all -i inventory -m copy -a 'src=html/index.html dest=/var/www/html' -b

$ curl 172.31.25.93
<!DOCTYPE html>
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <p>HELLO ANSIBLE</p>
    </body>
</html>
```

## reboot : OS 재시작하고 기다리기
---
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/reboot_module.html
```
ansible web -i inventory -m reboot -a 'reboot_timeout=500' -bv
sample-1 | CHANGED => {
    "changed": true,
    "elapsed": 48,
    "rebooted": true
}
```

# Ansible Playbook
> Ansible Playbook은 yaml 포맷으로 되어 있는 파일로, Inventory에 정의된 호스트들이 무엇을 해야하는지 정의.
쉽게 작성하고 사람이 읽을 수 있는 형식으로 설계되어 있어 습득과 이해가 빠름.
여러 개의 playbook 을 정의하는 것이 가능.

> nginx 설치 playbook 예제
```
---
- name: "Install Nginx"
  hosts: web
  become: true

  tasks:
  - name: "Install Nginx"
    apt:
      name: nginx
      state: present
  
  - name: "Start Nginx"
    service:
      name: nginx
      state: started
```


# Ansible Tutorial
https://github.com/ansible/ansible-examples