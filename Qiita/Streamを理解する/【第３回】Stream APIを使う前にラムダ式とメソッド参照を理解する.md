出所：[【第三回】Stream APIを使う前にラムダ式とメソッド参照を理解する #java8 - Qiita](https://qiita.com/tech_newbie/items/2588970f3d2ce08a1b16)

取得年月日：2023(R5).11.13

[@tech_newbie(tech newbie)](https://qiita.com/tech_newbie)

# 【第三回】Stream APIを使う前にラムダ式とメソッド参照を理解する

- [ラムダ式](https://qiita.com/tags/ラムダ式)
- [java8](https://qiita.com/tags/java8)
- [Java入門](https://qiita.com/tags/java入門)
- [StreamAPI](https://qiita.com/tags/streamapi)

最終更新日 2019年11月30日投稿日 2019年11月30日

# はじめに

この投稿は、Java8におけるStream APIを活用する為の事前知識として書いています。
Streamを理解するためには複数の知識が必要なので、下記流れで投稿していこうと思っています！
今回はその三回目です！

- 第一回　[ジェネリクス](https://qiita.com/tech_newbie/items/6adcb7dc5f59165f3fb9)
- 第二回　[関数型インターフェース](https://qiita.com/tech_newbie/items/083cf1ffcfa9744b0c88)
- 第三回　[ラムダ式とメソッド参照](https://qiita.com/tech_newbie/items/2588970f3d2ce08a1b16)
- 第四回　[Stream API](https://qiita.com/tech_newbie/items/3ee76c639f6a1a311174)

今回はジェネリクスや関数型インターフェースの知識が必要になるので過去の投稿もご覧頂ければと思います。

# ラムダ式とは

ラムダ式は**メソッド定義を変数のように扱うことができる**ことが大きな特徴です。
そして、**関数型インターフェースととても相性がいいです**。
というのも、ラムダ式を用いることで、関数型インターフェースの抽象メソッドの実装が従来の無名クラスを用いる場合より簡単に記述することができます。

ここで
**無名クラス?**
**関数型インターフェースの抽象メソッド?**

と思われる方もいるかもしれません。
その場合は、[前回の投稿](https://qiita.com/tech_newbie/items/083cf1ffcfa9744b0c88)を参照頂けると理解しやすいかと思います。

# ラムダ式の使われ方

前回の投稿では、関数型インターフェイスを無名クラスを用いて実装していました。
コードを再度見てみましょう。

SampleIf.java

```
//関数型インターフェースの定義
@FunctionalInterface
public interface SampleIf {
    public abstract Integer countLength(String str);
}
```

Sample.java

```
public static void main(String[] args) {
    SampleIf count = new SampleIf() {
        //抽象メソッド(countLength)を実装
        @Override
        public Integer countLength(String str) {
            return str.length();
        };
    };
    //実行
    System.out.println("カウント結果：" + count.countLength("テスト")); // -> カウント結果：3
}
```

- **文字列を受け取って**
- **その文字列の数を数えるだけの実装でした。**

無名クラスを用いる場合、わざわざnewして抽象メソッドをOverride(実装)しています。

この投稿では**実装にラムダ式を使って簡潔にしよう**ということが目的となります。
実際にラムダ式を使ったパターンを見てみましょう。

LambdaSample.java

```
public static void main(String[] args) {
    //SampleIfの抽象メソッド(countLength)を実装
    SampleIf lambdaSample = (String str) -> {return str.length();}; 
    //実行
    System.out.println("カウント結果：" + lambdaSample.countLength("テスト")); // -> カウント結果：3
}
```

もちろん**`java.util.function`****パッケージ内の関数型インターフェース**も使用できます。

LambdaFncSample.java

```
public static void main(String[] args) {
    //ToIntFunctionの抽象メソッド(applyAsInt)を実装
    ToIntFunction<String> lambdaSample = (String str) -> {return str.length();}; 
    //実行
    System.out.println("カウント結果：" + lambdaSample.applyAsInt("テスト")); // -> カウント結果：3
}
```

そして、コード内のこの部分がラムダ式です。

```
(String str) -> {return str.length();};
```

無名クラスと比べてだいぶスッキリとした記述になったのではないでしょうか！
このように関数型インターフェースの抽象メソッドを簡潔に実装できるのがラムダ式のポイントです。

# ラムダ式内の記述方法

ラムダ式の基本的な記述方法は

```
( 引数 ) -> { 処理 }
```

といった形になります。

# 省略した記述方法

先程のコードを更に簡潔に短く書くことができます。
先程のコードのラムダ式の部分に注目して下さい。

省略前

```
(String str) -> {return str.length();};
```

- 省略ポイント1
  - **引数の型は省略可能 (型推論される為)**

省略ポイント1

```
(str) -> {return str.length();};
```

- 省略ポイント2
  - **渡された引数が1つであり、その型が型推論される場合は引数の`()`を省略可能**

省略ポイント2

```
str -> {return str.length();};
```

- 省略ポイント3
  - **処理が1行である場合、`return`と`{}`は省略可能**

省略ポイント3

```
str -> (str.length());
```

※returnだけ、{ }だけ省略ということはできないので、必ずセットで省略すること

ここまでの省略記法を使うと最終的には下記のコードになりますね。
だいぶスッキリすると思います。

最終段階

```
//省略前
ToIntFunction<String> lambdaSample = (String str) -> {return str.length();};

//省略後
ToIntFunction<String> lambdaSample = str -> str.length();
```

※注意点
ラムダ式での実装において、引数がない場合の`( )`は省略できません。
そういった場合は **`() -> 処理`** といった形になります。

```
//引数がなく、"test"という文字列を返すだけの実装
Supplier<String> supplySample = () -> "test"; 
```

※`Supplier<T>`は`java.util.function`パッケージ内で用意されている、
引数がなく、T型の戻り値を返す関数型インターフェースです。

# 変数のスコープ

ラムダ式内では無名クラスと同様に外部の変数を参照する場合、その変数は**final**もしくは**実質的final**でなければなりません。
ただし、**Java8では変数の宣言時に値が代入され、それ以降変更されていない変数を実質的にfinalとみなす**ことができるようになっている為、ラムダ式内で変数を参照すること自体は問題ありません。

コードで実際に見てみると下記になります。

スコープ例（参照は可能）

```
public static void main(String[] args) {
    //finalを省略
    int num = 8;

    Consumer<String> test = str -> {
        System.out.println(str + num); //変数の参照は可能
    };
    test.accept("Java"); //Java8
}
```

スコープ例（書き換えは不可）

```
public static void main(String[] args) {
    //finalを省略
    int num = 8;

    Consumer<String> test = str -> {
        num = 13; //実質的にfinalの変数を書き換えようとしている為、コンパイルエラー
        System.out.println(str + num); 
    };
    test.accept("Java");
}
```

# ラムダ式での実装パターン例 (java.util.function)

[第二回](https://qiita.com/tech_newbie/items/083cf1ffcfa9744b0c88)では**`java.util.function`**内に定義されている関数型インターフェースを紹介しました。
ここではラムダ式を使った場合の実装例をご紹介します。

### Consumer<T>

抽象メソッド: **`void accept(T t)`** ※T型の引数を1つ受け取り、戻り値を返さない

```
Consumer<String> lambda1 = str -> System.out.println(str);
lambda1.accept("テスト"); // -> テスト
```

### Supplier<T>

抽象メソッド: **`T get()`** ※引数がなく、T型の戻り値を返す

```
Supplier<String> lambda2 = () -> "test";
System.out.println(lambda2.get()); // -> test
```

### Function<T,R>

抽象メソッド: **`R apply(T t)`** ※T型の引数を1つ受け取り、R型の戻り値を返す

```
Function<String,Integer> lambda3 = str -> str.length();
System.out.println(lambda3.apply("テスト")); // -> 3
```

### UnaryOperator<T>

抽象メソッド: **`T apply(T t)`** ※T型の引数を1つ受け取り、同じT型の戻り値を返す

```
UnaryOperator<String> lambda4 = str -> "[" + str + "]";
System.out.println(lambda4.apply("テスト")); // -> [テスト]
```

### Predicate<T>

抽象メソッド: **`boolean test(T t)`** ※T型の引数を1つ受け取り、booleanの値を結果として返す

```
Predicate<Integer> lambda5 = num -> num % 2 == 0;
System.out.println(lambda5.test(2)); // -> true
```

# メソッド参照

ここまでラムダ式について紹介しましたが、場合によってはラムダ式より簡潔に書くことができる方法があります。
それが**メソッド参照**です。

メソッド参照を使うことで既に定義されているメソッドを引数を記述することなく呼び出すことができます。

# メソッド参照の記述方法

- staticメソッドを呼び出す場合
  - **クラス名 :: メソッド名**
- インスタンスメソッドを呼び出す場合
  - **インスタンス変数名 :: メソッド名**

```
//ラムダ式の場合
ToIntFunction<String> lambdaFnc1 = str -> str.length();

//メソッド参照の場合
ToIntFunction<String> methodRef1 = String :: length;
```

# メソッド参照の使いどころ

メソッド参照は便利ですが、ラムダ式の代わりとしてどこでも使えるという訳ではありません。

**ラムダ式がstaticメソッドまたはインスタンスメソッドを
直接的に呼び出している(ラムダ式がただ引数を渡している)場合において、代替手段となります。**

結果を返す前にその値を修正しなくてはならない場面においてはメソッド参照は利用できません。

代替手段となる例をご紹介します。

```
//ラムダ式の場合
ToIntFunction<String> lambda1 = str -> str.length();
UnaryOperator<String> lambda2 = str -> str.toUpperCase();
ToIntFunction<String> lambda3 = str -> Integer.parseInt(str);
BinaryOperator<Integer> lambda4 = (num1,num2) -> Integer.sum(num1,num2);

//メソッド参照の場合
ToIntFunction<String> methodRef1 = String :: length;
UnaryOperator<String> methodRef2 = String :: toUpperCase;
ToIntFunction<String> methodRef3 = Integer :: parseInt;
BinaryOperator<Integer> methodRef4 = Integer :: sum;
```

# まとめ

- ラムダ式を使うことで関数型インターフェースの抽象メソッドの実装を簡潔に記述ができる
- ラムダ式は省略記法で記述する
- メソッド参照はラムダ式がメソッドを直接的に呼び出している場合において、代替手段として利用できる

# おわりに

引数としてラムダ式を受け取るメソッドを定義したり他にも様々な活用方法がありますが、
Stream APIの事前知識として深いところまでは突っ込まないように記述しました。

次回は本題のStream APIとなります！
こちらも合わせてご覧頂ければと思います！