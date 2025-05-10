## 2025/05/05

## Cockpit インストール

```
$ sudo apt install cockpit
```

http://(IPアドレス):9090 で接続可能になる。  
IP アドレスは、

```
$ ip a
```

## Cockpit Plugins

### 1. Cockpit Files 
公式プラグイン  
ファイルを扱うことができるようになる。  
```
$ sudo apt install gettext nodejs npm make

$ git clone https://github.com/cockpit-project/cockpit-files.git
$ cd cockpit-files
$ make
$ mkdir -p ~/.local/share/cockpit
$ ln -s `pwd`/dist ~/.local/share/cockpit/cockpit-files
$ npm run watch
```

反映されていない…。

### 2. Navigator
```
$ wget https://github.com/45Drives/cockpit-navigator/releases/download/v0.5.10/cockpit-navigator_0.5.10-1focal_all.deb
$ sudo apt install ./cockpit-navigator_0.5.10-1focal_all.deb
```

反映された…。

### 3. Cockpit maPodman


```
$ sudo apt install gettext nodejs make

$ git clone https://github.com/cockpit-project/cockpit-podman
$ cd cockpit-podman
$ make
```

反映されていない…。


## Cockpit-Podman リベンジ

```
$ sudo apt install cockpit-navigator cockpit-podman
```
※cockpit-navigatorは、aptにパッケージ公開されていたので、ついでに…。  

再起動  

```
$ sudo podman pull ubuntu
$ sudo podman run ubuntu /bin/echo "Welcome to the Podman World"
```

ubuntuのイメージを引っ張ってこれたので、OK！

