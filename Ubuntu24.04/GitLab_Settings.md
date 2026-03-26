# GitLab Settings

## 1. Japanese 
[motoneta](https://www.server-world.info/query?os=Ubuntu_24.04&p=japanese)

```bash
$ sudo apt -y install language-pack-ja-base language-pack-ja
$ sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja" 
$ source /etc/default/locale
$ echo $LANG

$ sudo apt -y install task-japanese-gnome-desktop language-pack-gnome-ja-base language-pack-gnome-ja gnome-user-docs-ja libreoffice-help-ja libreoffice-l10n-ja thunderbird-locale-ja fonts-noto-cjk-extra 
```
