# 2025/04/30 pleasanter インストール Windows 11

Ubuntu 24.04/22.04 共にインストーラは失敗。
一番無難そうなWindowsで挑戦する。

## 1. Windowsの機能の有効化

Windowsの機能の有効化（Windows 10 / 11）

### 1.1. IISおよびASP.NETの有効化を行う

インターネットインフォメーションサービス
　├─ Web 管理ツール
　│　　 └─ IIS 管理コンソール
　└─ World Wide Web サービス
　　　　 ├─ HTTP 共通機能
　　　　 │　　 └─ 静的なコンテンツ
　　　　 └─ アプリケーション開発機能
　　　　　　　 └─ ASP.NET 4.8

IIS 管理コンソール、静的なコンテンツ、ASP.NET 4.xにチェックを入れる。

### 1.2. IIS の設定変更

1.2.1. IIS 起動

1.2.2. 「アプリケーションプール」の「DefaultAppPool」を右クリックし、「アプリケーションプール既定値の設定」をクリック

1.2.3. 「プロセスモデル」の「アイドルタイムアウトの操作」を、「Terminate」から「Suspend」に変更し、「OK」をクリック


## 2. .NETのインストール

.NETのインストール（Windows環境）

### 2.1. .NET8.0 のインストール

.NET8.0の「SDK 8.0.x」と「Hosting Bundle」の2つをインストールする。

2.1.1. SDKのインストール

せっかくなので、Winget経由でインストールする。

2.1.1.1. ターミナル起動（スタートでターミナルでOK）

2.1.1.2. Winget で、.NET SDK をインストール

```
PS > winget install Microsoft.DotNet.SDK.8
```

2.1.1.3. .NETのバージョン確認

ターミナルもしくはコマンドプロンプトを立ち上げなおして

```
PS > dotnet --version
8.0.408
```

2.2.2. Hosting Bundle のインストール

これはインストーラをダウンロードするしかない。
https://dotnet.microsoft.com/ja-jp/download/dotnet/8.0

2.2.2.1. インストーラダウンロード

ASP.NET Core ランタイム 8.0.xx のOS：Windowsにある 
[Hosting Bundle](https://dotnet.microsoft.com/ja-jp/download/dotnet/thank-you/runtime-aspnetcore-8.0.15-windows-hosting-bundle-installer)
 をクリックする。

2.2.2.2. dotnet-hosting-8.0.xx-win.exe を実行する。


## 3. データベースのインストール

Windows ということを考えると、SQL Server になるのだが、Ubuntu のリベンジでもあるので、PostgreSQL で挑戦する。

### 3.1. PostgreSQLのインストール（Windows OS）

3.1.1. PostgreSQL ダウンロード

https://www.enterprisedb.com/downloads/postgres-postgresql-downloads

Ubuntuの時、バージョン 16 だったが、バージョン 17.4 があったので、無謀にも上位バージョンで進めてみる。

3.1.2. PostgreSQL インストール

ダウンロードした postgresql-17.4-1-windows-x64.exe を実行。

1．C++のランタイム更新（自動）
2. Next
3. インストールディレクトリ：デフォルトでNext (C:\Program Files\PostgreSQL\17)
4. コンポーネントの選択：全部選択でNext
5. データディレクトリ：デフォルトでNext (C:\Program Files\PostgreSQL\17\data)
6. DBユーザー「postgres」のパスワード：psgr0429 でNext
7. ポート：デフォルトでNext (5432)
8. ロケール：デフォルトでNext ([Default locale])
9. Next
10. Next
11. Launch Stack Builder at exit? のチェックを外して、Finish


## 4. プリザンターのインストールおよび設定

### 4.1. プリザンターをWindowsにインストールする

4.1.1. プリザンターのダウンロード

https://pleasanter.org/dlcenter (要メールアドレス)

4.1.2. メールアドレス内のダウンロードリンクからプリザンターをダウンロード

Pleasanter_1.4.15.0.zip

4.1.3. zipを解凍。

4.1.4. C:\ 直下に、ディレクトリ"web"を作成。 c:\web  

4.1.5. c:\web 直下に、解凍したディレクトリ内のディレクトリ"pleasanter"をコピー

4.1.6. C:\web\pleasanter\Implem.Pleasanter\App_Data\Parameters\Rds.jsonにデータベースへの接続情報を設定

| 1 | Dbms | PostgreSQL |
| 2 | SaConnectionString | "Server=localhost;Port=5432;Database=postgres;UID=postgres;PWD=psgr0429" |
| 3 | OwnerConnectionString | "Server=localhost;Port=5432;Database=#ServiceName#;UID=#ServiceName#_Owner;PWD=owner0429" |
| 4 | UserConnectionString | "Server=localhost;Port=5432;Database=#ServiceName#;UID=#ServiceName#_User;PWD=user0429" |

4.1.7. CodeDefinerの実行

毎度盛大にこけている箇所までやってきました。  
さて、今回は・・・。  

```
PS > cd C:\web\pleasanter\Implem.CodeDefiner
PS > dotnet Implem.CodeDefiner.dll _rds /l "ja" /z "Tokyo Standard Time"
```

ぱちぱち！

4.1.8. IISのセットアップ

4.1.8.1. 「サーバマネージャー」の「ツール(T)」メニューを開き「インターネット インフォメーション サービス（IIS)マネージャー」を起動。

4.1.8.2. 「アプリケーションプール」の「DefaultAppPool」を右クリックし、「基本設定」をクリック

4.1.8.3. .Net CLR バージョンを、「マネージドコードなし」に変更

4.1.8.4. 左ペインより、サイト-「Default Web Site」を選択して、右ペインの詳細設定をクリック

4.1.8.5. 物理パスを、「C:\web\pleasanter\Implem.Pleasanter」と入力

4.1.8.6. 右ペインより再起動をクリック

4.1.8.7. 再起動後、右ペインの「*.80(http)参照」をクリックし、プリザンターを起動

4.1.8.8. プリザンターのログイン画面にて「ログインID: Administrator」、「初期パスワード: pleasanter」を入力し、「ログイン」ボタンをクリックします。

4.1.8.9. パスワード変更 

### 4.2. プリザンターからメールを送信できるように設定する  

当初、gmail をSMTPサーバーとして使用するよう設定したものの、メールが送信できていないようだったので、下記を基にダミーメールサーバー(smtp4dev)による方法で確認する。

https://qiita.com/implem-taguchi/items/09b059e1c7efa3255d11

#### 4.2.1. smtp4dev のダウンロードとインストール

1. [smtp4dev](https://github.com/rnwood/smtp4dev) の画面右 Releases から、最新版をクリック(今回、3.8.6)。  
2. Assets から、Rnwood.Smtp4dev-win-x64-x.x.x.zip をダウンロード
3. zipファイルを解凍
4. 解凍したzipファイルから、Rnwood.Smtp4dev.exe をダブルクリックして実行。
5. コンソールが起動し、smtp4dev が実行される。

http://localhost:5000

### 4.2.2. プリザンターの設定変更

3.2.6. 下記　Mail.json を編集する。

/web/pleasanter/Implem.Pleasanter/App_Data/Parameters/Mail.json

```
{
    "SmtpHost": "localhost",        // smtp4dev があるサイト。同じ場合は、localhost でOK
    "SmtpPort": 25,                 // SSL使わないので25
    "SmtpUserName": null,           // null
    "SmtpPassword": null,           // null
    "SmtpEnableSsl": false,         // SSL使わないので false
    "ServerCertificateValidationCallback": false,
    "SecureSocketOptions": null,
    "FixedFrom": null,
    "AllowedFrom": null,
    "SupportFrom": "\"Pleasanter.org\" <support@pleasanter.org>",
    "InternalDomains": "",
    "Encoding": null,
    "ContentEncoding": null
}
```

### 4.2.3. プリザンターサーバー再起動

IIS を起動し、「Default Web Site」を再起動




---



プリザンターのリマインダー機能を有効化する
プリザンターのデータベース(SQL Server)をバックアップする


