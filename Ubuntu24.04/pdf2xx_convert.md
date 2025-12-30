# PDFを他のファイル形式に変換する

2025/06/04

## 目的

RAG環境に突っ込むドキュメントを構造化するために、ある程度のテキスト化が必要と考える。
ドキュメントの大半は、PDFもしくはWord、Excel、PowerPointである。

とりあえず、手軽そうに写ったPDFをtextファイルないし、html、markdown形式にコンバートさせよう。

その過程を経た上で、Word等にも着手しよう。

## PDFからxx への変換

### PANDOC

参考： https://qiita.com/sky_y/items/3c5c46ebd319490907e8  
　　　 https://qiita.com/sky_y/items/15bf7737f4b37da50372  
　　　 https://qiita.com/sky_y/items/80bcd0f353ef5b8980ee  
　　　 https://pandoc-doc-ja.readthedocs.io/ja/latest/users-guide.html  
　　　 https://wkhtmltopdf.org/  

#### 1. インストール
```
$ sudo apt -y install pandoc
```

特に波乱起きず

#### 2. 変換実行

```
$ pandoc 001496685.pdf -s -o 001496685.html
Unknown input format pdf
Pandoc can convert to PDF, but not from PDF.
```

あれ？

#### 3. 変換結果

変換出来ず。説明サイトをよく読んだら、PDFの読み込みと生成はできないとのこと。  
なんてこった。

#### 4. 補足

必要かな？と思い、wkhtmltopdf をインストールしたが、そもそもPDFの読み込みができないため、意味がなかった。  
wkhtmltopdf は、html から PDF を生成するためのアプリ  
一応、インストールコマンド

```
$ sudo apt -y install wkhtmltopdf
```

### PDFtoText

参考： https://www.adobe.com/jp/acrobat/hub/how-to-convert-pdf-to-text-in-linux.html  

#### 1. インストール
```
$ sudo apt -y install poppler-utils
```

#### 2. 変換実行

```
$ pdftotext -layout 001496685.pdf 001496685.txt
```

-layout を付与することで、レイアウトを考慮した形で出力する（らしい）

#### 3. 変換結果

文字列部分は特に問題なし。  
部分部分で、文字化けが発生している。  
ただ、表型式や図に相当する箇所で、大きなレイアウトズレが発生する。

### PDFtoHTML

Textだと、それはそうかと思い、HTML型式に変換することにする。  
先ほどと同じシリーズから…。  

参考： https://blog.k-bushi.com/post/tech/tips/convert-html-to-pdf/  
　　　 https://pdf-file.nnn2.com/?p=884  

#### 1. インストール
```
$ sudo apt -y install poppler-utils
```

#### 2. 変換実行

```
$ pdftohtml 001496685.pdf ./output/001496685
```
出力先をカレントのままにで処理すると、大変なことになる（なった）。

#### 3. 変換結果

文字列部分は特に問題なし。  
部分部分で、文字化けが発生している。  
ただ、表型式や図に相当する箇所で、大きなレイアウトズレが発生する。  
フレームが付いたり、リンクもするが、文字列の切り出しはPDFtoTEXTと同じような感じ。  

### PDF2HTMLEX

参考：　https://pdf-file.nnn2.com/?p=883  

#### 1. インストール
波乱あり試行錯誤する。たぶん、下記の方法でよい。

1. インストーラをダウンロード

https://github.com/pdf2htmlEX/pdf2htmlEX/releases/tag/v0.18.8.rc1  

pdf2htmlEX-0.18.8.rc1-master-20200630-Ubuntu-bionic-x86_64.deb  をダウンロード

```
$ cd ~/Downloads
$ sudo apt install ./pdf2htmlEX-0.18.8.rc1-master-20200630-Ubuntu-bionic-x86_64.deb
   ：
注意、'./pdf2htmlEX-0.18.8.rc1-master-20200630-Ubuntu-bionic-x86_64.deb' の代わりに 'pdf2htmlex' を選択します
以下のパッケージが新たにインストールされます:
  pdf2htmlex
   ：
N: ファイル '/home/mi/Downloads/pdf2htmlEX-0.18.8.rc1-master-20200630-Ubuntu-bionic-x86_64.deb' がユーザ '_apt' からアクセスできないため、ダウンロードは root でサンドボックスを通さずに行われます。 - pkgAcquire::Run (13: 許可がありません)
```

まあ、とりあえず…。

変換実行と変換結果を、複数回表示します。

#### 2. 変換実行（1）

注） pdf2htmlex だと動きません。 **pdf2htmlEX** です。※EXが大文字
```
$ mkdir output2
$ pdf2htmlEX 001496685.pdf ./output2/001496685
Preprocessing: 113/113
ToUnicode CMap is not valid and got dropped for font: 6
Working: 113/113
```
文字コード系のエラーが出るも変換できた。

#### 3. 変換結果（1）

PDFと見た目がまんまそっくりなHTMLが生成される。  
ただ、1つのHTMLファイルに全てが詰まっている。  
背景イメージもHTMLファイル内で、BASE64かSVGで存在。  
ちょっと気持ちが悪いぞと、CSSやイメージ等を分離させることにする。

#### 2. 変換実行（2）

```
$ cd ..
$ mkdir output3
$ cd output3
$ pdf2htmlEX ../001496685.pdf 001496685.html --embed cfijo
Preprocessing: 113/113
ToUnicode CMap is not valid and got dropped for font: 6
Working: 113/113
```
--embed cfijo は、CSS等を外部ファイル化するためのオプション

#### 3. 変換結果（2）

PDFと見た目がまんまそっくりなHTMLが生成される。  
HTMLとCSS、イメージ類は別ファイルになった。  
ただ、文字ごとに CLASS タブが発生している。なんぞこれ。  

ここで、HTMLからTXTやMDに変換すれば、いいことあるのではと、PANDOCを実行する。
```
$ pandoc 001496685.html -s -o 001496685.md
$ pandoc 001496685.html -s -o 001496685.txt
```
CLASS タブ乱立の影響で、まともに読めない。

#### 2. 変換実行（3）

```
$ cd ..
$ mkdir output4
$ cd output4
$ pdf2htmlEX ../001496685.pdf 001496685.html --embed cfijo --process-nontext 0 --process-outline 0
Preprocessing: 113/113
ToUnicode CMap is not valid and got dropped for font: 6
Working: 113/113
```
 --process-nontext 0 は、イメージ等の非テキストオブジェクトを含めないためのオプション  
 --process-outline 0 は、アウトライン表示を行わないオプション  

#### 3. 変換結果（3）

PDFと見た目がまんまそっくりなHTMLが生成される。  
HTMLとCSSは別ファイルになった。  
イメージがなくなった分、ページによっては見た目が変わった。  
ただ、肝心の文字ごとに CLASS タブが発生している件は変わっていない。  

#### 2. 変換実行（4）

```
$ cd ..
$ mkdir output5
$ cd output5
$ pdf2htmlEX ../001496685.pdf 001496685.html --embed cfijo --process-nontext 0 --process-outline 0 --printing 0 --fallback 0
Preprocessing: 113/113
ToUnicode CMap is not valid and got dropped for font: 6
Working: 113/113
```
 --printing 0 は、印刷を可能にしないことで、CSSを小さくするオプション  
 --fallback 0 は、フォールバックモードを無効にするオプション  
 フォールバックモードでの出力は、優れた精度とブラウザの互換性の為ですが、サイズが大きくなります。  

#### 3. 変換結果（4）

変換結果（3）と、あまり変わらない。

### MarkItDown

一旦、PDF2HTMLEX から離れる。  
次は、Microsoftによる OSS で、いろんなフォーマットのファイルを MarkDown にするというものを試す。

参考： https://zenn.dev/nyagato_00/articles/719b8c4749365f  
　　　 https://blog.future.ad.jp/microsoft-markitdown-pdf-markdown  
　　　 https://github.com/microsoft/markitdown  

#### 1. インストール
試行錯誤の結果を記す（実際のインストール順と異なる

```
// PYTHON上での仮想環境を作成する。よくわかっていない。
$ sudo apt install python3-venv
// 動画や画像等のマルチエンコーダ。PDFによってはエラーが出るため。
$ sudo apt install fmpeg

// 仮想環境作成
$ python3 -m venv .venv
// 仮想環境に切り替え
$ source .venv/bin/activate
// PYTHON用のパッケージツールをインストールして最新化
$ python3 -m pip install --upgrade pip
// 本命をインストール ※ [all] を付けること
$ pip3 install markitdown[all]
```

#### 2. 変換実行

```
$ markitdown 001496685.pdf -o ./md1/001496685.md
```

#### 3. 変換結果

markdown 的な要素なし。
文字列の切り出しはPDFtoTEXTと同じような感じ。  

### その他試していない

* https://qiita.com/vko/items/04fb0756abd89dff8573

  PYTHONのライブラリなので単独で動かない。

* https://zenn.dev/kun432/scraps/c87a2570953747

  ちゃんと読めていないけど、敷居が高そう。





