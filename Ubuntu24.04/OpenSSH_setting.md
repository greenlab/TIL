## OpenSSH

参照元：https://www.server-world.info/query?os=Ubuntu_24.04&p=ssh

```
$ sudo apt -y install openssh-server
$ sudo nano /etc/ssh/sshd_config
// 33行目 : コメント解除して [no] に変更すると root ログイン一切禁止
PermitRootLogin no

$ sudo systemctl restart ssh 
```

### OpenSSH SSH 鍵ペア認証

鍵ペアはユーザー各々で作成します。よって、鍵ペアを作成するユーザーでサーバー側にログインして作業します。
```
$ ssh-keygen 
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/mi/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
```

authorized_keys に、ed25519.pub の内容をコピーする。
```
$ cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys 
```

### Connect from Client PC
Windows 11
サーバー側で作成した秘密鍵をクライアント側にファイル転送すると、そのクライアントから対象サーバーに、鍵認証でログイン出来るようになります。 
```
// Windowsのユーザーフォルダ直下に、「.ssh」フォルダーを作る必要があるが、別件で作成済みにつき割愛。
// サーバーで作成した秘密鍵を SCP で持ってくる。
> scp mi@192.168.11.xx:/home/mi/.ssh/id_ed25519 .ssh
// passwordを登録
```

鍵認証にした場合、以下のように SSH サーバーへのパスワード認証を禁止する
```
$ sudo nano /etc/ssh/sshd_config 

// 57行目 : コメント解除してパスワード認証不可に変更
PasswordAuthentication no
// 62行目 : 以下の設定になっているか確認
KbdInteractiveAuthentication no 

// 以下のファイルが存在する場合は 同様に内容を変更 または ファイル自体を削除
$ sudo cat /etc/ssh/sshd_config.d/50-cloud-init.conf 
cat: /etc/ssh/sshd_config.d/50-cloud.init.conf: No such file 

$ sudo systemctl restart ssh
```
