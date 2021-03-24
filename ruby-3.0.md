# Ruby 3.0.0 コネタ集

author
:   Kazuhiro NISHIYAMA

content-source
:   Fukuoka.rb 200回 LT大会 (#202)

date
:   2021-03-24

institution
:   株式会社Ruby開発

allotted-time
:   5m

theme
:   lightning-simple

# 自己紹介

- 西山 和広
- Ruby のコミッター
- twitter, github など: @znz
- 株式会社Ruby開発 www.ruby-dev.jp

# はじめに

LT なので内容はコネタ集です

# Ractor 関連

# Ractor で SEGV

- 3.0.0 では SEGV
- コア実装の experimental な機能は SEGV バグがみつけやすいかも?

```
% ruby -e Ractor.current.dup
-e:1:in `dup': allocator undefined for Ractor (TypeError)
	from -e:1:in `<main>'
```

# Ractor をまたぐ Thread

- Ractor 終了時なら Thread がそのまま別 Ractor に移動可能
- 他の制限にひっかかって問題が起きる可能性は未発見

```
% ruby -W0 -e 'r=Ractor.new{p Thread.new{loop{}}}; p r.take'
#<Thread:0x00007fcf2586bfb8 -e:1 run>
#<Thread:0x00007fcf2586bfb8 -e:1 run>
```

# Ractor as global Queue

main Ractor を Queue 代わりに使えるかも?

```
% ruby -e 'Ractor.current.send("foo"); p Ractor.receive'
"foo"
```

# shareable の影響あり

ただし shareable ではないオブジェクトはコピーされてしまうので Queue 代わりには使いにくい

```
% ruby -e 'Ractor.current.send("foo".tap{|x|p x.object_id});
           p Ractor.receive.tap{|x|p x.object_id}'
60
80
"foo"
```

# 互換性関連のコネタ

# frozen_string_literal

`frozen_string_literal: true` magic comment 対応を 3.0.0 以降のみで確認すると対応漏れする可能性あり

```
% ruby --enable=frozen_string_literal -e 'p "#{}".frozen?'
false
```

string interpolation (文字列補間) があると frozen にならなくなった


# Warning[:deprecated]

`ruby -w` や `ruby -v` で `$VERBOSE = true` にすると `Warning[:deprecated]` も `true` になるが、プログラム中で `$VERBOSE = true` にしても `Warning[:deprecated]` は `false` のまま

```
% ruby -e 'p Warning[:deprecated]'
false
% ruby -w -e 'p Warning[:deprecated]'
true
% ruby -e '$VERBOSE=true; p Warning[:deprecated]'
false
```

# ruby -T

`$SAFE` 関連が消えて `-T` オプションが消えた

```
ruby 3.0:
% ruby -T0 -e 0
ruby: invalid option -T  (-h will show valid options) (RuntimeError)

ruby 2.7:
% ruby -T0 -e 0
ruby: warning: ruby -T will be removed in Ruby 3.0
```

将来何か他の意味に使われるかも?

# $SAFE / $KCODE

普通のグローバル変数になった (これも普通は使わない)

```
% ruby -e '$KCODE = "foo"; p $KCODE'
"foo"
```

# TRUE / FALSE / NIL

ついに消えたので普通の定数として利用可能 (普通は使わない)

```
% ruby -e 'NIL = :dummy; p NIL.nil?'
false
```

# おわり

- Ractor はまだバグがありそうなので探すと面白いかも
- 気付きにくい非互換もあるので複数 ruby バージョン対応するときには注意
- 長い間残っていて 3.0 で消えているものがあります
