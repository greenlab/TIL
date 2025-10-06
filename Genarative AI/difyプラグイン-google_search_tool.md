# Difyプラグイン: Google Search Tool 作成ガイド

このドキュメントは、Google Cloud Platform (GCP) のCustom Search JSON APIを使ったGoogle検索ツールをDifyプラグインとして作成する全手順をまとめています。APIキーは埋め込み型で、ユーザー入力不要としますが、ソース内への直接埋め込みである点、留意が必要です。Difyの環境設定変更（署名検証無効化）も含めています。

**最終更新**: 2025年10月3日  
**作者**: t.miyamoto

## ステップ1: GCPのセットアップ
1. **Programmable Search Engineを作成**:
   - [Programmable Search Engine](https://programmablesearchengine.google.com/controlpanel/create) にアクセス。
   - 新しい検索エンジンを作成（ウェブ全体を検索）。
   - **Search Engine ID (cx)** をメモ（例: `0123456789abcdef`）。

   なお、`https://cse.google.com/cse?cx=(Search Engine ID (cx))` で、作成した検索エンジンによるWeb検索ができます。

2. **APIキーを取得**:
   - [Google Cloud Console](https://console.cloud.google.com/apis/credentials) にアクセス。
   - 「認証情報」→「APIキーを作成」→「Custom Search JSON API」を有効化。
   - **APIキー** をメモ（例: `AIzaSyB...`）。

3. **制限設定（セキュリティのため）**:
   - APIキー選択 → 「API制限」で「Custom Search JSON API」のみ許可。
   - 「アプリケーション制限」でIPアドレス（サーバーIP）またはHTTPリファラ（例: `http://localhost:5000/*`）を設定。  …　**本番時要対応**
   - 無料枠: 1日100クエリ。

## ステップ2: Dify CLIのインストール
1. [Dify GitHubリリース](https://github.com/langgenius/dify/releases) からCLIツールをダウンロード（例: `dify-plugin-linux-amd64`）。
2. 実行権限を付与:
   ```bash
   chmod +x dify-plugin-linux-amd64
   ```
3. ~~別名設定:~~　
   ```bash
   alias dify='./dify-plugin-linux-amd64'
   ```
   便利ですが、使用頻度少ないため、実施せず。

## ステップ3: プロジェクトの初期化
1. プロジェクトフォルダ作成:
   ```bash
   mkdir google_search_tool
   ```
   使える文字種は、小文字英数字とアンダーバー( _ )のみ。  
   ※ハイフン( - )も使用できるが、Pythonモジュールではハイフンが使用できないため、避けたほうが良い。 

2. 初期化:

    Dify CLI が保存されているフォルダ上から、コマンド実施すること。  
   ```bash
   ./dify-plugin-linux-amd64 plugin init ./google_search_tool
   ```
   - プロンプト入力:
     - Plugin name(プラグイン名): `google_search_tool`
     - Author(作者): `t-miyamoto`
     - Description(説明): `GCP経由のGoogle検索ツール`
     - Repository URL(リポジトリ): `(Enter)` 
     - multilingual(言語): `English, 日本語` を選択。Tabキーで選択可否を切替
     - language(開発言語): `python` を選択
     - Plugin types(タイプ): `tool` を選択
     - Permission(権限): `Tools, Apps, Storage, Endpoints` を選択  
     ※Permission の Apps や Endpoints が何のために必要かは考慮せずに選択している。
     - Minimal Dify version(Difyの必要バージョン): `(Enter)`

3. 仮想環境作成:
   ```bash
   cd ./google_search_tool
   python3 -m venv venv
   source venv/bin/activate
   ```
   - 初期化により作成された `requirements.txt` に、`requests` を加える。  
     `requirements.txt`
     ```
     dify_plugin>=0.4.0,<0.7.0
     requests
     ```
   (補足) 仮想環境を中止する方法
    ```bash
    deactivate
    ```

4. ライブラリインストール:
    ```bash
    pip -r install requirements.txt
    ```

## ステップ4: ファイル設定
プロジェクト構造:
```
google_search_tool/
├── manifest.yaml
├── provider/
│   ├── google_search_tool.yaml
│   └── google_search_tool.py
├── tools/
│   ├── google_search_tool.yaml
│   └── google_search_tool.py
├── _assets/
│   ├── icon.svg
│   └── icon-dark.svg
├── readme/
│   └── README_ja_JP.md
├── PRIVACY.md
├── requirements.txt
├── .env
├── .env.example
└── README.md
```

1. **manifest.yaml**:
   ```yaml
   version: 0.0.1
   type: plugin
   author: t-miyamoto
   name: google_search_tool
   label:
     en_US: google_search_tool
     ja_JP: google_search_tool
     zh_Hans: google_search_tool
     pt_BR: google_search_tool
   description:
     en_US: GCP経由のGoogle検索ツール
     ja_JP: GCP経由のGoogle検索ツール
     zh_Hans: GCP経由のGoogle検索ツール
     pt_BR: GCP経由のGoogle検索ツール
   icon: icon.svg
   icon_dark: icon-dark.svg
   resource:
     memory: 268435456
     permission:
       tool:
         enabled: true
       endpoint:
         enabled: true
       app:
         enabled: true
       storage:
         enabled: true
         size: 1048576
   plugins:
     tools:
       - provider/google_search_tool.yaml
   meta:
     version: 0.0.1
     arch:
       - amd64
       - arm64
     runner:
       language: python
       version: "3.12"
       entrypoint: main
     minimum_dify_version: null
   created_at: 2025-10-03T21:25:41.079982596+09:00
   privacy: PRIVACY.md
   verified: false
   ```

2. **provider/google_search_tool.yaml**:
   ```yaml
   identity:
   author: "t-miyamoto"
   name: "google_search_tool"
   label:
     en_US: "google_search_tool"
     zh_Hans: "google_search_tool"
     pt_BR: "google_search_tool"
     ja_JP: "google_search_tool"
   description:
     en_US: "GCP経由のGoogle検索ツール"
     zh_Hans: "GCP経由のGoogle検索ツール"
     pt_BR: "GCP経由のGoogle検索ツール"
     ja_JP: "GCP経由のGoogle検索ツール"
   icon: "icon.svg"

   #########################################################################################
   # If you want to support OAuth, you can uncomment the following code.
   #########################################################################################
   # oauth_schema:
   #   client_schema:
   　　〜中略〜
   #         zh_Hans: "Access Token"
   #         en_US: "Access Token"

   tools:
     - tools/google_search_tool.yaml
   extra:
     python:
       source: provider/google_search_tool.py
   ```
   今回は変更していない。  
   ※プラグインに対し、APIキー等の引数を設定したい場合、変更する必要あり。

3. **provider/google_search_tool.py**:
   ```python
   from typing import Any

   from dify_plugin import ToolProvider
   from dify_plugin.errors.tool import ToolProviderCredentialValidationError
   from tools.google_search_tool import GoogleSearchToolTool  # 追加

   class GoogleSearchToolProvider(ToolProvider):

       def get_tools(self):                   # 追加
           return [GoogleSearchToolTool]      # 追加
           
       def _validate_credentials(self, credentials: dict[str, Any]) -> None:
           try:
               pass
               """
               IMPLEMENT YOUR VALIDATION HERE
               """
           except Exception as e:
               raise ToolProviderCredentialValidationError(str(e))
   ```

4. **tools/google_search_tool.yaml**:
   ```yaml
    identity:
      name: "google_search_tool"
      author: "t-miyamoto"
      label:
        en_US: "google_search_tool"
        zh_Hans: "google_search_tool"
        pt_BR: "google_search_tool"
        ja_JP: "google_search_tool"
    description:
      human:
        en_US: "GCP経由のGoogle検索ツール"
        zh_Hans: "GCP経由のGoogle検索ツール"
        pt_BR: "GCP経由のGoogle検索ツール"
        ja_JP: "GCP経由のGoogle検索ツール"
      llm: "GCP経由のGoogle検索ツール"
    parameters:
      - name: query
        type: string
        required: true
        label:
          en_US: Query string
          zh_Hans: 查询语句
          pt_BR: Query string
          ja_JP: クエリ文字列
        human_description:
          en_US: "GCP経由のGoogle検索ツール"
          zh_Hans: "GCP経由のGoogle検索ツール"
          pt_BR: "GCP経由のGoogle検索ツール"
          ja_JP: "GCP経由のGoogle検索ツール"
        llm_description: "GCP経由のGoogle検索ツール"
        form: llm
    extra:
      python:
        source: tools/google_search_tool.py

    # 仮置き
   name: google_search
   label:
     en_US: "Google Search"
     zh_Hans: "Google 搜索"
     pt_BR: "Pesquisa do Google"
     ja_JP: "Google検索"
   description:
     en_US: "Perform a Google web search via GCP Custom Search API and return results."
     zh_Hans: "通过 GCP Custom Search API 执行 Google 网页搜索并返回结果。"
     pt_BR: "Execute uma pesquisa na web do Google via API de Pesquisa Personalizada do GCP e retorne os resultados."
     ja_JP: "GCP Custom Search API経由でGoogleウェブ検索を実行し、結果を返します。"
   parameters:
     - name: query
       type: string
       required: true
       label:
         en_US: "Search Query"
         zh_Hans: "搜索查询"
         pt_BR: "Consulta de Pesquisa"
         ja_JP: "検索クエリ"
       description:
         en_US: "The search query string to send to Google."
         zh_Hans: "发送到 Google 的搜索查询字符串。"
         pt_BR: "A string de consulta de pesquisa para enviar ao Google."
         ja_JP: "Googleに送信する検索クエリ文字列。"
       placeholder:
         en_US: "e.g., Dify plugin"
         zh_Hans: "例如，Dify 插件"
         pt_BR: "ex.: plugin Dify"
         ja_JP: "例: Difyプラグイン"
     - name: num_results
       type: integer
       required: false
       label:
         en_US: "Number of Results"
         zh_Hans: "结果数量"
         pt_BR: "Número de Resultados"
         ja_JP: "結果数"
       description:
         en_US: "Number of search results to return (default: 10, max: 10)."
         zh_Hans: "要返回的搜索结果数量（默认：10，最大：10）。"
         pt_BR: "Número de resultados de pesquisa a retornar (padrão: 10, máx: 10)."
         ja_JP: "返す検索結果の数（デフォルト: 10、最大: 10）。"
       default: 10
       placeholder:
         en_US: "e.g., 5"
         zh_Hans: "例如，5"
         pt_BR: "ex.: 5"
         ja_JP: "例: 5"
   ```

5. **tools/google_search_tool.py**（最新SDK対応）:
   ```python
    from collections.abc import Generator
    from typing import Any
    import requests
    from dify_plugin import Tool
    from dify_plugin.entities.tool import ToolInvokeMessage
    from dify_plugin.entities.invoke_message import InvokeMessage
    import os

    class GoogleSearchToolTool(Tool):

        def _invoke(self, tool_parameters: dict[str, Any]) -> Generator[ToolInvokeMessage, None, None]:
            # APIキーを直接埋め込み
            api_key = "(Google Cloud Platform 上で設定した API Key(ステップ1 2.))"
            # 環境変数を使用する場合
            # api_key = os.getenv("GOOGLE_API_KEY")
            if not api_key:
                raise Exception("Google APIキーが設定されていません。")

            # パラメータ取得
            query = tool_parameters.get("query", "")
            if not query:
                raise Exception("検索クエリは空にできません。")

            num_results = tool_parameters.get("num_results", 10)
            num_results = max(1, min(10, num_results))      # 1-10に制限

            # Search Engine ID (cx) を直接埋め込み
            cx = "(Search Engin ID (CX)(ステップ1 1.))"
            # 環境変数を使用する場合
            # cx = os.getenv("GOOGLE_CX")
            if not cx:
                raise Exception("Search Engine ID (Google CX) が設定されていません")
            
            # APIリクエスト構築
            url = "https://www.googleapis.com/customsearch/v1"
            params = {
                "key": api_key,
                "cx": cx,
                "q": query,
                "num": num_results
            }

            try:
                response = requests.get(url, params=params)
                response.raise_for_status()
                data = response.json()

                # 結果をフォーマット（日本語対応）
                results = []
                if "items" in data:
                    for item in data["items"]:
                        results.append(f"タイトル： {item['title']}\n概要： {item['snippet']}\nリンク: {item['link']}\n")
                
                message_text = "\n\n".join(results) if results else "検索結果が見つかりませんでした。"
                
                yield ToolInvokeMessage(
                    type=ToolInvokeMessage.MessageType.TEXT,
                    message=InvokeMessage.TextMessage(text=message_text)
                )
            
            except requests.RequestException as e:
                error_text = f"Google API呼び出しエラー: {e}"
                yield ToolInvokeMessage(
                    type=ToolInvokeMessage.MessageType.TEXT,
                    message=InvokeMessage.TextMessage(text=error_text)
                )
            except Exception as e:
                error_text = f"検索処理エラー: {e}"
                yield ToolInvokeMessage(
                    type=ToolInvokeMessage.MessageType.TEXT,
                    message=InvokeMessage.TextMessage(text=error_text)
                )



   # 仮置き
   from collections.abc import Generator
   from typing import Any
   import requests
   from dify_plugin import Tool
   from dify_plugin.entities.tool import ToolInvokeMessage
   from dify_plugin.entities.message import TextMessage
   import os

   class GoogleSearchTool(Tool):
       def _invoke(self, tool_parameters: dict[str, Any]) -> Generator[ToolInvokeMessage, None, None]:
           api_key = os.getenv("GOOGLE_API_KEY")
           if not api_key:
               raise Exception("Google APIキーが環境変数（GOOGLE_API_KEY）に設定されていません。")

           query = tool_parameters.get("query", "")
           if not query:
               raise Exception("検索クエリは空にできません。")

           num_results = tool_parameters.get("num_results", 10)
           num_results = max(1, min(10, num_results))

           cx = os.getenv("GOOGLE_CX")
           if not cx:
               raise Exception("Search Engine ID (GOOGLE_CX) が環境変数に設定されていません。")

           url = "https://www.googleapis.com/customsearch/v1"
           params = {
               "key": api_key,
               "cx": cx,
               "q": query,
               "num": num_results
           }

           try:
               response = requests.get(url, params=params)
               response.raise_for_status()
               data = response.json()

               results = []
               if "items" in data:
                   for item in data["items"]:
                       results.append(f"タイトル: {item['title']}\n概要: {item['snippet']}\nリンク: {item['link']}\n")
               message_text = "\n\n".join(results) if results else "検索結果が見つかりませんでした。"

               yield ToolInvokeMessage(
                   type=ToolInvokeMessage.MessageType.TEXT,
                   message=TextMessage(text=message_text)
               )
           except requests.RequestException as e:
               error_text = f"Google API呼び出しエラー: {e}"
               yield ToolInvokeMessage(
                   type=ToolInvokeMessage.MessageType.TEXT,
                   message=TextMessage(text=error_text)
               )
           except Exception as e:
               error_text = f"検索処理エラー: {e}"
               yield ToolInvokeMessage(
                   type=ToolInvokeMessage.MessageType.TEXT,
                   message=TextMessage(text=error_text)
               )
   ```

7. **.gitignore**:

   デフォルトのまま使用

8. **アイコンファイル（仮）**:
   ```bash
   echo '<svg width="100" height="100"><rect width="100" height="100" fill="#4285f4"/></svg>' > icon.svg
   echo '<svg width="100" height="100"><rect width="100" height="100" fill="#ffffff"/></svg>' > icon-dark.svg
   ```

9. **PRIVACY.md**:  
  参考
   ```
   ## プライバシーポリシー
   このプラグインはユーザーのクエリデータを保存しません。Google API経由の検索結果のみを返します。
   ```

## ステップ5: Dify環境設定変更（署名検証無効化） 重要！
セルフホスト版Difyの場合、署名エラーを回避するため:
1. Difyインストールフォルダ（例: `~/dify`）に移動:
   ```bash
   cd ~/dify
   ```
2. `.env` を編集:
   ```bash
   nano .env
   ```
   - 末尾に追加:
     ```
     FORCE_VERIFYING_SIGNATURE=false
     ```
3. Dify再起動:
   ```bash
   docker compose down
   docker compose up -d
   ```

(補足) クラウド版の場合: サポートに問い合わせ、またはセルフホスト版に移行。

## ステップ6: テスト
1. **ローカルサーバー起動**:
   ```bash
   source venv/bin/activate
   python3 -m main
   ```
   ローカルサーバーは、`Flask`により起動。  
   なお、`Flask`とはPythonでWebアプリケーションを開発する際に利用する軽量なWebフレームワークのこと。

2. **接続確認**:
   ```bash
   curl http://localhost:5000/
   ```
    接続できない場合の確認。  
    1. ポートが使用されているか
        ```bash
        lsof -i :5000
        ```
    2. `Fask`サーバーが有効か確認する。
        ```Python
        # test_server.py
        from flask import Flask
        app = Flask(__name__)

        @app.route('/')
        def hello():
            return {"message": "Test server"}

        if __name__ == "__main__":
            app.run(host="0.0.0.0")
        ```
        仮想環境上で実行
        ```bash
        pip install flask
        python3 test_server.py
        ```
        コンソールを起動して実行
        ```bash
        curl -v http://localhost:5000

        * Host localhost:5000 was resolved.
        * IPv6: ::1
        * IPv4: 127.0.0.1
        *   Trying [::1]:5000...
        * connect to ::1 port 5000 from ::1 port 43854 failed: 接続を拒否されました
        *   Trying 127.0.0.1:5000...
        * Connected to localhost (127.0.0.1) port 5000
        > GET / HTTP/1.1
        > Host: localhost:5000
        > User-Agent: curl/8.5.0
        > Accept: */*
        > 
        < HTTP/1.1 200 OK
        < Server: Werkzeug/3.1.3 Python/3.12.3
        < Date: Mon, 06 Oct 2025 07:28:32 GMT
        < Content-Type: application/json
        < Content-Length: 26
        < Connection: close
        < 
        {"message":"Test server"}
        * Closing connection
        ```

3. **パッケージ化**:  
  `dify-plugin-linux-amd64` があるフォルダにカレントディレクトリを移してから
   ```bash
   ./dify-plugin-linux-amd64 plugin package ./google_search_tool
   ```

4. **Difyインストール**:
   - ダッシュボード → 「プラグイン」 → 「Local Upload」 → .difypkgアップロード。
   - インストール後、ワークフローでテスト（query: "Dify プラグイン"）。

