# readコマンドで対話的に文字入力を受け取る

`read` コマンドで対話的に文字入力を受け取り環境変数に値を代入する方法についてのメモ。

以下のようにコマンドを実行すると入力待ちになり、Enterキーを押すとコマンドが終了し、環境変数 `INPUT` に値が代入される。

```console
$ read INPUT
```

`-p` オプションでプロンプトを指定でき、 `-s` オプションで入力された文字列をターミナルに表示しないようにできる。

```console
$ read -s -p 'Password: ' PASSWORD
Password:
```

プロンプトが `Password: ` の直後に表示されるので、これが気になる場合は改行を挟むために `echo` を呼ぶと良い。

```console
$ read -s -p 'Password: ' PASSWORD && echo
Password:
```

ヘルプを表示するときは以下のコマンドを実行する。

```console
$ help read
```
