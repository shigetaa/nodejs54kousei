# 簡単な文章校正ツールを作ろう

形態素解析の利用例として、簡単な文章校正ツールを制作してみます。
単語数のカウントや助詞「の」の連続や括弧の非対応、文字種の不整合等を指摘するツールを作ります。

## 文章校正ツールについて
大なり小なり、日々の生活の中で文章を書く機会があります。
業務報告書や、商品の宣伝文など多くの機会があります。
そこで、活用したいのが文章の校正ツールです。
ここでは、簡単な形態素解析の利用例として文章の校正ツールを作っていきます。

### ここで作成するプログラム

ここで作成するのは、文章校正ツールです。
文章の書き間違い、文法の間違い、表記の揺れをチェックします。
テキストファイルをプログラムの引数として渡すと、テキストを解析して
校正点を指摘するというものにします。

校正ポイント
- 助詞「の」連続
- 長すぎる文章
- 並列助詞「～したり、～したり」が対応してない場合
- 同じ接続詞が連続で使われている場合
- 「いい訳」と「言い訳」など表記の揺れ
- 「プログラマー」と「プログラマ」など音引きの有無の相違

### 実際のプログラム