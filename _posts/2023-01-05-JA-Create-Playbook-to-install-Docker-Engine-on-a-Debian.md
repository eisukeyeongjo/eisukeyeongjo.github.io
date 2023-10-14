# 5. DebianにDockerをインストールするAnsible Playbookを作成する 

昨年新たにRaspberry Piを購入したのですが、 Dockerを都度インストールするのがめんどくさいので[Ansible](https://www.ansible.com/)でセットアップできるようにしました。
Raspberry Piの標準OSであるRaspberry Pi OSは[Debian](https://www.debian.org/)がベースとなっているので、 
Debianのシステムにインストールする[ドキュメント](https://docs.docker.com/engine/install/debian/)を参考にしました。 
Playbookにするのは以下の手順になります。

```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

作成したplaybookはこんな感じ。 
`apt_key` の箇所は公式ドキュメントと少し違うところがありますが、 [ドキュメント](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_key_module.html)とエラーメッセージをみながらよしなにへんこうしております。

```
$ cat playbooks/debian/_docker.yml 
- name: Uninstall old versions
  become: yes
  apt:
    name: ['docker', 'docker-engine', 'docker.io', 'containerd', 'runc']
    state: absent

- name: Set up the repository
  become: yes
  apt:
    name: ['ca-certificates', 'curl', 'gnupg', 'lsb-release', 'software-properties-common']
    update_cache: yes

- name: Add Dockers official GPG key
  become: yes
  apt_key:
    url: https://download.docker.com/linux/debian/gpg
    keyring: /etc/apt/trusted.gpg.d/docker.gpg

- name: Set debian architecture
  command: dpkg --print-architecture
  register: architecture

- name: Set ubuntu codename
  command: lsb_release -cs
  register: codename

- name: Set up the stable repository
  become: yes
  apt_repository:
    repo: deb [arch="{{ architecture.stdout }}" signed-by=/etc/apt/trusted.gpg.d/docker.gpg] https://download.docker.com/linux/debian "{{ codename.stdout }}" stable

- name: Install Docker Engine
  become: yes
  apt:
    name: ['docker-ce', 'docker-ce-cli', 'containerd.io', 'docker-compose-plugin']
    update_cache: yes
```

上記Playbookをモジュール化しました。

```
$ cat playbooks/setup_debian.yml 
- hosts: dev03_debian
  tasks:
  - include_tasks: ./debian/_docker.yml
```

inventoryファイルを設置します。

```
$ cat inventory
---

devs:
  hosts:
    dev03_debian:
      ansible_host: public_ip
      ansible_user: username
      ansible_ssh_private_key_file: "~/path/to/key.pem"
```

実行してみます。（初回実行ではないですが証跡）

```
$ ansible-playbook -i inventory playbooks/setup_debian.yml 

PLAY [dev03_debian] *****************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [dev03_debian]

TASK [include_tasks] ****************************************************************************************************
included: /home/ezquerro/projects/setup/playbooks/debian/_docker.yml for dev03_debian

TASK [Uninstall old versions] *******************************************************************************************
ok: [dev03_debian]

TASK [Set up the repository] ********************************************************************************************
ok: [dev03_debian]

TASK [Add Dockers official GPG key] *************************************************************************************
changed: [dev03_debian]

TASK [Set debian architecture] ******************************************************************************************
changed: [dev03_debian]

TASK [Set ubuntu codename] **********************************************************************************************
changed: [dev03_debian]

TASK [Set up the stable repository] *************************************************************************************
changed: [dev03_debian]

TASK [Install Docker Engine] ********************************************************************************************
changed: [dev03_debian]

PLAY RECAP **************************************************************************************************************
dev03_debian               : ok=9    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

EC2で立てたDebian 11にログインしてDockerがインストールされていることを確認。

```
admin@ip-10-0-1-34:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
7050e35b49f5: Pull complete 
Digest: sha256:94ebc7edf3401f299cd3376a1669bc0a49aef92d6d2669005f9bc5ef028dc333
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```


