### 2025/04/27 Pleasanter 用の環境を構築する。

## Ubuntu 環境に、Pleasanter をインストールする。  

下記公式サイトを参考にする。

[プリザンターを Ubuntu にインストールする](https://pleasanter.org/ja/manual/getting-started-pleasanter-ubuntu)

### 1. dotnet をインストールする
```
$ sudo wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
$ sudo chmod +x ./dotnet-install.sh
$ sudo ./dotnet-install.sh -c 8.0 -i /usr/local/bin
$ dotnet --version
```

### 2. DBをインストールする

postgresql と、 mysql が選べるようなので、postgresql にする。

#### 2.1 Postgresql をインストール
```
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt -y install postgresql-16
```

#### 2.2. PostgreSQLユーザの設定

##### 2.2.1 OS上のpostgresql管理用ユーザーのパスワード設定

```
$ sudo passwd postgres
```

パスワードは、いつものやつ

##### 2.2.2. Postgresqlログイン

ユーザーを切り替えてログイン

```
$ sudo su - postgres
$ psql -U postgres
```

##### 2.2.3. PostgreSQLの管理ユーザー "postgres" のパスワードを設定

```
$ alter role postgres with password '<新しいパスワード>';
$ exit;
```

パスワードは、いつものやつ

#### 2.3. PostgreSQLのログ出力設定

```
$ nano /etc/postgresql/16/main/postgresql.conf
```

"REPORTING AND LOGGING" の下記を編集

```
log_destination = 'stderr'
logging_collector = on
log_line_prefix = '[%t]%u %d %p[%l]'
```

#### 2.4. PostgreSQLのサービス再起動、サービス化

通常の管理者ユーザーに戻してから下記実施。

```
su <管理者ユーザー>
sudo systemctl restart postgresql
sudo systemctl enable postgresql
```

#### 2.5. 外部からDBへのアクセスを許可する場合の設定

先に言ってよ。3.のログ出力と一緒にやりたかった・・・。

##### 2.5.1. DBへのアクセス許可

```
$ sudo su - postgres
$ nano /etc/postgresql/16/main/postgresql.conf
```

```
# - Connection Settings -
listen_addresses = '*'  # what IP address(es) to listen on;
port = 5432             # (change requires restart)
```

##### 2.5.2. IPアドレスの範囲を指定

```
$ sudo su - postgres
$ nano /etc/postgresql/16/main/pg_hba.conf
```

IPv4 に追記

```
# IPv4 local connections:
host    all             all             192.168.1.0/24          scram-sha-256
```

##### 2.5.3. サーバー再起動

```
su <管理者ユーザー>
sudo systemctl restart postgresql
```

### 3. プリザンターのセットアップ

#### 3.1. アプリケーションの準備

##### 3.1.1. ルートディレクトリ作成

```
$ sudo mkdir /web
```

##### 3.1.2. ダウンロード

[ダウンロードセンター](https://pleasanter.org/dlcenter)からプリザンター最新バージョンをダウンロード（要メアド）。  
今回のバージョンは、1.4.15.0

##### 3.1.3. 解凍して /web にコピー

解凍すると、pleasanter フォルダと、Readme.pdf があるので、フォルダ丸ごとをコピー。

##### 3.1.4. フォルダの権限変更

```
sudo chown -R <プリザンターを起動するユーザ> /web/pleasanter
```

とりあえず、OSログイン用のユーザーで・・・。

#### 3.2. データベースの構成

```
$ sudo nano /web/pleasanter/Implem.Pleasanter/App_Data/Parameters/Rds.json
```

データベースへの接続情報を設定

SaConnectionString のPWDだけ設定。
設定値は、2.2.3.で設定した値。

他、"OwnerConnectionString", "UserConnectionString" のパスワードも変更するべしだが、まずはデフォルトのまま。

#### 3.3. CodeDefinerの実行

ver.1.4.6. 以降の手順で実行

```
$ cd /web/pleasanter/Implem.CodeDefiner
$ sudo -u <プリザンターを起動するユーザ> /usr/local/bin/dotnet Implem.CodeDefiner.dll _rds /l "ja" /z "Asia/Tokyo"
```

ここでエラー。  
あれ？

```
<ERROR> Starter.Main: UnhandledException : Object reference not set to an instance of an object.
   at Implem.Libraries.Classes.XlsIo.ReadXls(String name) in C:\Implem\Pleasanter.NetCore\Implem.Libraries\Classes\XlsIo.cs:line 43
   at Implem.Libraries.Classes.XlsIo..ctor(String path, String name) in C:\Implem\Pleasanter.NetCore\Implem.Libraries\Classes\XlsIo.cs:line 18
   at Implem.DefinitionAccessor.Initializer.DefinitionFile(String name) in C:\Implem\Pleasanter.NetCore\Implem.DefinitionAccessor\Initializer.cs:line 728
   at Implem.DefinitionAccessor.Def.ConstructCodeDefinitions() in C:\Implem\Pleasanter.NetCore\Implem.DefinitionAccessor\Def.cs:line 1643
   at Implem.DefinitionAccessor.Def.SetCodeDefinition() in C:\Implem\Pleasanter.NetCore\Implem.DefinitionAccessor\Def.cs:line 261
   at Implem.DefinitionAccessor.Initializer.SetDefinitions() in C:\Implem\Pleasanter.NetCore\Implem.DefinitionAccessor\Initializer.cs:line 707
   at Implem.DefinitionAccessor.Initializer.Initialize(String path, String assemblyVersion, String setLanguage, String setTimeZone, Boolean codeDefiner, Boolean pleasanterTest, Boolean setSaPassword, Boolean setRandomPassword) in C:\Implem\Pleasanter.NetCore\Implem.DefinitionAccessor\Initializer.cs:line 47
   at Implem.CodeDefiner.Starter.Main(String[] args) in C:\Implem\Pleasanter.NetCore\Implem.CodeDefiner\Starter.cs:line 53
```

ダウンロードしたものがまずかったか？  
タイムアップなので、今回はココまで。  
