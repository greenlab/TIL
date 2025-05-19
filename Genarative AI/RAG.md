とりあえず、調べたこと、琴線に触れたことをほぼメモの形で記します。

# RAGについて

## OSSでRAGを実装するには

* RAGapp 
[オープンソースのRAGアプリ「RAGapp」を試す](https://zenn.dev/kun432/scraps/b0266a88584b9f)
* RAGFlow
[オープンソースのRAGアプリ「RAGFlow」を試す](https://zenn.dev/kun432/scraps/5b8547c6aa1c95)
* Dify
[Tanuki-8BとOllamaとDifyを使って日本語ローカルRAG構築](https://zenn.dev/mkj/articles/93dbd6c9d94c58)

見る限り、Dify＞RAGapp＞RAGFlowの順か。

## Ollama

いろいろわかっていないけど、とりあえず記事をスクラップ

[Ollama + Open WebUI でローカルLLMを手軽に楽しむ](https://zenn.dev/karaage0703/articles/c271ca65b91bdb)  
[Raspberry Pi（ラズパイ）のローカル環境でLLMを動かす](https://zenn.dev/karaage0703/articles/5fd411a9358898)

## Agetic RAG とは

RAGも色々と進化しているらしい。

[Agentic RAGとは？深層思考型フレームワークの仕組み【コード解説付き】](https://zenn.dev/givery_ai_lab/articles/5c684e9fefa9e6)

## Dify を深堀りしてみる。

元ネタはコレ  
[Tanuki-8BとOllamaとDifyを使って日本語ローカルRAG構築](https://zenn.dev/mkj/articles/93dbd6c9d94c58)

1. wsl をインストールする（本当に必要かは不明）
```
wsl --install
```
2. wsl をシャットダウンする
```
wsl --shutdown
```
3. wsl で使用するディストリビューションを確認して、インストールする。  
Nameの方の名前を入れよう。
```
wsl --list --online
wsl --install -d Ubuntu-22.04
```
4. 管理者権限となるユーザー名とパスワードを登録（忘れないように…）
5. 必要に応じて、メモリの確保を行う。（たいてい必要になる  
[WSL2でのメモリ不足](https://zenn.dev/karaage0703/articles/d38e17bd6efbaa)
6. wsl 上で docker をインストールする。バージョン情報が出れば問題なし。
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
docker -v
```
その他、docker については、とりあえず、下記で。  
[Dockerを使った機械学習環境の構築方法](https://zenn.dev/mkj/articles/33befbaf38c693)　　

7. ollama インストール
ちょっと、このあたりゴタゴタしたような…。  
```
curl -fsSL https://ollama.com/install.sh | sh
```
8. docker から ollama 起動  
よく起動していますと、失敗するので、おまじないでコレしてます。
それぞれのプロセス止めて、イメージを削除して…はわかるが、その理由等の部分は別途勉強しないと。  
```
docker stop $(docker ps -q)
docker rm $(docker ps -q -a)
```
GPUなしなので
```
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```
9. 日本語に強いらしい Tanuki さんのモデルをダウンロードして、 ollama で動かす。
```
docker exec ollama ollama pull 7shi/tanuki-dpo-v1.0:8b-q6_K
docker exec -it ollama ollama run 7shi/tanuki-dpo-v1.0:8b-q6_K
```
10. コマンドラインベースになるけど、 ollama 上で、AIによる会話ができます。
終了するときは、 /bye  
11. Dify をインストールする。
```
cd && git clone https://github.com/langgenius/dify
```
12. docker composer  ※そもそもがよくわかっていない部分
```
cd ~/dify/docker
docker compose up -d
```
13. Webブラウザでログインする。
メールアドレスとパスワードの登録が必要になります。
```
http://localhost/install
```
14. Dify でナレッジを作成する。  
ナレッジとは、RAG で使用するデータのこと。
とりあえず健康保険法の PDF をナレッジにした。

  1. ナレッジ→ナレッジベースを作成
  2. PDF をインポート
  3. 設定　については、要調査。

思ったより簡単だった。

15. モデルの追加  

Dify で、一番はまった個所

  1.  Dify 右上のユーザー→設定→モデルプロバイダー と選択する。
  2.  ollama を選択　※最初からあったかどうか記憶が…。なければ追加
  3.  右下の「モデルを追加」をクリックする。

Model Type: LLM  
Model Name: 7shi/tanuki-dpo-v1.0:8b-q6_K　　※9. でダウンロードした際の名前
Base URL: http://(wsl上のIP):11434  
※wsl 上で、下記コマンドを実行し、127.0.0.1 の次のIPアドレスを採用すること
```
ip address
```
なお、IPが有効であれば、ブラウザ上で、http://(wsl上のIP):11434 を指定すると、「Ollama is running」と、返ってきます。  
以降の設定、とりあえずデフォルトのまま。
15. スタジオの作成
  1. スタジオ→最初から作成→チャットボット
  2. スタジオの名前と説明を登録、モデルを選択、ナレッジを選択
  3. テスト用のチャットが動けば良し。
  4. 右上の「公開する」をクリックする
  5. 「公開」をクリックし、モデルを選択する
  6. 「アプリを実行する」で立ち上がったブラウザから確認する。
  

## Dify 追記

https://www.8asue.com/fukuei/b20250423/