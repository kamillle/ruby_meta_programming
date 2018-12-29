ntegerやString)に対してメソッドを追加したい時は、そのクラスをオープンしてメソッドを追加することができる

```
class String
  def add_hoge_as_suffix
    "#{self}_hoge"
  end
end

"huga".add_hoge_as_suffix
# => "huga_hoge"
```

## 2.1.1 クラス定義の内部
クラスの中にあるコードがその他のコードと同時に実行された時、同じ名前のクラスが定義されることはない

```
class D
  def x; 'x'; end
end

class D
  def y; 'y'; end
end

o = D.new
o.x # => 'x'
o.y # => 'y'
```

最初の `class D` の段階ではまだDクラスは存在していない。rubyがこのクラスと xメソッドが定義されているコードに入った時に初めてrubyがクラスを定義する。
2回目の `class D` の時にはすでにDクラスは存在するので、既存のDクラスをオープンして、 yメソッドを追加する。

いつでも既存のクラスをオープンして、その場で追加や修正が行える。
この技法を **オープンクラス** と呼ぶ

## 2.1.2 Monetizeの例
Monetizeという金額や通過の扱いの長けたgemでもオープンクラスが使われている。
下記は Numericクラスをオープンしてメソッドを追加してる例だ。

```
m = 100.to_money('USD')
m.format # => '$100.00'

class Numeric
  def to_money(currnecy = nil)
    Monetize.from_numeric(self, currency || Money.dafault_currency)
  end
end
```


## 2.1.3 オープンクラスの問題点
下記のようにオープンにするクラスに元々あるメソッドをオープンクラスを用いて行うと、元々のメソッド(下記の例では replace)に依存していた別のメソッドの挙動が変わってしまうため、注意が必要

```
class Array
  def replace(original, replacement)
    self.map { |e| e == original ? replacement : e }
  end
end
```

このようなクラスへの安易なパッチのことを **モンキーパッチ** と呼ぶ

## 2.2 オブジェクトモデルの内部
オブジェクトモデルについて知るべきことは多いので、まずはクラスやオブジェクトについてから理解する

## 2.2.1 オブジェクトの中身
以下のコードを例として解説を進める

```
class MyClass
  def my_method
    @v = 1
  end
end
```

### インスタンス変数
オブジェクトに含まれるインスタンス変数は `Object#instance_variables` メソッドを呼び出せばわかる。
↑の MyClassインスタンスにはインスタンス変数は含まれないが、 `my_method` 実行後にインスタンス変数が１つ追加される。

```
m = MyClass.new
m.instance_variables # => 
m.my_method # => 1
m.instance_variables # => [:@v]
```

rubyでは同じクラスのオブジェクトでもインスタンス変数の数が異なることがある。上記の例のようにメソッドが実行されて初めて値が代入されることがあるからだ。

### メソッド
オブジェクトのメソッドは `Object#methods` を呼び出せば一覧を取得できる
インスタンスメソッドなら `Object#instance_methods` 。引数にfalseを渡すと継承しているメソッドの除いたインスタンスメソッドを返してくれる

オブジェクトのインスタンス変数はオブジェクトに住んでいる。オブジェクトのメソッドはオブジェクトのクラスに住んでいる。

## 2.2.2 クラスの真相
オブジェクトモデルを学ぶ時に最も重要なのは、「クラスもオブジェクトである」ということオブジェクトに当てはまるものはクラスにも当てはまるということになる。
オブジェクトと同じでクラスにもクラスがあり、コードにすると下記のようになる

```
'hello'.class # => String
String.class  # => Class
```

rubyにおける Classクラスのインスタンスはクラスそのものといえて、他のオブジェクトを操作することもできる。

オブジェクトのメソッドはそのクラスのインスタンスメソッドとなる。ではクラスのメソッドはどうなるのか？
答えは Classクラスのインスタンスメソッドである。

```
Class.instance_methods(false) # => [:allocate, :new, :superclass]
```

※ `allocate` は引数などを無視してクラスのインスタンスを作成できる
`superclass` は継承してる親のクラスを参照できる
ArrayクラスはObjectクラスを継承しているので、Arrayはオブジェクトであるということができる。また

```
Array.superclass # => Object
Object.superclass # => BasicObject
BasicObject # => nil
```

### モジュール
Classクラスのスーパークラスが Moduleクラスだ。
つまりすべてのクラスはModuleクラスと言える。正確に言うとクラス(Class)はオブジェクトの生成やクラスを継承するための3つのインスタンスメソッド(new, allocate, superclass)を追加したモジュールらしい。

使い分けとして、インクルードをするときはModule, インスタンスの生成や継承を行うときはClassを使うのが一般的らしい。

## ここまでのまとめ
MyClassとmy_classは、どちらもClassクラスのインスタンスの参照である。両者の違いはmy_classは変数であり、MyClassは定数である。
つまり、クラスはオブジェクトで、クラス名は定数であると言える。

## 2.2.3 定数
rubyではクラス名やモジュール名も定数に分類される。
スコープが違う場合は、同じ名前の定数を何個でも作成可能

```
module Kamijima
  Constant = '定数'

  class Yuji #=> Kamijima::Yuji
    X = 'ある定数'
  end
end

Kamijima::Constant # => 定数の参照
Kamijima::Yuji     # => クラスの参照
Kamijima::Yuji::X  # => 定数の参照
Yuji::X            # => 定数の参照
```

定数ツリーの奥の方にいる時は、ルートを示すコロン2つで書き始めれば、外部の定数を絶対パスで指定できる

```
Y = 'ルートレベルの定数'

module M
  Y = 'Mにある定数'
  Y # => 'Mにある定数'
  ::Y # => 'ルートレベルにある定数'
end
```

Moduleクラスは定数を呼び出すインスタンスメソッドとクラスメソッドを持っている。(どちらも同じ名前)
インスタンスメソッドの `Module#constants` は現在スコープにある定数をすべて返す。ファイルシステムのlsのようなもの
クラスメソッドの `Module.constants` は現在のプログラムのトップレベルの定数を戻す。(クラス名の含む)

```
module M
  Y = 'Mにある定数'
  
  class Z
  end
end


M.constants # => [:Y, :Z] インスタンスメソッド
Module.constants # クラスメソッド
```

## 2.2.4 オブジェクトとクラスのまとめ
オブジェクトはインスタンス変数の集まりにクラスへのリンク(object.class # => MyClassのこと)がついたもの。
オブジェクトのメソッドはオブジェクトではなくオブジェクトのクラスに住んでいて、クラスのインスタンスメソッドと呼ばれている。
クラスはオブジェクト(classクラスのインスタンス)にインスタンスメソッドの一覧とスーパークラスへのリンクがついたものである。
classクラスはmoduleクラスのサブクラスである。つまりクラスもモジュールである。

## 2.2.5 ネームスペースを使う
汎用的な名前のクラス(たとえばStringとかText)はすでに定義されている可能性が高い。すでに定義されているクラス名を猜疑しようとすると `ClassName is not a class` を返す。
その場合はmoduleを使ってネームスペースを切るといい。

### loadとrequire
loadには副作用があり、読み込んだファイルの定数やクラスが読み込み先のプログラムに入り込み汚染してしまう。そこで第二引数にオプションを渡すと読み込むファイル内で定義されている定数やクラスは一度無名モジュール内に取り込まれ、コードが実行されたらそのモジュールは破棄される。

```
load('hoge.rb')
# hoge.rbのクラスや定数はプログラムを汚染しない
load('hoge.rb', true)
```

loadは読み込んだファイルを実行するために使い、requireはライブラリをインポートするのに使う。ライブラリをインポートするには定数が必要なのでrequireには第二引数のオプションが存在しないが、ファイルを実行するloadでは定数が必要なのは実行時だけなので実行後に破棄ができる

## 2.4 メソッドを呼び出す時に何が起きているの？
メソッドを呼び出すとrubyは2つのことを行う。
1. メソッドを探す。これをメソッド探索と呼ぶ。
2. メソッドを実行する。これにはselfが必要になる。

メソッド探索はすべてのオブジェクト指向言語で起きている。

## 2.4.1 メソッド探索
メソッド探索を理解する前にレシーバーと継承チェーンについて理解する必要がある。
レシーバーは、呼び出すメソッドが属するオブジェクトのこと。例えば `my_string.reverse` と書けば、my_stringがレシーバー。
継承チェーンは、クラスからスーパークラスへ、スーパークラスからそのスーパークラスへの移動。クラス階層のルートのBasicObjectまで移動すること。(クラスの継承チェーンは `ancestors` メソッドで確認できる)
メソッド探索は、rubyがレシーバーのクラスに入ってメソッドを見つけるまで継承チェーンを登ること。

下記を例に説明する。
my_methodが呼ばれると、SubClassのインスタンスであるobjがレシーバーとなる。rubyはレシーバーのクラス内に入ってこのmy_methodがないか探す。
なかった場合、SubClassのスーパークラスのMyClassに移動してメソッドを探す。(メソッドを見つけるまでこれを繰り返す。)

```
class MyClass
  def my_method; 'hoge' end
end

class SubClass < MyClass
end

obj = SubClass.new
obj.my_method # => 'hoge'
```

### モジュールと探索
継承チェーン内にはスーパークラスとモジュールが含まれる。インクルードされたモジュールは、継承チェーン上ではインクルード先のクラスの上にはいる。

```
class M
end

# KernelはObject内でインクルードされているモジュール
M.ancestors # => [:M, Object, Kernel, BasicObject]
```

モジュールをインクルードするには `include` or `prepend` メソッドを使う。
違いとしては
includeではインクルードされたモジュールは継承チェーン上では継承チェーン上ではインクルード先のクラスの上にはいる。
prependではインクルードされたモジュールは継承チェーン上では継承チェーン上ではインクルード先のクラスのしたにはいる。

### 多重インクルード
下記の例だと、M3にM2がインクルードされた時には、M2内でインクルードされているM1はすでにインクルード(prepend)されているため、このincludeは意味をなさない。
rubyは2回目以降の挿入を無視するらしい。

```
module M1; end

module M2
  include M1
end

module M3
  prepend M1
  include M2
end

M3.ancestors # => [:M1, :M3, :M2]
```

### Kernelとは
ObjectでインクルードされているKernelとはなんだろう。
rubyではどこからでも呼び出せるメソッド、たとえばprintなどがそうだ。printはすべてのオブジェクト内で定義されているわけではなく、Kernelモジュールのプライベートメソッドとして定義されている。
printメソッドが呼ばれるとrubyは継承チェーン内を探索して、Kernel内で見つけてメソッドを実行している。

※ Kernalモジュールをオープンして自分でメソッドを追加するとどこでも使えるKernelメソッドができあがる

## 2.4.2 メソッドの実行
レシーバーをselfとしてメソッドを実行する。実行する上で参照が必要になる変数もselfに属するものとみなしてrubyは実行する。

## 2.4.3 Refinements
ruby2.0からは Refinements という魔術的なモンキーパッチができるようになった。
下記のコードではStringExtensionsというモジュールの中で refine String という記述を用いて Stringクラスにメソッドを追加してる。

```
module StringExtensions
  refine String do
    def to_alphanumeric
      gsub(/[^\w\s]/, '')
    end
  end
end
```

refineを用いたモンキーパッチでは通常のオープンクラスと異なりそのままでは有効にならない。

```
# メソッドがないと言われる
"my *1st* refinement!".to_alphanumeric
=> NoMethodError: undefine method `to_alphanumeric`
```

変更を反映するには usingを使って明示的に追加したメソッドを有効にしてやらないといけない

```
using StringExtensions

"my *1st* refinement!".to_alphanumeric # =>  'my 1st refinement'
```

ruby2.1からはモジュール定義の中でもusingが呼べるようになった。
refinementはモジュール定義の終わりまで有効になる。以下のコードでは `String#reverse` にパッチを当てているが、それが有効なのはStringStuffモジュールの定義のないだけである

```
module StringExtensions
  refine String do
    def reverse
      'eserver'
    end
  end
end

module StringStuff
  using StringExtensions
  'my_string'.reverse # => 'eserver'
end

# StringStuffの外なので #reverse の挙動はいつもどおり
'my_string'.reverse # => 'gnirts_ym'
```

Refinementはグローバルで有効にならない。有効になるのは、refineブロックとusingを呼び出した場所からモジュールの終わりまで、またはファイルの終わりまで。
Refinementが有効になっているコードは、リファインされた側のコード(上の例でいうとStringクラスのコード)よりも優先される。またインクルードやプリペン度したモジュールのコードよりも優先される。

## 2.6 まとめ
- オブジェクトは複数のインスタンス変数とクラスへのリンクで構成されている
- オブジェクトのメソッドはオブジェクトのクラスに住んでいて、これはクラスから見るとインスタンスメソッドと呼ばれる
- クラスはClassクラスのインスタンスである。クラス名は定数という扱い
- ClassはModuleのサブクラス(Class < Module)である。モジュールは基本的にはメソッドをまとめたものである。それに加えてクラスは、newでインスタンス化したり、superclassで階層構造を作ったりできる
- 定数はファイルシステムのようなツリー状に配置されている。モジュールやクラスの名前がディレクトリ、通常の定数がファイルのような扱い。
- クラスはそれぞれ BasicObject まで続く継承チェーンを持っている。
- メソッドを呼び出すとrubyはレシーバーのクラスに向かって一歩右に進み、それから継承チェーンを上に上がっていく。メソッドを発見するか継承チェーンが終わるまで続く。
- クラスにモジュールをインクルードすると、そのクラスの継承チェーンの上にモジュールが挿入される。モジュールをプリペンドすると下に挿入される。
- メソッドを呼び出す時はselfがレシーバーになる。
- モジュールを定義するときにはそのモジュールがselfになる
- インスタンス変数は常にselfのインスタンス変数とみなされる
- レシーバーを明示的に指定せずにメソッドを呼び出すとselfのメソッドだとみなされる
- Refinementはクラスのコードにパッチを当てて、通常のメソッド探索をオーバライドするようにイメージ。ただしusingを呼び出したところからファイルやモジュールのｐ定義が終わるところまでの限られた部分でのみ有効になる。