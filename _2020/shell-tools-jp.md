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
- 関数は一度定義が読み込まれればロードされる。スクリプトは実行するたびに毎回ロードされる。そのため関数はロードが僅かに早いが、変更されるたびに定義を再読込しなければいけない。
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
This property can be incredibly helpful to simplify what could be fairly monotonous tasks.
この性質は、とにかく単調な作業の簡略化に驚くほど役に立ちます。
```bash
# .tmp拡張子のファイルを全て削除する
find . -name '*.tmp' -exec rm {} \;
# 全てのPNGファイルを検索し、JPGに変換する
find . -name '*.png' -exec convert {} {}.jpg \;
```

Despite `find`'s ubiquitousness, its syntax can sometimes be tricky to remember.
For instance, to simply find files that match some pattern `PATTERN` you have to execute `find -name '*PATTERN*'` (or `-iname` if you want the pattern matching to be case insensitive).
You could start building aliases for those scenarios, but part of the shell philosophy is that it is good to explore alternatives.
Remember, one of the best properties of the shell is that you are just calling programs, so you can find (or even write yourself) replacements for some.
For instance, [`fd`](https://github.com/sharkdp/fd) is a simple, fast, and user-friendly alternative to `find`.
It offers some nice defaults like colorized output, default regex matching, and Unicode support. It also has, in my opinion, a more intuitive syntax.
For example, the syntax to find a pattern `PATTERN` is `fd PATTERN`.

Most would agree that `find` and `fd` are good, but some of you might be wondering about the efficiency of looking for files every time versus compiling some sort of index or database for quickly searching.
That is what [`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html) is for.
`locate` uses a database that is updated using [`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html).
In most systems, `updatedb` is updated daily via [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html).
Therefore one trade-off between the two is speed vs freshness.
Moreover `find` and similar tools can also find files using attributes such as file size, modification time, or file permissions, while `locate` just uses the file name.
A more in-depth comparison can be found [here](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

## Finding code

Finding files by name is useful, but quite often you want to search based on file *content*. 
A common scenario is wanting to search for all files that contain some pattern, along with where in those files said pattern occurs.
To achieve this, most UNIX-like systems provide [`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html), a generic tool for matching patterns from the input text.
`grep` is an incredibly valuable shell tool that we will cover in greater detail during the data wrangling lecture.

For now, know that `grep` has many flags that make it a very versatile tool.
Some I frequently use are `-C` for getting **C**ontext around the matching line and `-v` for in**v**erting the match, i.e. print all lines that do **not** match the pattern. For example, `grep -C 5` will print 5 lines before and after the match.
When it comes to quickly searching through many files, you want to use `-R` since it will **R**ecursively go into directories and look for files for the matching string.

But `grep -R` can be improved in many ways, such as ignoring `.git` folders, using multi CPU support, &c.
Many `grep` alternatives have been developed, including [ack](https://beyondgrep.com/), [ag](https://github.com/ggreer/the_silver_searcher) and [rg](https://github.com/BurntSushi/ripgrep).
All of them are fantastic and pretty much provide the same functionality.
For now I am sticking with ripgrep (`rg`), given how fast and intuitive it is. Some examples:
```bash
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

Note that as with `find`/`fd`, it is important that you know that these problems can be quickly solved using one of these tools, while the specific tools you use are not as important.

## Finding shell commands

So far we have seen how to find files and code, but as you start spending more time in the shell, you may want to find specific commands you typed at some point.
The first thing to know is that typing the up arrow will give you back your last command, and if you keep pressing it you will slowly go through your shell history.

The `history` command will let you access your shell history programmatically.
It will print your shell history to the standard output.
If we want to search there we can pipe that output to `grep` and search for patterns.
`history | grep find` will print commands that contain the substring "find".

In most shells, you can make use of `Ctrl+R` to perform backwards search through your history.
After pressing `Ctrl+R`, you can type a substring you want to match for commands in your history.
As you keep pressing it, you will cycle through the matches in your history.
This can also be enabled with the UP/DOWN arrows in [zsh](https://github.com/zsh-users/zsh-history-substring-search).
A nice addition on top of `Ctrl+R` comes with using [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) bindings.
`fzf` is a general-purpose fuzzy finder that can be used with many commands.
Here it is used to fuzzily match through your history and present results in a convenient and visually pleasing manner.

Another cool history-related trick I really enjoy is **history-based autosuggestions**.
First introduced by the [fish](https://fishshell.com/) shell, this feature dynamically autocompletes your current shell command with the most recent command that you typed that shares a common prefix with it.
It can be enabled in [zsh](https://github.com/zsh-users/zsh-autosuggestions) and it is a great quality of life trick for your shell.

You can modify your shell's history behavior, like preventing commands with a leading space from being included. This comes in handy when you are typing commands with passwords or other bits of sensitive information.
To do this, add `HISTCONTROL=ignorespace` to your `.bashrc` or `setopt HIST_IGNORE_SPACE` to your `.zshrc`.
If you make the mistake of not adding the leading space, you can always manually remove the entry by editing your `.bash_history` or `.zhistory`.

## Directory Navigation

So far, we have assumed that you are already where you need to be to perform these actions. But how do you go about quickly navigating directories?
There are many simple ways that you could do this, such as writing shell aliases or creating symlinks with [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html), but the truth is that developers have figured out quite clever and sophisticated solutions by now.

As with the theme of this course, you often want to optimize for the common case.
Finding frequent and/or recent files and directories can be done through tools like [`fasd`](https://github.com/clvv/fasd) and [`autojump`](https://github.com/wting/autojump).
Fasd ranks files and directories by [_frecency_](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm), that is, by both _frequency_ and _recency_.
By default, `fasd` adds a `z` command that you can use to quickly `cd` using a substring of a _frecent_ directory. For example, if you often go to `/home/user/files/cool_project` you can simply use `z cool` to jump there. Using autojump, this same change of directory could be accomplished using `j cool`.

More complex tools exist to quickly get an overview of a directory structure: [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) or even full fledged file managers like [`nnn`](https://github.com/jarun/nnn) or [`ranger`](https://github.com/ranger/ranger).

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
