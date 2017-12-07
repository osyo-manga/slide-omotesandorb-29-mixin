## 表参道.rb #29 ~Module~
- - -
## mixin と module について

---

## 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)

---

## [自作 gem](https://rubygems.org/profiles/osyo-manga)
- - -

* [iolite](https://github.com/osyo-manga/gem-iolite)
  * メソッドを遅延呼び出しするための gem
* [use_arguments](https://github.com/osyo-manga/gem-use_arguments)
  * ブロック引数を省略出来る gem
* [unmixer](https://github.com/osyo-manga/gem-unmixer)
  * mixin したモジュールを削除する gem
* [proc-unbind](https://github.com/osyo-manga/gem-proc-unbind)
  * `Proc` から `UnboundMethod` を定義する

---

## アドベントカレンダーやってます！！

---

#### アドベントカレンダー
- - -

* [Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby)
  * [Ruby で型チェックを実装してみよう](http://secret-garden.hatenablog.com/entry/2017/12/01/000154)
* [一人 Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby_pink_bangbi)
  * Ruby に関する小ネタみたいなのを書いてます
* [一人 vimrc Advent Calendar 2017](https://qiita.com/advent-calendar/2017/vimrc_pink_bangbi)
  * vimrc に関する小ネタみたいなのを書いてます

---

# 今日のテーマ

---

# Module(≠module)

* Module はモジュールクラスのことを刺しますが今回話すのは module について       <!-- .element: class="fragment" -->

---

## 今日話すこと
- - -

1. 継承と mixin についておさらい       <!-- .element: class="fragment" -->
2. 継承リストと mixin の関係性       <!-- .element: class="fragment" -->
3. prepend を利用して簡単なデコレータを実装する       <!-- .element: class="fragment" -->

---

## 1. 継承と mixin についておさらい

---


# 継承とは

---

#### 継承とは
- - -

任意のクラスを拡張する（取り込む）ための機能

```ruby
class Super
    def super_method
        :super_method
    end
end

# Super クラスの機能（メソッド）を Sub クラスで継承（取り込む）する
class Sub < Super
end

# Super クラスのメソッドが利用できる
p Sub.new.super_method
# => :super_method
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2,12"></span>

---

# mixin とは

---

#### mixin とは
- - -
モジュールのメソッドをクラスに取り込むための機能

```ruby
module Mod
    def mod_method
        :mod_method
    end
end

class X
    # Mod モジュールの機能（メソッド）を X クラスで取り込む
    include Mod
end

# Mod モジュールのメソッドが利用できる
p X.new.mod_method
# => :mod_method
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="9"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2,13"></span>


---

## 継承と mixin のおさらいまとめ
- - -

* 両方共『外部の機能をクラスに取り込む』という目的では共通している  <!-- .element: class="fragment" -->
* 継承の場合は 1つのクラスしか取り込むことができない  <!-- .element: class="fragment" -->
* mixin は複数のモジュールを取り込む事が出来る  <!-- .element: class="fragment" -->

---

## 2．継承リストと mixin の関係性

---

## mixin がどうなっているのかを
## 確認するために

---

## `.ancestors` を使って
## 継承リストをみてみよう

---

## `.ancestors` とは
- - -

* .ancestors は『継承しているクラスを優先順位順の配列』として返す    <!-- .element: class="fragment" -->
* いわゆる継承リスト的なもの    <!-- .element: class="fragment" -->
  * 今回のスライドでは `.ancestors` の結果を継承リストと呼ぶことが多々あります
* メソッドを呼び出す優先順位を確認する目的として利用できる    <!-- .element: class="fragment" -->

---

## 実際使ってみよう

---

## ライブコーディング

(コードは↓↓↓)

>>>

#### `.ancestors` を使ってみる
- - -

```ruby
# 継承しているクラスの一覧を返す
p Integer.ancestors
# => [Integer, Numeric, Comparable, Object, Kernel, BasicObject]

class X
end

# デフォルトでは暗黙的に Object などが継承されている
p X.ancestors
# => [X, Object, Kernel, BasicObject]

class Super
end

class Sub < Super
end

# 継承した場合は、継承したクラスが Object の前に来る
p Sub.ancestors
# => [Sub, Super, Object, Kernel, BasicObject]
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="9"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="19"></span>

>>>

```ruby
class Super
    def mami
        "mami"
    end

    def homu
        "homu"
    end
end

class Sub < Super
    # Super クラスのメソッドを上書きする
    def homu
        "homuhomu"
    end
end

# Sub では定義されていないので次の Super のメソッドが呼び出される
p Sub.new.mami
# => "mami"

# Sub で定義されているので Sub のメソッドが呼び出される
p Sub.new.homu
# => "homuhomu"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2,19"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="13,23"></span>

---

## 以上を踏まえた上で
## mixin を試してみよう

---

## ライブコーディング

(コードは↓↓↓)

>>>

#### `#include` してモジュールを mixin する
- - -

```ruby
module Mod

end

class X
	# モジュールを mixin
	include Mod
end

# include したモジュールが X の次に追加される
p X.ancestors
# => [X, Mod, Object, Kernel, BasicObject]
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="7"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11"></span>

>>>

#### `#prepend` してモジュールを mixin する
- - -

```ruby
module Mod
	def homu
		"Mod#homu"
	end
end

class X
	prepend Mod

	def homu
		"X#homu"
	end
end

# prepend したモジュールが X よりも前に追加される
p X.ancestors
# => [Mod, X, Object, Kernel, BasicObject]

# これにより X よりも Mod のメソッドが優先して呼ばれる
p X.new.homu
# => "Mod#homu"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="16"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2,20"></span>

---

# つまり

---

## mixin も実は継承だったんだ！
## な、なんだってー！！

---

#### `#include` と `#prepend` まとめ
- - -

* 継承リスト（.ancestors）に対して      <!-- .element: class="fragment" -->
  * #include は自クラスよりも後に追加される  <!-- .element: class="fragment" --> 
  * #prepend は自クラスよりも前に追加される  <!-- .element: class="fragment" -->
    * これにより自クラスよりも優先して呼ばれるメソッドを定義できる     <!-- .element: class="fragment" -->
* つまり #include や #prepend のような mixin も実質継承しているのと同じ     <!-- .element: class="fragment" -->

---

## あれ…そういえば

---

## `#extend` も mixin では…？？？

---

## じゃあ `#extend` も試してみよう

---

## ライブコーディング

(コードは↓↓↓)

>>>

#### `#extend` おさらい
- - -

```ruby
module Mod
	def homu
		"homu"
	end
end

class X
	extend Mod
end

# X オブジェクトに対して mixin したモジュールのメソッドを呼び出すことが出来る
p X.homu
# => "homu"

# これはインスタンスオブジェクトでも同じ
x = X.new
x.extend Mod
p x.homu
# => "homu"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="12"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="17-18"></span>

>>>

#### `#extend` は特異クラスの継承リストに追加される
- - -

```ruby
module Mod
end

class X
	extend Mod
end

p X.ancestors
# => [X, Object, Kernel, BasicObject]

# 特異クラスの継承リストに追加される
p X.singleton_class.ancestors
# => [#<Class:X>, Mod, #<Class:Object>, #<Class:BasicObject>, Class, Module, Object, Kernel, BasicObject]

x = X.new.extend Mod
p x.singleton_class.ancestors
# => [#<Class:X>, Mod, #<Class:Object>, #<Class:BasicObject>, Class, Module, Object, Kernel, BasicObject]
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="5"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="12"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="15-16"></span>

>>>

#### 特異メソッドよりも優先順位が高いメソッドを定義する
- - -

```ruby
x = Object.new
def x.homu
	"homuhomu"
end
p x.homu
# => "homuhomu"

# 特異メソッドよりも優先順位が高いメソッドを定義する
# prepend することで特異クラスよりも優先順位が高くなる
x.singleton_class.prepend Module.new {
	def homu
		"homuhomuhomu"
	end
}
p x.singleton_class.ancestors
# => [#<Module:0x0000000001f66c00>, #<Class:#<X:0x0000000001f679e8>>, Mod, X, Object, Kernel, BasicObject]

p x.homu
# => "homuhomuhomu"
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2-4"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="10"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="15"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11,18"></span>


---

#### `#extend` まとめ
- - -

* #extend はレシーバの特異クラスの継承リストに追加される  <!-- .element: class="fragment" -->
* 正確にいえばメソッド呼び出しの優先順位はレシーバの特異クラスの継承リストになる  <!-- .element: class="fragment" -->
* 特異クラスに対して直接 mixin することも出来る  <!-- .element: class="fragment" -->
* extend Mod と singleton_class.include Mod は実質同じ（多分  <!-- .element: class="fragment" -->

---

## 3. `prepend` を利用して簡単なデコレータを実装する

---

## デコレータとは
- - -

任意のメソッドをいい感じにラップしたりメソッド呼び出しに処理をフックする機能(イメージ)


---

## まあ実装してみよう

---

## ライブコーディング

(コードは↓↓↓)

>>>

#### 実装イメージ

```ruby
class X
	def plus a, b
		a + b
	end

	# prepend を利用して X#plus よりも優先順位が高いメソッドを定義
	prepend Module.new {
		def plus *args
			# メソッドの引数を出力
			p args
			# X#plus を呼び出す
			super(*args)
		end
	}
end

x = X.new
p x.plus 1, 2
# => 3
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="7"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="18"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="12"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2-4"></span>

>>>

#### 処理の流れ
- - -

1. `#plus` メソッドを呼び出す
2. `prepend` したモジュールの `#plus` メソッドが呼ばれる
3. そのメソッド内から `super()` を呼び出すことで `X#plus` を呼ぶことが出来る

>>>

#### 汎用化
- - -

```ruby
# クラス定義内で呼び出すことを前提にしているので Module クラスに定義
class Module
	def decorate name, &block
		prepend Module.new {
			define_method name, &block
		}
	end
end

class X
	def plus a, b
		a + b
	end

	decorate(:plus){ |*args|
		p args
		super(*args)
	}
end

p X.new.plus 1, 2
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="3"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="15"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="5,21"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="15"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="17"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11-13"></span>

---

## デコレータの応用

コードは↓↓↓

>>>

#### 予めラップするメソッドを用意してく
- - -

```ruby
def print_args
	proc { |*args|
		p "args : #{args}"
		super(*args)
	}
end


def memoize
	cache = {}
	proc { |*args|
		cache[args] = super(*args) unless cache.has_key? args
		cache[args]
	}
end
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2,11"></span>

>>>

#### 引数をログ出力
- - -

```ruby
class X
	def plus a, b
		a + b
	end
	# 引数のログを出力するようにする
	decorate :plus, &print_args

	# def の戻り値はメソッド名なのでこういう風に書くことも
	decorate def minus a, b
		a - b
	end, &print_args
end

x = X.new

p x.plus(1, 2)
p x.minus(1, 2)
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="6"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="9-11"></span>

>>>

#### メソッドをメモ化
- - -

```ruby
class X
	# フィボナッチ数列をメモ化
	def fibonacci n
		n > 1 ? fibonacci(n - 2) + fibonacci(n - 1) : n
	end
	decorate :fibonacci, &memoize
end


x = X.new

p x.fibonacci 40
# = >102334155
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="3"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="6"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="12"></span>


---

# まとめ

---

## まとめ
- - -

* .ancestors で継承リストを取得する事が出来る  <!-- .element: class="fragment" -->
* メソッドの優先順位は（特異クラスの）継承リストに依存する  <!-- .element: class="fragment" -->
* mixin も継承リストに追加しているに過ぎない  <!-- .element: class="fragment" -->
* #prepend で任意のメソッドに対して簡単に処理をフックする事が出来る  <!-- .element: class="fragment" -->

---


## 参照
- - -

* [今更聞けない! Ruby の継承と mixin の概念を継承リストから学ぶ - Secret Garden(Instrumental)](http://secret-garden.hatenablog.com/entry/2017/09/11/4756)
* [クラスとモジュールの違い - Secret Garden(Instrumental)](http://secret-garden.hatenablog.com/entry/2017/12/07/001029)

---

## ご清聴
## ありがとうございました
