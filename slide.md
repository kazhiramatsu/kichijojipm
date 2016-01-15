# 業務で使えるPerl6 

---

## 自己紹介
@kaz_hiramatsu

* Perlの同人誌書いています

<img src=http://localhost:5000/miyabi_hyoshi_c88.jpg/ width=300 height=400>
---
## Perl6ついにリリース！！
* Larry Wallによるポエム的なリリース宣言
* https://perl6advent.wordpress.com/2015/12/24/an-unexpectedly-long-expected-party/  
* 1.0とかではなくクリスマスバージョン(v6.c)

---
## Perl6は業務で使えるのか

* 速度面でもかなり改善されたので、十分使える
* 標準モジュールが充実しているため、日常のタスクをこなすには十分
* 全てを使う必要はなく、理解できる範囲で適切な機能を使う
* おすすめの使い方は社内ツールなどのスクリプトをPerl6で書くこと
* Perl5では面倒だった非同期な処理も書きやすい
* Webアプリもかけるよ

---
## rakudoのインストール

* 処理系はrakudo
* インストールはrakudobrewを使う
* rbenvやplenvに相当する
* rakudoの更新ペースは非常に早いのでrakudobrewで管理しておく
* エコシステムの管理もできる

```
$ git clone https://github.com/tadzik/rakudobrew ~/.rakudobrew
$ export PATH=~/.rakudobrew/bin:$PATH
$ rakudobrew init
$ echo 'eval "$(/home/hiramatsu/.rakudobrew/bin/rakudobrew init -)"' >> ~/.bash_profile
$ rakudobrew build moar   
```

---
## pandaのインストール

* pandaとはcpanmに相当するモジュール管理ツール
* 依存関係の管理もやってくれる
* rakudobrewでインストールできる

```
$ rakudobrew build panda
```

---

## REPL用のモジュール

* デフォルトのREPLはカーソル移動がきかない
* Linenoiseモジュールをいれる

```
$ panda install Linenoise
```

---
## Perl6 Tips 

* 業務で使えるPerl6のTipsを紹介
* 細かい言語仕様はチュートリアル参照

---
## コマンドライン引数を処理したい

* MAINサブルーチンを宣言する
* コマンドライン引数の値がMAINサブルーチンの引数の値として代入される
* ちょっとしたスクリプトを書くときに便利

---
## long/short optionはnamed argsで受け取る

```
$ perl6 foo.pl --a=1 --b=2
```

```
# foo.pl
sub MAIN(:$a, :$b) {
  say $a;       # 1
  say $b;       # 2
}
```
---
## long/short optionはハッシュのslurpy argsでも受け取れる

```
$ perl6 foo.pl --a=1 --b=2
```

```
# foo.pl
sub MAIN(*%args) {
    %args.keys.say;       # a b
    %args.values.say;     # 1 2
}
```
---
## 通常の引数は配列のslurpy argsとして受け取る 

```
$ perl6 foo.pl 1 2 3 
```

```
# foo.pl
sub MAIN(*@a) {
    say @a;       # [1, 2, 3]
}
```

---
## 特定の値のみ受け取りたい

```
$ perl6 foo.pl --a=server1 --b=server2
```

```
# foo.pl
sub MAIN(:$a where $a ~~ /server1/, :$b where $b ~~ /server2/) {
  say $a;       # server1のみ受け付ける 
  say $b;       # server2のみ受け付ける 
}
```

---
## 非同期処理を行いたい 

* Promiseを使う
* startメソッドで処理を登録
* thenで完了時の処理を行う
* awaitでPromiseの完了を待つ
* resultで結果を取得

```
my $p = Promise.start({ sleep 5; 1000});
$p.then({ say .result });
await $p;
$p.result;             
```

---

## ストリームデータを扱いたい

* Supplyオブジェクトを使う
* tapメソッドでデータを受信した時の処理を行う
* emitでデータを出力
* 1 .. \*で無限リストを生成

```
# Supplierを使ってSupply生成
my $supplier = Supplier.new;
my $supply = $supplier.Supply;
my $s1 = $supply.grep({ $_ % 2 }).map({ $_ ** 3 }).tap(-> $v { say $v });
for 1 .. * {
    $supplier.emit($_); 
}
```

---
## コマンドを非同期に実行したい 

* Proc::Async
* startメソッドでPromiseが返る
* コマンドを非同期に実行できる
* 時間のかかるコマンドを複数実行してその完了を待機できる

---
## S3にファイルをアップロードする

```
# Proc::Asyncオブジェクトを作る
my @files = <a.pdf b.pdf c.pdf d.pdf>;
my @promises;
for @files -> $file {
    my $proc = Proc::Async.new('aws', 's3', 'cp', $file, "s3://hoge/$file");
    my $promise = $proc.start;
    push @promises, $promise;
}
await @promises;
```

---
## S3にファイルを並列でアップロードする

* doで並列的に実行

```
my @files = <a.pdf b.pdf c.pdf d.pdf>;
await do for @files -> $file {
    my $proc = Proc::Async.new('aws', 's3', 'cp', $file, "s3://hoge/$file");
    $proc.stdout.tap: -> $a { note $a;};
    my $promise = $proc.start;
}

```

---
## 特定のディレクトリを監視して処理したい

* Perl6流のReactive Programming
* supplyブロックで送信側のSupplyオブジェクトを作る
* IO::Notification.watch-pathはSupplyオブジェクトを返す
* wheneverはSupplyオブジェクトから値を取得するとブロックを実行する
* emitでsupplyブロックのSupplyオブジェクトへ値を出力
* reactブロックのwheneverで値を取得
---

## Supply 

```
sub MAIN($dir) {
    my $s = supply {
        whenever IO::Notification.watch-path($dir) {
            emit .path  if .path ~~ /\.inp/;
        }
    }
    react {
        whenever $s -> $path {
            say $path;
        }
    }
}
```

---

## Webアプリケーションを開発したい

* Crustを使う
* PlackのPerl6版
* Plackを使ったことがある人ならすぐに使い方がわかる

```
$ panda install Crust
$ rakudobrew rehash 
```
---
## CrustによるWebアプリ

* sub app($env) {}というサブルーチンを定義
* Crust::Requestのpath-infoでURIを取得してgiven whenでディスパッチ
* ただし、処理の中でawaitを使うと処理が途中で終わってしまう

---
## サンプル 
```
use v6;

use Crust::Request;

my $page = q:heredoc/END/.encode('utf-8');
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Crust Test</title>
</head>
<body>
    <div id="content">
        <span class="big">Crust Test</span><br />
    <div>
</body>
</html>
END

sub app($env) {
    my $req = Crust::Request.new($env);
    given $req.path-info {
        when "/" {
            return 200, [ 'Content-Type' => 'text/html; charset=utf-8' ], [$page];
        }
        when "/upload" {
            # これが動かない
            my @files = <a.pdf a.json a.pl a.t>;
            await do for @files -> $file {
                my $proc = Proc::Async.new('aws', 's3', 'cp', $file, "s3://hoge/$file");
                $proc.stdout.tap: -> $a { note $a;};
                my $promise = $proc.start;
            }
            return 200, [ 'Content-Type' => 'text/html; charset=utf-8' ], [$page];
        }
        default {
            return 400, [], ['not found'];
        }
    }
}

```

---
## まとめ
* Perl6は業務で使える！！
* Perl6はオブジェクト指向だけでなく、コマンドラインツールも書きやすい
* 並列処理が得意なので、クラスの設計も非同期を考慮するのが重要
* 非同期で実行したい処理はPromiseを返すように設計すると使いやすくなりそう

