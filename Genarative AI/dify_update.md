## Dify のアップデート

v1.3.1　→　v1.5.1

参考：https://github.com/langgenius/dify/releases/tag/1.5.1

1. docker-composer.yaml のバックアップを作成する。

```
$ cd ~/dify/docker
$ cp docker-compose.yaml docker-compose.yaml.$(date +%s).bak

docker-compose.yaml.1751599713.bak
```

2. Gitのメインブランチから最新版を取得

```
$ cd ~/dify
$ git checkout main
// gitと比較して変更されているdocker-compose.yamlをスタッシュ上に退避させる
// 実際はバックアップをコピーするしかないか？
$ git stash
$ git pull origin main
```

3. docker 停止

```
$ cd ~/dify/docker
$ sudo docker compose down
```

4. データバックアップ

```
$ sudo tar -cvf volumes-$(date +%s).tgz volumes
```

5. docker compose

```
$ cp docker-compose.yaml.9999999999.bak docker-compose.yaml
$ sudo docker compose up -d
```

Git的に正しく対応したいところだが…。

一応、問題なく起動した。
