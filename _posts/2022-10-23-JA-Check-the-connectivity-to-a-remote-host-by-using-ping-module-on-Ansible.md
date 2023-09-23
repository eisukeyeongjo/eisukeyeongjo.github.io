# 3. Ansibleのpingモジュールで対象リモートホストに疎通確認する

### 事前準備

ここでの説明は割愛させて頂きますが、予め実行サーバーにAnsibleのインストールを行い、対象のリモートサーバーも用意しておく。
実行サーバはローカルの macOS Monterey 12.4 、Ansibleのバージョンは 2.13.5 で行う。
リモートサーバーは AWS の Amazon Linux 2 を使用します。

### Inventoryを準備

実行対象のリモートサーバーを管理するために以下の inventory ファイルを作成しリポジトリのルートディレクトリに配置します。

```
$ pwd 
/Users/lazy-ez/projects/playbook
$ cat inventory
---
  hosts:
    target-host:
      ansible_host: ip_address
      ansible_user: ec2-user
      ansible_ssh_private_key_file: "~/.ssh/path/to/key.pem"
```

### Playbookを準備

こちらを参考にそのまま ping モジュールを記入する

```
$ pwd 
/Users/lazy-ez/projects/playbook
$ cat ping.yml 
- hosts: target-host
  tasks:
  - name: Example from an Ansible Playbook
    ansible.builtin.ping:
```

### Pingモジュールを実行し疎通を確認

リポジトリのルートディレクトリからinventoryファイルとPlaybookファイルを指定して実行する

```
$ pwd 
/Users/lazy-ez/projects/playbook
$ ansible-playbook -i inventory ping.yml 

PLAY [target-host] *****************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************
ok: [target-host]

TASK [Example from an Ansible Playbook] *********************************************************************************************************************
ok: [target-host]

PLAY RECAP **************************************************************************************************************************************************
target-host               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

