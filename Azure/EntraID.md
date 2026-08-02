# Entra ID について
2026/03/28より

参考：  
- [タメシカタ Microsoft Entra ID - 基本編](https://zenn.dev/takuyaot/books/45d4f4494a63ce/viewer/aff69e)
- [ナガシヨミ Microsoft 365 - Microsoft Entra ID 概要](https://www.docswell.com/s/takuyaot/5DE47L-entraid)

マイルストーン：  
- Entra ID によるWindowsログイン
- クラウド上のActive Directory と、Entra IDどちらの管理がよいか


## Entra ID とは
Microsoft Entra IDとは、旧 Azure ID のことを指し、クラウドベースのIDおよびアクセス管理サービスのことである。  
IDaaSとして提供され、基本機能は以下の通り。
　- ユーザーとグループの管理
  - オンプレミスディレクトリ同期
  - 基本レポート
  - クラウドユーザー向けのセルフサービスのパスワード変更
  - Azure、Microsoft365等の一般的なSaaSアプリ全体のシングルサインオンを提供

Microsoft のクラウドサービスを利用する場合必須となる。

エディションは以下の3つ
- Free (無償)
- P1 (有償)
- P2 (有償)

### Active Directory との比較

||Active Directory|Entra ID|
|--|--|--|
|基本技術|- LDAP<br>- Kerberos<br>- NTLM|- OAuth 2.0<br>- Open ID Connect<br>- SAML<br>- WS-Fed|
|基本機能|オンプレミスで使われる認証のサポート|クラウドで使われる認証のサポート|
|機能|- Microsoft製ディレクトリサービス<br>- オンプレミス型ID管理<br>- オンプレミスシステムと連携<br>- Windowsの管理(グループポリシー)|- Microsoft製IDaaS<br>- クラウド型ID管理<br>- SaaSサービスと連携<br>- デバイスのポリシー機能なし（Intuneで提供）|

目的は似て非なることがわかる。



## 各種設定方法



