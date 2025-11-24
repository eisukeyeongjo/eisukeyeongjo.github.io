# 4. Fedora Serverに公開鍵認証でSSHログインする

先日VMWareのFusion Proを衝動買いしてしまったので、使い倒していきたい。
今後Ansibleで開発サーバーをセットアップしていくに当たり Fusion上のFedora Serverに対して公開鍵認証でSSHログインできるようにする。
Fedora Serverのセットアップ方法は割愛する。

### ホストマシーンで公開鍵と秘密鍵を生成する

```
$  ssh-keygen -t ed25519 -C "test@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/lazy_ez/.ssh/id_ed25519): 
/Users/lazy_ez/.ssh/id_ed25519 already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/lazy_ez/.ssh/id_ed25519
Your public key has been saved in /Users/lazy_ez/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:Fjp1yPEJU+UJ3iiuJWQRXdFGLH40sDntgAKrydonhWI test@example.com
The key's randomart image is:
+--[ED25519 256]--+
|    . oo+o*Bo    |
|     o o.O.X*.   |
|    . + B.@+=.   |
| . + o = +.+.    |
|.E= . + S  ..    |
|.+ .   *         |
|. o . .          |
|   o             |
|                 |
+----[SHA256]-----+
```

### SCPコマンドでFedora Serverに公開鍵を送る

この時パスワード認証でのSSH接続が可能である必要がある（公開鍵を移していないので）

```
$ scp ~/.ssh/id_ed25519.pub user@172.16.150.129:~/.ssh/
@172.16.150.129's password: 
id_ed25519.pub
```

### Fedora Server上で公開鍵の設定を行う

```
[lazy_ez@fedora-server ~]$ touch ~/.ssh/authorized_keys
[lazy_ez@fedora-server ~]$ cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

### Fedora Server上のSSHの設定でパスワード認証を不可にする

```
[lazy_ez@fedora-server ~]$ sudo cat /etc/ssh/sshd_config
# PasswordAuthentication no # <- パスワード認証不可の部分がコメントアウトされているか yes になっている
[lazy_ez@fedora-server ~]$ sudo vi /etc/ssh/sshd_config
[lazy_ez@fedora-server ~]$ sudo cat /etc/ssh/sshd_config
PasswordAuthentication no
```

### SSHを再起動する

```
[lazy_ez@fedora-server ~]$ sudo systemctl restart sshd
[lazy_ez@fedora-server ~]$ sudo systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor pres>
     Active: active (running) since Sun 2022-10-30 11:13:12 EDT; 6s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 12423 (sshd)
      Tasks: 1 (limit: 2288)
     Memory: 1.3M
        CPU: 14ms
     CGroup: /system.slice/sshd.service
             └─ 12423 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Oct 30 11:13:12 fedora systemd[1]: Starting sshd.service - OpenSSH server daemo>
Oct 30 11:13:12 fedora sshd[12423]: Server listening on 0.0.0.0 port 22.
Oct 30 11:13:12 fedora sshd[12423]: Server listening on :: port 22.
Oct 30 11:13:12 fedora systemd[1]: Started sshd.service - OpenSSH server daemon.
```

### 試しにホストマシーンからSSHを試みる

鍵ファイル作成時に入力したパスフレーズの入力を求められる

```
[lazy_ez@fedora-server ~]$ ssh -i ~/.ssh/id_ed25519 user@172.16.150.129
Web console: https://fedora:9090/ or https://172.16.150.129:9090/

Last login: Sun Oct 30 16:56:53 2022 from 172.16.150.1
[lazy_ez@fedora-server ~]$ 
```

### おまけ）Ansibleのinventoryファイルで秘密鍵のパスを指定してpingを実行する

こちらの記事を参考に、inventoryファイルの `ansible_ssh_private_key_file` の項目に秘密鍵のパスを指定する

```
$ cat inventory
---

devs:
  hosts:
    fedora_server:
      ansible_host: ipaddress 
      ansible_user: lazy_ez
      ansible_ssh_private_key_file: "~/path/to/private_key"

$ ansible-playbook -i inventory playbooks/ping.yml 

PLAY [fedora-server] ************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************
ok: [fedora-server]

TASK [Example from an Ansible Playbook] ****************************************************************************************************************
ok: [fedora-server]

PLAY RECAP *********************************************************************************************************************************************
fedora-server               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

これで完了！

