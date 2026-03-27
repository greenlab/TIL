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

## 2. VSCode インストール
エディタを拡充する。

[参照した元ネタ](https://code.visualstudio.com/docs/setup/linux#_install-vs-code-on-linux)

1. 上のリンク先から debパッケージをダウンロード
2. 下記実行 ※debパッケージ名はダウンロード時のバージョンに応じて変更
  ```bash
  $ cd Downloads
  $ sudo apt install ./code_1.113.0-1774364744_amd64.deb
  ```
3. VSCodeを起動して、Extensionsから、'Japanese Language Pack for Visual Studio Code'をインストール。
　　インストールが完了すると表示される 'Change Language and Restart' ボタンを押すこと

## 3. Git インストール

[参照した元ネタ](https://www.server-world.info/query?os=Ubuntu_24.04&p=japanese)

```bash
$ sudo apt -y install git
```
