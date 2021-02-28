--- <!-- 翻訳: logcpp -->
layout: lecture
title: "Shell Tools and Scripting"
date: 2020-01-14
ready: true
video:
  aspect: 56.25
  id: kgII-YWo3Zw
---

この講義では、bashを一つのスクリプト言語として扱い、コマンドラインで常日頃一番行っている作業のいくつかに関連したシェルツールと併用する基礎を紹介します。

# シェルスクリプトの書き方

これまでは、シェルでコマンドを実行したり、それらをパイプで繋げたりすることを学んできました。
しかし、多くの場合では複数のコマンドを一遍に実行すること、あるいは条件分岐や繰り返しなどの制御構文を利用することが必要になってきます。

シェルスクリプトはそういったより複雑な操作を実現してくれるのです。
ほとんどのシェルにはそれら固有のスクリプト言語があり、変数や制御構造、そして固有の文法があります。
他のスクリプト言語と異なるのは、シェルスクリプトはシェル関連の作業に特化しているという点でしょう。
故に、コマンドのパイプラインを作ったり、実行結果をファイルに保存したり、標準入力から読み込んだりするのはシェルスクリプトの本領なので、汎用的なスクリプト言語よりも使い勝手が良いのです。
この章では、最もよく使われているbashのスクリプト言語に注目して学んでいきます。

bashで変数に値を代入するには、`foo=bar`という構文を使います。変数の値にアクセスするには、`$foo`を利用します。
ここで注意してほしいですが、`foo = bar`ではうまく行きません。この構文だと`foo`というプログラムを呼び出して引数に`=`と`bar`を取ると解釈されてしまうからです。
一般に、シェルスクリプトにおいてスペース記号は引数を分割する役割を果たします。この挙動は最初のうちは混乱しがちなので、常に注意を払うようにしましょう。

bashでは文字列は`'`と`"`の区切り文字で定義できます。しかしこの２つは同等ではありません。
`'`で区切った文字列はリテラルの文字列であり、`"`のように変数の値を代入してくれません。

```bash
foo=bar
echo "$foo"
# barと出力する
echo '$foo'
# $fooと出力する
```

ほとんどのプログラミング言語と同じように、bashは`if`、`case`、`while`や`for`などの制御構造の利用が可能です。
同様に、`bash`にも関数があり、引数を受け取って計算することができます。ここでは関数の一例として、ディレクトリを作り`cd`コマンドでその中に移動するものを示します。


```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

ここで`$1`はこのスクリプト/関数へ渡される一番目の引数です。
他のスクリプト言語と違って、bashは色んな変わった変数を使って引数、エラー・コードなどを指定します。以下はその一部を示したリストです。より多くの内容をまとめたリストは[ここ](https://www.tldp.org/LDP/abs/html/special-chars.html)を参照してください。
- `$0` - スクリプト名
- `$1` から `$9` - スクリプトに渡される引数。`$1`は一番目の引数、以下省略。
- `$@` - 全ての引数
- `$#` - 引数の総数
- `$?` - 直前のコマンドのリターンコード
- `$$` - 現在のスクリプトのプロセス識別子（PID）
- `!!` - 引数を含む直前のコマンド全体。よくあるパターンはパーミッションがなかっただけでコマンドを実行できなかった場合で、`sudo !!`と打てばそのコマンドをsudoで速やかに再実行できます。
- `$_` - 直前のコマンドの最後の引数。もしインタラクティブシェルに入っていれば、`Esc`からの`.`と打てば即座にこの値を取れます。

コマンドはいつも出力を`STDOUT`で、エラーを`STDERR`で返し、そしてエラーを報告するリターンコードをよりスクリプトに親切な方法で返します。
リターンコードまたは終了ステータスは、スクリプト/コマンドが有する実行状況をやり取りする手段です。
値が0は通常は何も問題がなかったことを意味し、0以外の値はエラーが起こったことを意味します。

終了ステータスはコマンドを条件付きで実行するためにも使われます。その際は`&&`（and演算子）と`||`（or演算子）の２つの[短絡評価](https://en.wikipedia.org/wiki/Short-circuit_evaluation)演算子を使います。また、コマンドは同じ行内で`;`を用いて分割できます。
`true`であるプログラムは常に0のリターンコードを返し、`false`であるコマンドは常に1のリターンコードを返します。
いくつかの例を見てみましょう

```bash
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

もう一つのよくあるパターンはコマンドの出力を変数として受け取りたい場合です。これは_コマンド代入_でできます。
`$( CMD )`と書けば、`CMD`が実行され、そのコマンドの出力が渡されて元の場所に代入されます。
例えば、`for file in $(ls)`を実行すると、シェルはまず`ls`を呼び出し、次にその値たちについて繰り返し処理する。
これと似たものであまり知られていない書き方として、`<( CMD )`は`CMD`を実行してその出力を一時的なファイルに保存し、`<()`にそのファイルの名前を代入します。これは値がSTDINではなくファイルで受け渡されるコマンドを使うときには有用です。例えば、`diff <(ls foo) <(ls bar)`はディレクトリ`foo`と`bar`の中のファイルの違いを示してくれます。


とまあ多くの内容を一気に説明してきたので、これらの書き方のいくつかを用いた例を見てみましょう。ここでは与えられた引数について繰り返し処理していき、文字列`foobar`を`grep`で抽出し、もし見つからなかったらコメントとして`foobar`を処理中のファイルに追加します。

```bash
#!/bin/bash

echo "Starting program at $(date)" # 日付が代入される

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
	# パターンに一致するものがなかった場合、grepの終了ステータスは1
	# ここでSTDOUTとSTDERRはどうでもいいので、nullレジスタにリダイレクト
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

比較では`$?`が0と等しくないかどうかを調べました。
bashではこのような比較演算が多数用意されています。詳細はマニュアルページ[`test`](https://www.man7.org/linux/man-pages/man1/test.1.html)を参照してください。
bashにおいて比較を用いる際は、一重の角括弧`[ ]`ではなく二重の角括弧`[[ ]]`を使うようにしましょう。`sh`には移植できませんが、このほうが書き間違いは少ないです。より詳しい説明は[ここ](http://mywiki.wooledge.org/BashFAQ/031)を見てください。

スクリプトを走らせる際には、似たような引数を指定することが多々あります。bashにはこれを簡便に行う方法があり、ファイル名展開を行い式を展開することができます。このテクニックはよくshell globbingと呼ばれます。
- ワイルドカード - 何らかのワイルドカード検索をかけたい場合、`?`と`*`はそれぞれ1文字oおよび任意の文字数にマッチします。例えば、ファイル`foo`、`foo1`、`foo2`、`foo10`および`bar`があったとすると、コマンド`rm foo?`は`foo1`、`foo2`を削除しますが、一方で`rm foo*`は`bar`以外の全てのファイルを削除します。
- 波括弧 `{}` - 複数のコマンドにおいて共通の部分文字列があった場合、波括弧を使ってbashに自動的に展開させることができます。これはファイルの移動や変換において非常に便利です。

```bash
convert image.{png,jpg}
# は下のように展開される
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# は下のように展開される
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# glob手法も併用できる
mv *{.py,.sh} folder
# は全ての*.pyと*.shファイルを移動させる


mkdir foo bar
# これはfile/a、file/b、... file/a、bar/a、bar/b、... bar/hを生成する
touch {foo,bar}/{a..h}
touch foo/x bar/y
# fooとbarの違いを見てみる
diff <(ls foo) <(ls bar)
# 出力
# < x
# ---
# > y
```

<!-- 最後に、パイプ`|`はスクリプトにおける重要な構文です。パイプは一つのプログラムの出力と次のプログラムの入力を繋げてくれます。これについてはデータラングリングの講義で詳細に扱います。 -->

`bash`スクリプトを書くのは難解で非直感的になるかもしれません。sh/bashスクリプトのエラーを検出する[shellcheck](https://github.com/koalaman/shellcheck)のようなツールを利用するのも良いでしょう。

ここで、ターミナルで実行するスクリプトは必ずしもbashである必要はありません。例えば、以下は引数を逆順に出力する簡単なPythonのスクリプトです。

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

カーネルがこのスクリプトはshellコマンドではなくpythonインタプリタで実行するべきだと分かるのは、スクリプトの先頭に[シェバン](https://en.wikipedia.org/wiki/Shebang_(Unix))が書いてあるからです。
シェバン行を[`env`](https://www.man7.org/linux/man-pages/man1/env.1.html)コマンドで書くのは良い練習になります。これはシステムの中でコマンドがどこにあるのかを探し当ててくれるので、スクリプトの移植性を高めます。位置を検索するのに、`env`は第一回の講義で紹介した`PATH`環境変数を利用しています。
今回の例ではシェバン行は`#!/usr/bin/env python`のようになるでしょう。

シェル関数とスクリプトの違いで覚えるべきなのは:
- 関数はシェルと同じ言語でなければならない。一方スクリプトはどの言語でも良い。これがスクリプトにおいてシェバンが重要である理由です。
- 関数は一度定義が読み込まれればロードされる。スクリプトは実行するたびに毎回ロードされる。そのため関数はロードが僅かに速いが、変更されるたびに定義を再読込しなければいけない。
- 関数は現在のシェル環境で実行されるのに対し、スクリプトは自分のプロセスの中で実行される。故に、関数は環境変数を変えることができる。つまり関数はカレントディレクトリを変えられるが、スクリプトはそれができない。スクリプトには[`export`](https://www.man7.org/linux/man-pages/man1/export.1p.html)でエクスポートされた環境変数の値が渡される。
- どのプログラミング言語とも同様に、関数はモジュール性・再利用性・可読性を高めるのに強力な構造です。シェルスクリプトに固有の関数定義があることは珍しくありません。

# シェルツール

## コマンドの使い方

この時点で、エイリアスの章で紹介したコマンド、例えば`ls -l`、`mv -i`や`mkdir -p`を使うときに、どうやってフラグを見つけるのかが気になるかもしれません。
より一般的に、あるコマンドが与えられたとすると、それが何をするコマンドでどんなオプションがあるのかをどう調べればいいのでしょうか？
ググっても勿論いいのですが、UNIXはStackOverflowよりも昔に作られたので、こういった情報を得るための備え付けの方法があります。

シェルの講義で見てきたように、第一のアプローチとしてはコマンドを`-h`や`--help`フラグをつけて呼び出す方法です。より詳細に調べる方法は`man`コマンドを使う方法です。
manualの略で、[`man`](https://www.man7.org/linux/man-pages/man1/man.1.html)は指定したコマンドのマニュアルページ（manpageという）を出してくれます。
例えば、`man rm`は`rm`コマンドの動作と取りうるフラグを一緒に出力します。先に見せた`-i`フラグも含まれます。
実は、今まで各コマンドについて参照してきたリンクはそれらのコマンドのLinuxマニュアルページのオンラインバージョンです。
内部コマンドではなくインストールしたコマンドでさえ、開発者が書いていてインストールプロセスの一部として取り込まれていれば、マニュアルページの登録があります。
対話式ツール、例えばncursesを基に作られたコマンドなどのヘルプには、プログラム内で`:help`コマンドや`?`コマンドを用いてアクセスできることが多いです。

時々、マニュアルページがコマンドについて詳しく説明すぎて、普通に使うのにどのフラグ/構文を使えばいいのか却って解読しづらいことがあります。
[TLDRページ](https://tldr.sh/)はこれに対する素晴らしい補完策です。コマンドの使用例を重点的に教えてくれるので、どのオプションを使えば良いのかが即座にわかります。
例えば、私なんかは[`tar`](https://tldr.ostera.io/tar)とか[`ffmpeg`](https://tldr.ostera.io/ffmpeg)を調べるときは、マニュアルページよりもtldrページの方を遥かに多く引いたりしますね。


## ファイル検索

プログラマであれば誰しもが直面する、一番よくやる繰り返し作業の一つが、ファイルとディレクトリの検索です。
UNIX系のシステムは全て[`find`](https://www.man7.org/linux/man-pages/man1/find.1.html)というファイルを検索するための素晴らしいシェルツールが付属しています。`find`はいくつかの条件にマッチするファイルを再帰的に検索してくれます。ちょっとした例は:

```bash
# srcという名前の全てのディレクトリを検索
find . -name src -type d
# パスにtestという名前のフォルダがある全てのpythonファイルを検索
find . -path '*/test/*.py' -type f
# 一日前に変更された全てのファイルを検索
find . -mtime -1
# サイズが500kから10Mの間にある全てのzipファイルを検索
find . -size +500k -size -10M -name '*.tar.gz'
```
ファイルを列挙する以外にも、findは一致したファイルに対して処理ができます。
この性質は、結構単調になりがちな作業の簡略化に驚くほど役に立ちます。
```bash
# .tmp拡張子のファイルを全て削除する
find . -name '*.tmp' -exec rm {} \;
# 全てのPNGファイルを検索し、JPGに変換する
find . -name '*.png' -exec convert {} {}.jpg \;
```

ありきたりな単語であるにもかかわらず、`find`の文法は覚えるのに苦労することが時々あります。
例えば、ただ何らかのパターン`PATTERN`に一致するファイルを探すのにも、`find -name '*PATTERN*'`（大文字と小文字を区別しないためには`-iname`）を実行しなければなりません。
こういった場合に備えてエイリアスを作るのも良いですが、シェルに関する知見の一つとしては、代替手段を探るのも悪くないということです。
シェルの最良の性質の一つは、プログラムを呼び出している点にあることを覚えましょう。なので、一部のプログラムに対しては、代わりのものを探す（もしくは自分で書いてしまう）ことができます。
例えば、`find`の代わりとして、シンプルで動作が速く使いやすいのが[`fd`](https://github.com/sharkdp/fd)です。
このコマンドにはカラー出力、デフォルトの正規表現検索、そしてUnicode対応といった素晴らしいデフォルト設定があります。また私個人の意見として、こちらの方がより直感的な文法になっています。
例として、パターン`PATTERN`を検索する文法は`fd PATTERN`になっています。

多くの人は`find`よりも`fd`の方が良いという意見に賛成するでしょう。しかし、ファイルを毎回探しに行くのと、何らかの索引またはデータベースを構築して素早く検索するのと、どっちの効率が良いか悩む人もいるでしょう。
そのために用意されたのが[`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html)です。
`locate`は[`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html)によって更新されるデータベースを使っています。
ほとんどのシステムでは、`updatedb`は[`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html)を通して毎日アップデートされています。
そのためこの二者におけるトレードオフは実行の速さと情報の新しさです。
また、`find`とそれに似たようなツールは、ファイルサイズ、変更日時、あるいはファイルのパーミッションなどの属性を使って検索できます。それに対し、`locate`はファイル名のみ使用します。
より掘り下げた比較の議論は[ここ](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)を参照してください。

## コードの検索

ファイルを名前で検索するのは有用です。しかし、ファイルの*内容*を検索したい場合も結構よくあります。
よくあるのが何らかのパターンを含む全てのファイルを検索し、それが出現した箇所も一緒に出してほしいというケースです。
この目的を実現するために、ほとんどのUNIX系のシステムは[`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html)という入力テキストから一致パターンを探す汎用ツールが備えられています。
`grep`は驚くほど有用なシェルツールです。詳しい内容はデータラングリングの講義で触れます。

今のところは、`grep`は色んななフラグがあって、使い方が多種多様なツールとだけ覚えておいてください。
私がよく使うのは、一致した行周辺の**C**ontext（文脈）を出す`-C`と、一致をin**v**ert（反転）する、つまりパターンに一致**しない**行を全て出力する`-v`です。例えば、`grep -C 5`は一致した行の前の５行と後ろの５行を出力します。
多くのファイルを一気に検索するときには、`-R`を使いましょう。これは**R**ecursively(再帰的に)ディレクトリの中まで行って一致する文字列を含むファイルを探してくれます。

ただし`grep -R`は他にもたくさんの使い方に発展できます。例としては`.git`フォルダを無視したり、マルチコア処理をしたり、等々。
`grep`の代替ツールも色々作られています。例えば[ack](https://beyondgrep.com/)、[ag](https://github.com/ggreer/the_silver_searcher)、それから[rg](https://github.com/BurntSushi/ripgrep)などがあります。
これらは皆優秀なツールで、割と同じ機能を持っていたりします。
今回はripgrep（`rg`）に注目て、これがどれくらい速くて直感的なのかをお見せします。いくつかの例としては：
```bash
# requestsライブラリを使用した全てのpythonファイルを検索
rg -t py 'import requests'
# シェバン行がない全てのファイル（隠しファイルも含む）を検索
rg -u --files-without-match "^#!"
# 全てのfooを検索し、その後の５行を出力
rg foo -A 5
# 一致した集計（一致した行とファイルの数）を出力
rg --stats PATTERN
```

ここで注意してほしいですが、大事なのはこれらのツールのどれも`find`/`fd`と同じようにこの手問題を解決することができるのを知っていることで、具体的にどのツールを使うかはさほど重要ではないんです。

## シェルコマンドの検索

ここまではファイルやコードの検索の仕方について見てきました。しかし、シェルをもっと触ろうとすると、どこかで打ったコマンドをピンポイントで検索したくなったりするでしょう。
まず最初に知っておくべきこととして、上矢印キーを押すと最後に打ったコマンドが出てきて、そのまま押し続けるとシェルの履歴をちょっとずつ遡ることができます。

そこで、`history`コマンドはプログラム的にシェル履歴にアクセスできます。
これはシェルの履歴を標準出力に出力するコマンドです。
もしその履歴内で検索したいのなら、パイプで出力を`grep`に繋げてパターン検索をかけられます。
`history | grep find`は"find"という部分文字列を含むコマンドを出力してくれます。

ほとんどのシェルでは、`Ctrl+R`を使うことで履歴をさかのぼって検索できます。
`Ctrl+R`を押したあと、履歴の中で検索したいコマンドの部分列を入力すれば良い訳です。
`Ctrl+R`を押し続けると、履歴中で一致したものを巡回します。
これは[zsh](https://github.com/zsh-users/zsh-history-substring-search)では上/下矢印キーでも有効にできます。
`Ctrl+R`の更なる発展として便利なのが[fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r)バインディングです。
`fzf`は様々なコマンドで利用できる汎用的な曖昧検索ツールです。
ここでは履歴内を曖昧検索した結果を、手ごろで見やすい形に出力するのに使われています。

履歴関連で私が気に入っているもう一つの凄い技は、**履歴による自動補完**です。
[fish](https://fishshell.com/)シェルで初めて導入されたこの機能は、現在のシェルコマンドをそれと同じ書き出しである一番直近のコマンドで動的に自動保管してくれます。
この機能は[zsh](https://github.com/zsh-users/zsh-autosuggestions)でも有効にできます。あなたの素敵なシェルライフの一助となるでしょう。

シェル履歴の動作を変更することもできます。例えばスペースで始まるコマンドを履歴に含まない、など。これはパスワードや他の扱いに注意が必要な情報を含むコマンドを入力する場合ではお手ごろです。
これを機能させるのには、`.bashrc`を`HISTCONTROL=ignorespace`に、あるいは`setopt HIST_IGNORE_SPACE`を`.zshrc`に追加すれば良いです。
もし先頭のスペースを入れ忘れても、`.bash_history`や`.zhistory`を編集して手動で記録を削除するのはいつでもできます。

## ディレクトリ移動

さて、これまでの議論では、既に操作を実行する場所にいることを想定してきた訳ですが、それではディレクトリ間の移動を素早くこなすにはどうすればよいのでしょうか？
簡単な方法はたくさんあります。例えばシェルエイリアスを書いたり、[ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html)を使ってシンボリックリンクを作ったり。でも実は開発者が既にとても賢くて精巧な解決策を用意してくれました。

この講義のテーマに従って、一般的なケースを最適化する方法を考えていきましょう。
頻繁に、かつ/または直近で使ったファイルとディレクトリの検索は、[`fasd`](https://github.com/clvv/fasd)や[`autojump`](https://github.com/wting/autojump)などのツールで行えます。
fasdはファイルとディレクトリを[_frecency_](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm)、つまり_frequency(頻度)_と_recency(新しさ)_の両方の基準を使ってランク付けします。
デフォルトでは、`fasd`は追加として`z`というコマンドを使って、_frecent_なディレクトリの部分文字列をもとに素早くそこに`cd`できます。例として、`/home/user/files/cool_project`によく移動していたら、`z cool`と打つだけでそこへ飛べます。autojumpを使うなら、`j cool`で同じようなディレクトリ移動が可能です。

More complex tools exist to quickly get an overview of a directory structure: [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) or even full fledged file managers like [`nnn`](https://github.com/jarun/nnn) or [`ranger`](https://github.com/ranger/ranger).
もう少し複雑なツールとして、ディレクトリ構造の概観を手っ取り早く示してくれるもの：[`tree`](https://linux.die.net/man/1/tree)、[`broot`](https://github.com/Canop/broot)があります。もっというと、[`nnn`](https://github.com/jarun/nnn)や[`ranger`](https://github.com/ranger/ranger)などの本格的なファイルマネージャも挙げられます。

# Exercises

1. Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner

    - Includes all files, including hidden files
    - Sizes are listed in human readable format (e.g. 454M instead of 454279954)
    - Files are ordered by recency
    - Output is colorized

    A sample output would look like this

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. Write bash functions  `marco` and `polo` that do the following.
Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`.
For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

1. Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run.
Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end.
Bonus points if you can also report how many runs it took for the script to fail.

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "found error after $count runs"
cat out.txt
{% endcomment %}

1. As we covered in the lecture `find`'s `-exec` can be very powerful for performing operations over the files we are searching for.
However, what if we want to do something with **all** the files, like creating a zip file?
As you have seen so far commands will take input from both arguments and STDIN.
When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments.
To bridge this disconnect there's the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments.
For example `ls | xargs rm` will delete the files in the current directory.

    Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`).
    {% comment %}
    find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf archive.tar.gz
    {% endcomment %}

    If you're on macOS, note that the default BSD `find` is different from the one included in [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands). You can use `-print0` on `find` and the `-0` flag on `xargs`. As a macOS user, you should be aware that command-line utilities shipped with macOS may differ from the GNU counterparts; you can install the GNU versions if you like by [using brew](https://formulae.brew.sh/formula/coreutils).

1. (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?
