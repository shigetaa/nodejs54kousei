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
簡単なプログラムを書いて、その実行方法を確認していきましょう。 `kousei.js` と言う名前のファイルを作成して以下の様に記述してみます。
```javascript
var MAX_WORD = 40; // 最大単語数の警告

var Mecab = require('./mecab-mod.js');
var mecab = new Mecab();
var fs = require('fs');

// 引数をチェック
var args = process.argv;
args.shift(); // 除去 'node'
args.shift(); // 除去 スクリプトのパス
if (args.length <= 0) {
	console.log('node kousei.js textfile');
	process.exit();
}
var filename = args.shift();

// ファイルを読み込む
var text = fs.readFileSync(filename, "utf-8");
// 形態素解析を行う
mecab.parse(text, function (err, items) {
	checkSentence(items);
});

// 文章をチェックする
function checkSentence(items) {
	checkJosiNo(items);
	checkTaiou(items);
}

// 助詞の「の」の連続と単語数の長さを確認する
function checkJosiNo(items) {
	var cnt = 0; // 助詞「の」が出現した回数を数える
	var cur = []; // 現在読み込んでいる文を保存する
	var lineno = 1; // 行番号を数える
	for (var i in items) {
		var it = items[i];
		var w = it[0];
		if (w == "EOS") { // 改行
			lineno++; cur = []; cnt = 0;
			continue;
		}
		// 文末および句点の確認
		if (w == "。" || w == "、") {
			// 「の」の回数
			if (cnt >= 3) {
				console.log("[警告] 助詞「の」が" + cnt + "回連続しています。");
				console.log("\t(" + lineno + "行目)" + cur.join(""));
			}
			// 単語数を確認
			if (cur.length >= MAX_WORD) {
				console.log("[警告] 一文が長すぎます。" + cur.length + "以上の単語です。");
				console.log("\t(" + lineno + "行目)" + cur.join("|"));
			}
			cnt = 0;
			if (w == "。") { cur = []; }
			continue;
		}
		// 「の」があるか確認
		if (it[0] == "の" && it[1] == "助詞") cnt++;
		cur.push(w);
	}
}

// 対応チェック
function checkTaiou(items) {
	var heiritujosi = 0, cur = [], lineno = 1;
	var meisi = {};
	var setuzokusi = {}, oldCur = [];
	for (var i in items) {
		var it = items[i];
		var w = it[0];
		if (w == "EOS") { // 改行
			lineno++;
			setuzokusi = {};
			oldCur = cur; cur = [];
			continue;
		}
		// 文末の処理
		if (w == "。") {
			if (heiritujosi == 1) {
				console.log("[警告] 並立助詞「〜たり」が一度しか出現しません。");
				console.log("\t(" + lineno + "行目)" + cur.join(""));
			}
			oldCur = cur; cur = []; heiritujosi = 0;
			continue;
		}
		// 並立助詞「たり」のチェック
		if (it[2] == "並立助詞" && (w == "たり" || w == "だり")) {
			heiritujosi++;
		}
		// 接続詞のチェック(一行に同じ接続詞が出てこないようにする)
		if (it[1] == "接続詞") {
			if (typeof (setuzokusi[w]) == "undefined") {
				setuzokusi[w] = 1;
			} else {
				console.log("[警告] 一行に同じ接続詞「" + w + "」が複数回使われています。");
				console.log("\t(" + lineno + "行目)" + oldCur.join(""));
			}
		}
		// 表記の揺れチェック
		if (it[1] == "名詞" && w.length >= 2) {
			var kana = it[8];
			if (kana == undefined) kana = it[0]; // 辞書にない単語対策
			kana = kana.replace(/ー/g, ''); // カタカナ対策
			if (meisi[kana] == undefined) {
				meisi[kana] = w;
			} else if (meisi[kana] != w) {
				console.log("[確認] 表記の揺れ: " + meisi[kana] + " != " + w);
			}
		}
		cur.push(w);
	}
}
```

上記のプログラムで使用するモジュール`mecab-mod.js`と動作に必要な環境については下記のMeCabについてを参照してください。

[MeCabについて<br>https://github.com/shigetaa/nodejs52mecab](https://github.com/shigetaa/nodejs52mecab)

本ブログラムで使用する、文章データ`text.txt`は下記の様に記述します。

```txt
今日は朝早く起きました。天気は晴れです。

●「の」の連続
でも今日の朝食の卵焼きの醤油が辛かったのです。
学校で私の隣の席のジェニーはよく笑います。

●並立助詞の不整合
先週は、山に行ったり自然を堪能した日でした。
そこで食べたり飲んだり歌ったりしました。

●カタカナと漢字による表記揺れ
リンゴを食べたり、イチゴを食べたり、ミカンを食べたり、食べてばかりでした。
林檎は美味しい。苺も美味しい。

●「ー」による表記揺れ
Perlプログラマー対PHPプログラマの対決を見よう。
サーバーの性能を確かめよう。あのサーバは快適に使える。

●長文のチェック
毎朝しっかり食べないと元気が出ないので、学校に行く前は必ずしっかり食べるようにと、母から言われていますので、朝ご飯に魚とご飯と味噌汁が必要です。

●接続詞の重複
鮭は寿司ネタである。しかし、鮭はムニエルにもする。しかし、鮭が好きである。
```

実行するには、以下のコマンドを実行します。
```bash
node kousei.js text.txt
```
すると、以下の様に間違いを指摘します。
```bash
[警告] 助詞「の」が3回連続しています。
        (4行目)でも今日の朝食の卵焼きの醤油が辛かったのです
[警告] 助詞「の」が3回連続しています。
        (5行目)学校で私の隣の席のジェニーはよく笑います
[警告] 一文が長すぎます。40以上の単語です。
        (20行目)毎朝|しっかり|食べ|ない|と|元気|が|出|ない|ので|学校|に|行く|前|は|必ず|しっかり|食べる|よう|に|と|母|から|言わ|れ|て|い|ます|ので|朝|ご飯|に|魚|と|ご飯|と|味噌汁|が|必要|です
[警告] 並立助詞「〜たり」が一度しか出現しません。
        (8行目)先週は、山に行ったり自然を堪能した日でした
[確認] 表記の揺れ: リンゴ != 林檎
[確認] 表記の揺れ: プログラマー != プログラマ
[確認] 表記の揺れ: サーバー != サーバ
[警告] 一行に同じ接続詞「しかし」が複数回使われています。
        (23行目)しかし、鮭はムニエルにもする
```
