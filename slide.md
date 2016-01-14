# 業務で使えるPerl6 

---

## rakudoのインストール

* オススメはrakudobrew
* rbenvやplenvに相当する
* rakudoの更新ペースは非常に早いのでrakudobrewで管理しておく
* エコシステムの管理もできる

```
$ git clone https://github.com/tadzik/rakudobrew ~/.rakudobrew
$ export PATH=~/.rakudobrew/bin:$PATH
$ rakudobrew init
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

