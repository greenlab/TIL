# GitLab Settings
新規に、 OS インストールした Ubuntu 24.04 に対し、GitLab を設定する。  
開始日：　2026/03/26

## 1. 日本語設定
まず、GitLabの設定を行う前に、環境の日本語化から行う。  

[参照した元ネタ](https://www.server-world.info/query?os=Ubuntu_24.04&p=japanese)

```bash
$ sudo apt -y install language-pack-ja-base language-pack-ja
$ sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja" 
$ source /etc/default/locale
$ echo $LANG
ja_JP.UTF-8
$ sudo apt -y install task-japanese-gnome-desktop language-pack-gnome-ja-base language-pack-gnome-ja gnome-user-docs-ja libreoffice-help-ja libreoffice-l10n-ja thunderbird-locale-ja fonts-noto-cjk-extra 
```
OS 再起動

