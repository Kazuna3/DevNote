出所：[徹底解説! Javaのラムダ式はどういうもので、どう使えばいいのか! (bold.ne.jp)](https://www.bold.ne.jp/engineer-club/java-lambda-expression)

最終更新日：2019(R1).12.25

著者情報：大石 英人、開発エンジニア/Java20年/Java GOLD/リーダー/ボールド歴2年

この道一筋20年。情報システムについてなら、構築・運用・保守、なんでもござれなエンジニア。システムやデータベースの設計、ソースコードの品質には一家言あり。気持ちはまだまだ若いので、若い世代のエンジニアと一緒に成長していきたい。

取得年月日：2023(R5).10.26

# 徹底解説! Javaのラムダ式はどういうもので、どう使えばいいのか!

Javaのラムダ式(lambda expression)とは、関数型インターフェイスを実装したクラスのインスタンスを、ごく短いコーディング量でとても簡単に作れてしまう文法のことです。

ラムダ式は、Java 8から追加された比較的新しい機能です。ラムダ式ってそもそも何だろうと思って調べると、大抵はStream APIとセットでの説明で、覚えることがものすごく多そうですよね。

そんな印象から、Javaのラムダ式に難しい印象を持っているそこのあなた、それはまったくの間違いです。ラムダ式そのものは、ポイントを押さえれば、実はもっと簡単に分かるものなのです。

この記事では、Javaのラムダ式とは本当はどういうものかという考え方から、プログラムでの使いどころなどを、Javaの初心者や、Javaからしばらく離れていた人向けに、一つずつ解説します。

※この記事はJava 13時点の言語仕様・APIに基づいています。サンプルはJava 13の環境で動作確認しています。



目次

- １．ラムダ式の勘所
  - [１-１．ラムダ式はクラス定義とインスタンス生成をお手軽にやる文法](#01)
  - [１-２．関数型インターフェイスは、抽象メソッドを1つだけ持つ](#02)
  - [１-３．ラムダ式でプログラムが短く、分かりやすくなる](#03)
- ２．クラスがラムダ式になるまで
  - [２-１．普通のクラスから匿名クラスになるまで](#04)
  - [２-２．匿名クラスがラムダ式になるまで](#05)
  - [２-３．ラムダ式は関数型インターフェイスがあってこそ](#06)
  - [２-４．型は引数や戻り値などからも分かる](#07)
  - [２-５．Javaコンパイラはラムダ式をこう見ている](#08)
- ３．ラムダ式のお約束いろいろ
  - [３-１．ラムダ式の基本形](#09)
  - [３-２．ラムダ式の中で使える変数](#10)
  - [３-３．ラムダ式とオートボクシング・プリミティブ型の拡大変換](#11)
  - [３-４．ラムダ式と例外](#12)
  - [３-５．ラムダ式と変数のスコープ](#13)
- ４．ラムダ式の使われ方
  - [４-１．Stream APIの引数](#14)
  - [４-２．スレッド関連APIでのRunnable/Callable](#15)
  - [４-３．関数型インターフェイスの動作カスタマイズ](#16)
  - [４-４．その他の使われ方](#17)
- ５．【発展】ラムダ式・関数型インターフェイスと部品化の考え方
  - [５-１．メソッドによる部品化](#18)
  - [５-２．関数型インターフェイスによる部品化](#19)
  - [５-３．関数型インターフェイスなら処理を差し替えできる](#20)
  - [５-４．Javaでの関数は関数型インターフェイスのインスタンス](#21)
  - [５-５．ラムダ式が関数を作るハードルを下げる](#22)
- [６．まとめ](#23)



## １．ラムダ式の勘所

この章では、まずは皆さんにラムダ式の本当の勘所を知っていただきます。もうそれはばっちり分かっているよ、という方は「3.ラムダ式のお約束いろいろ」以降をお読みください。

ラムダ式へよくされるちょっと分かりづらい説明、例えば「関数を定義する」「メソッドを変数のように扱える」などは、一旦忘れましょう。この記事を最初から素直に読んでみてください。

インターフェイスとそれを実装したクラスという、Javaでのごく基本的な事柄。それだけ分かっていれば、Javaのラムダ式はそこから考え方を少し発展させるだけで理解できるのです。

<a id="01"></a>

### １-１．ラムダ式はクラス定義とインスタンス生成をお手軽にやる文法

Javaのラムダ式は、関数型インターフェイスを実装したクラスのインスタンスを、簡単に作るための文法です。言い換えれば、クラスの宣言とインスタンスの生成を同時に行う文法なのです。

以下のような、何かのインターフェイスを実装したクラスがあり、そのクラスのインスタンスを生成して使いたいとします。ここでの例では、java.util.Comparatorを使います。

```
import java.util.Comparator;

class ComparatorImpl implements Comparator<String> {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
}
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = new ComparatorImpl();
        System.out.println(c.compare("ABC", "DEF")); // → -1が表示される
    }
}
```

これを以下のように1行で書けます。ラムダ式は「(s1, s2) -> s1.compareTo(s2)」の部分です。最初のプログラムと同じように、インターフェイスが持つメソッドcompareが呼べていますね。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = (s1, s2) -> s1.compareTo(s2);
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

Javaのラムダ式は、本質的にはこれだけ!! 名無しのクラスを定義し、そのクラスのインスタンスを生成するものです。書き方のお作法や制限事項が少々ありはしますが、それは二の次です。

本来は複数行で行うクラスの定義が、なぜたった1行にできるのか。その理由はぱっと見では分からないですよね。この記事の続きでは、その仕組みをしっかりと説明していきます。

<a id="02"></a>
### １-２．関数型インターフェイスは、抽象メソッドを1つだけ持つ

前の節では「関数型インターフェイス」という用語をいきなり使いました。関数型インターフェイス(functional interface)とは、「抽象メソッドを1つだけ持つインターフェイス」のことです。

ラムダ式を学ぶ上で絶対に忘れてはならないのは、この「1つだけ」というところ。この特徴がラムダ式の理解には必須、かつラムダ式とは不可分なのです。その理由は後述します。

関数型インターフェイスとしては、例えば標準APIでは以下のものがあります。それに、抽象メソッドが1つだけあるインターフェイスに過ぎませんから、自分で作ることも当然できるのです。

> java.lang.Comparable
>
> java.lang.Runnable
>
> java.util.Comparator
>
> java.util.concurrent.Callable
>
> java.util.function.Consumer
>
> java.util.function.Function
>
> java.util.function.Predicate
>
> java.util.function.Supplier
>
> …など多数

ちなみに、関数型インターフェイスに関連する文章では、いろいろなところでSAM(サム)が出てきます。SAMは「Single Abstract Method」の略で、「1つだけの抽象メソッド」ということです。

#### １-２-１．関数型インターフェイスの例:RunnableとComparator

ここでは、java.Runnableとjava.util.Comparatorのインターフェイス定義を見てみましょう。これらはJava 13のソースコードから持ってきて、Javadoc部分などを削除したものです。

```
package java.lang;

@FunctionalInterface
public interface Runnable {
    public abstract void run(); // ←これがRunnableの抽象メソッド
}
package java.util;

※import文は省略

@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2); // ←これがComparatorの抽象メソッド

    boolean equals(Object obj);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    ※以下、default/staticメソッドの宣言がずっと続く…
}
```

それぞれ抽象メソッドは一つだけです(runとcompare)。ただ、ComparatorにはObjectのメソッド(equals)、抽象メソッド以外のメソッド(staticメソッド、defaultメソッド)があります。でも、これらは今は無視して構いません。

<a id="03"></a>
### １-３．ラムダ式でプログラムが短く、分かりやすくなる

ラムダ式を使うとうれしいのは、以下が同時に実現できることです。いずれも、プログラムの読みやすさ、いわゆる可読性の向上につながるものです。

1. 抽象メソッドを実装したクラスを最小のコード量で実装できる
2. 抽象メソッドの実装と、メソッドを使うところを一つにできる

ただし、2.はラムダ式の二次的な特徴です。でも、それを意識して使うと便利で分かりやすいよと、ラムダ式を考えた人たちがお勧めしているだけなので、勘違いしないようにしましょう。

#### １-３-１．ラムダ式で実現できる「プログラムの分かりやすさ」とは?

ラムダ式の分かりやすさとは、抽象メソッドの実装が「そのままそこに」書いてあることです。何をしているかを調べるために、別のファイルをわざわざ見に行かなくてもいいのです。

例えば、1-1の2つのプログラムは、どちらの方が「やっていることが分かりやすい」と言えそうでしょうか。

最初のプログラムでは、Comparatorの実装クラスとそれを使うクラスは別のファイルです。ですから、別ファイルのメソッド実装を見ないと実際の処理は分からないので、少し面倒です。

後の方でのラムダ式を使ったプログラムでは、処理の内容が「そのままそこに」書いてあります。この違いは、読みやすさの面からはなかなか大きいですよ。

## ２．クラスがラムダ式になるまで

この章では、普通のクラスがラムダ式になる過程を一つずつ追っていきます。この手順をしっかり一つずつ理解していけば、「ラムダ式脳」になれること間違いなしです。

そして、先ほどお伝えした関数型インターフェイスが「抽象メソッドを1つだけ持つ」という性質が、途中で大変役立つタイミングがあります。それがどこかを楽しみにしておいてください。

<a id="04"></a>
### ２-１．普通のクラスから匿名クラスになるまで

#### ２-１-１．普通のクラス

まずインターフェイスを実装した普通のクラスから始めます。これが分からないなら、ラムダ式に挑戦するのは早すぎます。申し訳ありませんが、Javaの基本から勉強してきてくださいね。

**【参考】java.util.Comparatorのインターフェイス定義(抜粋)**

```
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

**ComparatorImpl.java**

```
import java.util.Comparator;

class ComparatorImpl implements Comparator<String> {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
}
```

**ComparatorImplTest.java**

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = new ComparatorImpl();
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

#### ２-１-２．内部クラス

次に、それぞれ別のファイル(.java)だったのを、一つのファイルにしました。staticな内部クラス(inner class)になっていますが、mainから呼ぶためなので、本質的な違いはありません。

**ComparatorImplTest.java**

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = new ComparatorImpl();
        System.out.println(c.compare("ABC", "DEF"));
    }

    static class ComparatorImpl implements Comparator<String> {
        public int compare(String s1, String s2) {
            return s1.compareTo(s2);
        }
    }
}
```

#### ２-１-３．ローカルクラス

さらに、内部クラスをローカルクラス(local class)にします。ローカルクラスはあまり使ったことがない方が多いかもしれませんが、こういう書き方もJavaではありなのです。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        class ComparatorImpl implements Comparator<String> {
            public int compare(String s1, String s2) {
                return s1.compareTo(s2);
            }
        }
        Comparator<String> c = new ComparatorImpl();
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

#### ２-１-４．匿名（無名）クラス

そして、ローカルクラスを匿名クラス(anonymous class)にします。クラスの宣言とインスタンス生成(new)が1行になり、クラス名も消えました。この匿名クラスのクラス名は、Javaコンパイラが勝手に命名します。

これがラムダ式のベースになるものです。つまり、ラムダ式は特殊な書き方で匿名クラス(のようなもの)を作る文法だとも言えるのです。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = new Comparator<String>() {
            public int compare(String s1, String s2) {
                return s1.compareTo(s2);
            }
        };
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

さて皆さん、ここまではついてこれていますか? では、どんどん飛ばしていきますよ!!

<a id="05"></a>
### ２-２．匿名クラスがラムダ式になるまで

では、匿名クラスとなったところから、いらないものを削っていきましょう。最終的にラムダ式になるまでコンパイルは通りませんので、くれぐれも注意してください。

ちなみにここで「いらないもの」とは、プログラム上のどこかから得られる情報で推測ができるので、わざわざプログラム上に書かなくてもいいのではないか?と考えられるものです。

**【参考】java.util.Comparatorのインターフェイス定義(抜粋)**

```
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

#### ２-２-１．インターフェイス名を削除する

まずは、代入演算子の右辺にあるインターフェイス名を削除しました。

```
Comparator<String> c = new {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

代入演算子左辺の変数宣言での型と、右辺のインターフェイスを指定した部分が同じ(Comparator<String>)なので、同じことを二度書く必要はないからです。

#### ２-２-２．抽象メソッドのアクセス修飾子を削除する

次に、実装メソッドのアクセス修飾子であるpublicを削除しました。

```
Comparator<String> c = new {
    int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

Javaではインターフェイスの抽象メソッドは例外なくpublicなので、記述を省略してもその意味は変わらないからです。

#### ２-２-３．抽象メソッドの戻り値の型を削除する

実装メソッドの戻り値のintを削除しました。

```
Comparator<String> c = new {
    compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

Comparator<T>には、抽象メソッドがint compare(T o1, T o2)の「1つしか」ありませんので、戻り値の型はint以外にありえないからです。

#### ２-２-４．抽象メソッドのメソッド名を削除する

メソッド名も削除してしまいます。

```
Comparator<String> c = new {
    (String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

Comparatorの抽象メソッドはcompareの「1つしか」ないので、それしか選びようがないからです。

#### ２-２-５．抽象メソッドの引数の型を削除する

メソッドの引数の型(String)も削除しました。

```
Comparator<String> c = new {
    (s1, s2) {
        return s1.compareTo(s2);
    }
};
```

左辺のローカル変数の型から、Comparator<T>への実型引数がStringだと分かるので、メソッドの引数の型もStringで確定するからです。

#### ２-２-６．抽象メソッドのreturnを削除する

メソッドの中にあるreturnを削除しました。

```
Comparator<String> c = new {
    (s1, s2) {
        s1.compareTo(s2);
    }
};
```

「s1.compareTo(s2)」の戻り値はintです。ラッキーなことに、intはメソッドの戻り値の型と同じで、そのままreturnしているだけです!! なので、returnしていることにしちゃいましょう。

#### ２-２-７．抽象メソッドの{}と;を削除する

さらに、メソッド本体を囲っている括弧({})と、文を区切る;も一緒に削除しました。

```
Comparator<String> c = new {
    (s1, s2) 
        s1.compareTo(s2)
};
```

どうせメソッドの中身はreturnする1行しかないのですから、メソッドで複数行を書くために必要なだけの{}や;なんて、ぜんぜんいらないのです。

#### ２-２-８．匿名クラスの{}を削除する

今度は匿名クラスの括弧{}を削除しました。そろそろ終わりが見えてきましたね。

```
Comparator<String> c = new
    (s1, s2) 
        s1.compareTo(s2)
;
```

このクラスにはComparatorの抽象メソッドの実装だけあり、フィールドや他のメソッドがありません。ですから、複数のフィールドやメソッドを1つのクラスにまとめる括弧はいりません。

#### ２-２-９．newを削除する

ここまで来ると、もうnewもいりませんね。

```
Comparator<String> c =
    (s1, s2) 
        s1.compareTo(s2)
;
```

元々の匿名クラスの構文そのものが、クラスから作られたインスタンスを戻す構文なのですから。これもわざわざ書く必要はなくて、そういうことにしてしまえばいいのです。

#### ２-２-10．ラムダ式の完成

というわけで、最後まで残ったのは、抽象メソッドの引数の名前と、抽象メソッドの中身だけ。これにラムダ式だと見分けるための「->」を挟んで1行にすれば、めでたくラムダ式の完成です。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = (s1, s2) -> s1.compareTo(s2);
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

<a id="06"></a>
### ２-３．ラムダ式は関数型インターフェイスがあってこそ

この例で重要な役割を果たしたのは、代入演算子の左辺の変数の型「Comparator<String>」です。この型が関数型インターフェイスの決まりごとを守っているので、ラムダ式が成り立つのです。

関数型インターフェイスには1つしか抽象メソッドがないので、戻り値・引数の型と順番を、関数型インターフェイスの型からJavaコンパイラが推測できます。この仕組みを型推論といいます。

ラムダ式の裏側では、クラスの変形手順の一番最初に出てきた、関数型インターフェイスを実装した匿名クラスをJavaが自動で作り、そのクラスのインスタンスを新しく生成して戻しています。

ですから、今なら以下のラムダ式がコンパイルエラーになる理由が分かるでしょう。なぜなら、ラムダ式の対象となる関数型インターフェイスが何か、Javaコンパイラが分からないからですね。

```
class ComparatorImplTest {
    public static void main(String[] args) {
        (s1, s2) -> s1.compareTo(s2);
    }
}
```

さらに、変数の型が関数型インターフェイスでありさえすればいいので、以下のようにフィールドの宣言と初期化の時にも使えますし、変数に再代入をする時にも使えるのです。

```
import java.util.Comparator;

public class ComparatorImplTest {
    Comparator<String> c1 = (s1, s2) -> s1.compareTo(s2);
    static Comparator<String> c2 = (s1, s2) -> s1.compareTo(s2);
}
Comparator<String> c = null;
c = (s1, s2) -> s1.compareTo(s2);
```

<a id="07"></a>
### ２-４．型は引数や戻り値などからも分かる

この章では、ローカル変数の型をヒントに、匿名クラスをラムダ式に変形しました。でも、Javaで型が出てくる所は、ローカル変数の宣言以外だと、メソッドの引数と戻り値もありますよね。

ですから、以下のような関数型インターフェイスを引数にするメソッドがあるなら、そのメソッドを呼び出すところでもラムダ式が書けるのです。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        lamdaMethod((s1, s2) -> s1.compareTo(s2));
    }

    static void lambdaMethod(Comparator<String> c) {
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

なぜなら、ラムダ式の対象となる関数型インターフェイスが何かが、メソッドの引数の型からJavaコンパイラが分かるからです。これは、メソッドの引数へも型推論が働くということです。

つまり、以下のプログラムとやっていることは同じです。この匿名クラスのラムダ式への変形手順は、ローカル変数を相手にしていた時とまったく同じです。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        lamdaMethod(
            new Comparator<String>() {
                public int compare(String s1, String s2) {
                    return s1.compareTo(s2);
                }
            }
        );
    }

    static void lamdaMethod(Comparator<String> c) {
        System.out.println(c.compare("ABC", "DEF"));
    }
}
```

さらに、メソッドの戻り値の型も、ラムダ式の対象となる関数型インターフェイスを探るヒントとしては有効です。つまり、ラムダ式は以下のようにも書けてしまうのです。

```
import java.util.Comparator;

class ComparatorImplTest {
    public static void main(String[] args) {
        Comparator<String> c = getComparator();
        System.out.println(c.compare("ABC", "DEF"));
    }

    static Comparator<String> getComparator() {
        return (s1, s2) -> s1.compareTo(s2);
    }
}
```

マニアックですが、キャストでの変数の型も、Javaコンパイラがラムダ式の関数型インターフェイスとして認識してくれる場合があります。ただ、これが必要な箇所はあまり思いつきません。

```
Object o = (Runnable)() -> System.out.println("ABC");

if (o instanceof Runnable) {
    System.out.println("Runnableでーす"); // → oの実体はRunnableなのでこちらに来る
}
```

<a id="08"></a>
### ２-５．Javaコンパイラはラムダ式をこう見ている

Javaコンパイラがラムダ式を見つけたら、ラムダ式が書かれているところで扱われる「型」を最初に判断します。型の参照先は、変数の型、メソッドの引数の型、メソッドの戻り値の型です。

Javaコンパイラが見つけた型が関数型インターフェイスなら、その抽象メソッドの引数の型や数、戻り値の型などを認識して、ラムダ式で書かれていることとの整合性をチェックしていきます。

Javaコンパイラはこんな感じで考えているので、ラムダ式でコンパイルエラーが出た時は、まずは型の観点で見直しましょう。ジェネリクスがからむ場合は少々ややこしいですが、一つずつ追っていけば、ダメなところは必ず分かります。

ちなみに、Javaは静的型付き言語と呼ばれるタイプのプログラミング言語です。静的型付き言語では、変数、引数、戻り値すべてに何かの型があり、プログラム上で明確に宣言されています。

ですから、ラムダ式と関数型インターフェイスで、「型」で決まっていることがすべて一致しているか…の観点があれば、ラムダ式は不思議なものでも、訳が分からないものでもないのです。

## ３．ラムダ式のお約束いろいろ

ここまでの内容で、ラムダ式とは「関数型インターフェイスを実装したクラスのインスタンスを簡単に作るための文法」だということがお分かりいただけたでしょうか。

この章では、ラムダ式を使う上で覚えておくべきいろいろな仕様について、ざっと説明します。

<a id="09"></a>
### ３-１．ラムダ式の基本形

ラムダ式の基本形は「①引数の宣言部 -> ②抽象メソッドの本体部」です。ここで注目すべきは「->」(アロー演算子)です。Javaで -> が出たなら、それはラムダ式です。

①引数の宣言部と②抽象メソッドの本体部には、以下のパターンがあります。Javaのラムダ式は、この①と②のどれかの組み合わせで作られているのです。

#### ３-１-１．引数の宣言部

<table style="border-collapse: collapse; width: 100%; height: 168px;">
  <tbody>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">パターン</td>
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">引数部の書き方</td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">抽象メソッドに引数がない場合</td>
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">()</td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 72px; border-color: #000000; border-style: solid;" rowspan="3">抽象メソッドの引数が<span>1</span>つだけの場合</td>
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">(引数の型 引数の変数名<span>)</span></td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">(引数の変数名<span>)</span></td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">引数の変数名</td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 48px; border-color: #000000; border-style: solid;" rowspan="2">抽象メソッドの引数が<span>2</span>つ以上の場合</td>
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">(引数の型<span>1 </span>引数の変数名<span>1, </span>引数の型<span>2 </span>引数の変数名<span>2, …)</span></td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 50%; height: 24px; border-color: #000000; border-style: solid;">(引数の変数名<span>1, </span>引数の変数名<span>2, …)</span></td>
    </tr>
  </tbody>
</table>
どのパターンでも、必須なのは引数の変数名です。これはラムダ式を理解する上では大事なことなので、覚えておきましょう。

そして、引数の型が省略されても、その変数の型が何かはJavaコンパイラは分かっています。ですから、メソッド本体部ではその変数の型が持つフィールドやメソッドを全部使えます。

ちなみに、引数の型が書かれているなら、finalなどの修飾子や注釈(アノテーション)をつけることもできます。逆に、変数名のみだと修飾子・アノテーションは書けません。

なお、Java 11以降なら引数の型の代わりにvar(ローカル変数型推論でのキーワード)も使えます(JEP323)。ただ、ラムダ式は引数の型を元々省略できるので、varとする意味はあまりありません。

#### ３-１-２．抽象メソッドの本体部

<table style="border-collapse: collapse; width: 100%; height: 216px;">
  <tbody>
    <tr style="height: 48px;">
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;" rowspan="3">抽象メソッドの戻り値の型が<span>void</span>の場合</td>
      <td style="width: 33.3333%; height: 72px; border-style: solid; border-color: #000000;" rowspan="2">メソッドの内容が<span>1</span>行で書ける場合</td>
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;">処理</td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 33.3333%; height: 24px; border-style: solid; border-color: #000000;">{処理<span>;}</span></td>
    </tr>
    <tr style="height: 48px;">
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;">メソッドの内容が<span>2</span>行以上になる場合</td>
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;">{処理<span>1</span>行目<span>; </span>処理<span>2</span>行目<span>; …}</span></td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 33.3333%; height: 24px; border-style: solid; border-color: #000000;" rowspan="3">抽象メソッドの戻り値の型が<span>void</span>ではない場合</td>
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;" rowspan="2">メソッドの内容が<span>1</span>行で書ける場合</td>
      <td style="width: 33.3333%; height: 24px; border-style: solid; border-color: #000000;">戻り値を戻す処理</td>
    </tr>
    <tr style="height: 24px;">
      <td style="width: 33.3333%; height: 24px; border-style: solid; border-color: #000000;">{return 戻り値を戻す処理<span>;}</span></td>
    </tr>
    <tr style="height: 48px;">
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;">メソッドの内容が<span>2</span>行以上になる場合</td>
      <td style="width: 33.3333%; height: 48px; border-style: solid; border-color: #000000;">{処理<span>1</span>行目<span>; </span>処理<span>2</span>行目<span>; …; return </span>処理結果<span>;}</span></td>
    </tr>
  </tbody>
</table>

ラムダ式ならではのことは、{}やreturnを省略できる場合があることです。先述した、抽象メソッドを実装する上で必要最小限なことは何だろう…の視点があれば、納得できるかと思います。

{}があるならメソッドの中身は複数行で書けますし、1行だけでもOKです。普通のメソッドと同じく、改行やインデントは自由です。{}がないなら1行しか書けませんが、最後の;はいりません。

抽象メソッドの戻り値の型がvoidではなく、さらにメソッド本体部に{}がないなら、その文や式の評価結果が戻り値の型と合っていればOKです。これはプログラムでの「評価」の仕組みが分かっていないと、ピンと来ないかもしれませんね。

returnについては、戻り値の型がvoidでも途中でreturnはできますし、voidではないならどこかで必ず戻り値の型をreturnしなければなりません。これらも、普通のメソッドと同じルールです。

#### ３-１-３．引数とメソッド本体の組み合わせ例

以下の例では、すべて同じことをしています。特に注目してもらいたいのは、引数部がどう省略できるかと、メソッド本体の{}やreturnがどういう場合に省略できるかです。

**引数なし、戻り値の型がvoid**

```
Runnable runnable0 = new Runnable() {
    public void run() {
        System.out.println("ABC");
    }
};
Runnable runnable1 = () -> System.out.println("ABC");
Runnable runnable2 = () -> {System.out.println("ABC");};
```

**引数なし、戻り値の型がString**

```
Callable<String> callable0 = new Callable<String>() {
    public String call() {
        return "ABC";
    }
};
Callable<String> callable1 = () -> "ABC";
Callable<String> callable2 = () -> {return "ABC";};
```

**引数1つ、戻り値の型がboolean**

```
Predicate<String> predicate0 = new Predicate<String>() {
    public boolean test(String s) {
        return "ABC".equals(s);
    }
};
Predicate<String> predicate1 = (String s) -> "ABC".equals(s);
Predicate<String> predicate2 = (s) -> "ABC".equals(s);
Predicate<String> predicate3 = s -> "ABC".equals(s);
```

**引数2つ、戻り値の型がInteger**

```
Comparator<Integer> comparator0 = new Comparator<Integer>() {
    public int compare(Integer i1, Integer i2) {
        return i1.compareTo(i2);
    }
};
Comparator<Integer> comparator1 = (Integer i1, Integer i2) -> {return i1.compareTo(i2);};
Comparator<Integer> comparator2 = (i1, i2) -> i1.compareTo(i2);
// ↓コンパイルエラー、型を書くなら、すべての引数に書かなければならない
Comparator<Integer> comparator3 = (Ingeter i1, i2) -> i1.compareTo(i2);
```

**引数1つ、戻り値の型がInteger、メソッド本体が複数行**

```
Function<String, Integer> function0 = new Function<String, Integer>() {
    public Integer apply(String s) {
        if ("ABC".equals(s)) {
            return 1;
        } else {
            return 0;
        }
    }
};
Function<String, Integer> function1 = (String s) -> {
    if ("ABC".equals(s)) {
        return 1;
    } else {
        return 0;
    }
};
Function<String, Integer> function2 = (s) -> {
    if ("ABC".equals(s)) {
        return 1;
    } else {
        return 0;
    }
};
Function<String, Integer> function3 = s -> {
    if ("ABC".equals(s)) {
        return 1;
    } else {
        return 0;
    }
};
```

<a id="10"></a>
### <font color="red">３-２．ラムダ式の中で使える変数</font>

ラムダ式のメソッド本体では、メソッド本体内で宣言したローカル変数や引数以外には、以下の変数を使えます。少し難しいのは「実質的final(effectively final)なローカル変数」です。

- クラスのフィールド(インスタンス/クラスフィールド)
- ローカル変数のうち、ラムダ式の前で宣言済で、実質的finalまたはfinalであるもの

#### ３-２-１．実質的finalなローカル変数

Javaでの変数へのfinal修飾子とは何かを復習しましょう。プリミティブ型変数では代入した変数の値を変えられないこと、参照型変数では参照先のインスタンスを変えられないことでしたね。

それを頭に入れて以下のプログラムを読んでください。このラムダ式はコンパイルエラーです。変数iを宣言して初期値0を代入した後に、iの値をラムダ式の中で変えようとしているからです。

```
import java.util.function.Supplier;

class LambdaTest {
    public static void main(String[] args) {
        int i = 0;
        Supplier<Integer> supplier = () -> {
            i = i + 1; // iの値を変えているので、iは実質的finalではなくなる
            return i;
        };
    }
}
```

変数の値を後から変えられてしまっては、先ほどのfinalの決め事を守れませんよね。ですから、このラムダ式はコンパイルエラーになるのです。

以下のプログラムだと、変数iはfinalではないものの、ラムダ式があるmainメソッドの中では値が変わりません。ですので、「実質的に」finalと同じだと、Javaコンパイラは判断します。

```
import java.util.function.Supplier;

class LambdaTest {
    public static void main(String[] args) {
        int i = 0;
        Supplier<Integer> supplier = () -> i;
    }
}
```

でも、以下のプログラムではラムダ式の外ですが、mainメソッドの中で変数iの値が変わっていますので、実質的finalとはみなされません。そのため、コンパイルエラーになります。

```
import java.util.function.Supplier;

class LambdaTest {
    public static void main(String[] args) {
        int i = 0;
        Supplier<Integer> supplier = () -> i;
        i = 1;
    }
}
```

もちろん、実質的ではなく明示的に変数をfinalとしても何も問題はありません。実質的finalは、ローカル変数にfinal修飾子をいちいちつける手間を省いてくれる、Javaのお助け機能なのです。

```
import java.util.function.Supplier;

class LambdaTest {
    public static void main(String[] args) {
        final int i = 0;
        Supplier<Integer> supplier = () -> i;
    }
}
```

#### <font color="red">３-２-２．ラムダ式の外にある参照型変数には要注意!</font>

参照型変数のfinalとは、変数が指すインスタンスを変えられないということです。ですから、以下のようにすると、ラムダ式の中で配列の内容を書き換えられます。

```
import java.util.function.Consumer;

class LambdaTest {
    public static void main(String[] args) {
        final int[] i = {0, 1, 2};
        System.out.println(i[1]); // → 1

        Consumer<?> consumer = x -> {
            i[1] = 256;
        };
        consumer.accept(null);

        System.out.println(i[1]); // → 256!
    }
}
```

これは、参照型変数iが指しているインスタンスを変更していないからです。iは同じ配列のインスタンスを指し続けていて、その配列の内容を変える操作は、finalには抵触しないのです。

同じように、参照型変数の型にgetter/setterがあったなら、ラムダ式の中でsetterを呼び出すことには何も制限はありません。メソッド呼び出しやフィールドの参照には制限はないからです。

ですから、finalだから変数が指すインスタンスの状態が変わらないというわけではないのです。このようにすると、ラムダ式の中と外とで、間接的に値のやり取りができたりもします。

<font color="red">どうしてもこう書かざるを得ないケースもあリますが、あまりお勧めはされないプログラミングスタイルです。なぜなら、ラムダ式で表現される「関数」が副作用を持つことになるからです。</font>

#### ３-２-３．インスタンスフィールド・クラスフィールド

ラムダ式の中から、インスタンスフィールド(インスタンス変数)とクラスフィールド(static変数)は、自由に使えます。

つまり、普通の変数として扱えるので、プリミティブ型変数は値を変更できますし、参照型変数は参照先を変えられます。以下のラムダ式は、いずれもコンパイルは通り、実行もできます。

```
import java.util.function.Supplier;

class LambdaTest {
    int i = 0;
    static j = 256:

    String s1 = "ABC";
    static String s2 = "あいうえお";

    void instanceMethod() {
        Supplier<Integer> supplier1 = () -> ++i;
        Supplier<Integer> supplier2 = () -> ++j;
        Supplier<String> supplier3 = () -> s1 += "EDF";
        Supplier<String> supplier4 = () -> s2 += "かきくけこ";
    }

    public static void main(String[] args) {
        new LambdaTest().instanceMethod();
    }
}
```

<font color="red">注意したいのは、インスタンスフィールドとクラスフィールドは、ラムダ式の外で予期せぬ変更が行われうること。またラムダ式の中での変数操作により、ラムダ式の外にも影響を及ぼしうることです。</font>

<font color="red">また、ラムダ式で生成した1つのインスタンスが、マルチスレッド環境下で動くこともあります。ですから、フィールドへアクセスする際に、同期ブロックなどでの保護が必要になる場合があります。</font>

後述しますが、ラムダ式は部品のように使われるせいで、いろいろなシチュエーションの下で使われうるからです。

#### <font color="green">３-２-４．【参考】なぜラムダ式で参照できるローカル変数はfinalだけなのか</font>

<font color="green">Javaのラムダ式の中で参照するローカル変数は、なぜ(実質的)finalでなければならないのでしょうか。</font>

<font color="green">それをお伝えするには、Java仮想マシンがローカル変数・参照型変数・スレッドをどう管理しているか、ヒープとスタックの使われ方の違いなど、Java仮想マシンの動きに少なからず触れなければなりません。</font>

<font color="green">その内容はこの記事の範囲を超えますので、詳細には触れません。そこにたどり着くためのキーワードは「クロージャ(closure、関数閉包)」や「変数の束縛(binding)」などです。興味があれば調べてみてもいいでしょう。</font>

<a id="11"></a>
### ３-３．ラムダ式とオートボクシング・プリミティブ型の拡大変換

ラムダ式を使う上では、関数型インターフェイスの抽象メソッドの引数の型・戻り値の型へ行われるオートボクシングとプリミティブ型の拡大変換について、少々注意する事柄があります。

オートボクシング(autoboxing)とは、intとInteger、longとLongのように、プリミティブ型と対応するクラスの間で自動的に相互変換が行われることです。

プリミティブ型の拡大変換(widening primitive conversion)とは、intからlong、longからdoubleなど、より広い値の範囲の型へなら、変換先の型への明示的なキャストがいらないものです。

#### ３-３-１．引数はオートボクシングされない

ラムダ式では引数はオートボクシングされませんので、プリミティブ型と対応するクラスは異なる型として扱われます。

ここで、java.util.function.Consumer<T>の抽象メソッドの宣言は「void accept(T t)」、java.util.function.IntConsumerは「void accept(int value)」です。

```
Consumer<Integer> c1 = (int i) -> System.out.println(i);
Consumer<Integer> c2 = (Integer i) -> System.out.println(i);
IntConsumer c3 = (int i) -> System.out.println(i);
IntConsumer c4 = (Integer i) -> System.out.println(i);
```

これらのラムダ式では、c2とc3はコンパイルが通り、c1とc4はコンパイルエラーです。なぜなら、ラムダ式の引数においては、Integerとintは違うモノだからです。

この理解には、以下のメソッドを見てください。これらは、Javaではオーバーロードされた別のメソッドとして扱われるという仕様が分かっていれば、納得できるでしょうか。

```
void method(int i) {
    System.out.println(i);
}

void method(Integer i) {
    System.out.println(i);
}
```

#### ３-３-２．戻り値でも注意は必要

ラムダ式の戻り値でも、オートボクシングとプリミティブ型の拡大変換を意識すべきケースがあります。

ここで、java.util.function.Supplier<T>の抽象メソッドの宣言は「T get()」、java.util.function.Predicate<T>は「boolean test(T t)」、java.util.LongSupplierは「long getAsLong()」です。

```
Supplier<Integer> s1 = () -> 100;
Supplier<Integer> s2 = () -> Integer.valueOf(100);
Supplier<Long> s3 = () -> 100;
Supplier<Long> s4 = () -> Integer.valueOf(100);
Predicate<String> p = s -> Boolean.valueOf(true);
LongSupplier ls1 = () -> 100;
LongSupplier ls2 = () -> Integer.valueOf(100);
```

s3、s4はコンパイルエラーです。それ以外の行はコンパイルが通ります。

これは、以下をやっているのだと考えてください。以下でl1の行がコンパイルエラーになるのは、プリミティブ型→参照型のオートボクシングでは型変換が行われないケースがあるからです。

```
Integer i1 = 100; // オートボクシング(int→Integer)
Integer i2 = Integer.valueOf(100); // クラスが同じ(Integer同士)
Long l1 = 100; // NG:intはLongへはオートボクシングされない
Long l2 = Integer.valueOf(100); // NG:IntegerとLongは違うクラスなので互換性がない
boolean b = Boolean.valueOf(true); // オートボクシング(Boolean→boolean)
long l3 = 100; // プリミティブ型の拡大変換(int→long)
long l4 = Integer.valueOf(100); // オートボクシング+拡大変換(Integer→int→long)
```

ですので、以下のようにキャストを明示的に行えば、s3とs4でもコンパイルが通ります。

```
Supplier<Long> s3 = () -> (long)100;
Supplier<Long> s4 = () -> (long)Integer.valueOf(100);
```

<a id="12"></a>
### ３-４．ラムダ式と例外

ラムダ式で生成されるクラスは、クラスの定義のタイミングがラムダ式が評価された時というだけなので、普通に定義されたクラスと比べても、機能としては同じものです。

ですから、関数型インターフェイスの抽象メソッドにthrows句があるなら、ラムダ式で生成したインスタンスの抽象メソッドで発生した例外はそのままthrowできます。Javaの文法どおりですね。

**AutoCloseableの定義**

```
package java.lang;

public interface AutoCloseable {
    void close() throws Exception;
}
// ラムダ式でインスタンスを生成した時点ではExceptionはthrowされず、
AutoCloseable c = () -> {throw new Exception();};
// 抽象メソッドを呼び出したタイミングで、Exceptionがthrowされる
c.close();
```

ですから、ラムダ式の抽象メソッドを呼び出した時に発生した例外がどう扱われるかは、ラムダ式のインスタンスの抽象メソッドをどう呼び出すかによります。当たり前と言えば当たり前です。

例えば、以下のmethod2とmethod5がコンパイルエラーになる理由は分かるでしょうか。メソッド呼び出しで発生しうるExceptionを、メソッドの中でcatchもthrowもしていないからですね。

```
void method1() throws Exception {
    AutoCloseable c = () -> {throw new Exception();};
    c.close();
}

void method2() {
    AutoCloseable c = () -> {throw new Exception();};
    c.close();
}

void method3() {
    AutoCloseable c = () -> {throw new Exception();};
}

void method4(AutoCloseable c) throws Exception {
    c.close();
}

void method5(AutoCloseable c) {
    c.close();
}
```

ですので、ラムダ式であっても、例外を扱う上では何も特別なことはないのです。

<a id="13"></a>
### ３-５．ラムダ式と変数のスコープ

ここまで見てきたとおり、ラムダ式と匿名クラスはほぼ同じ考え方ができます。ですが、thisが指すものと変数のスコープなどの違いもありますので、それらを簡単に説明します。

#### ３-５-１．thisが指すものが違う

Javaでのthisは、自分自身のインスタンスを指すものです。ここでラムダ式と匿名クラスのthisは、指しているインスタンスが違います。

```
class LambdaTest {
    public static void main(String[] args) {
        new LambdaTest().instanceMethod();
    }

    void instanceMethod() {
        System.out.println("インスタンスメソッドでのthis: " + System.identityHashCode(this));

        Runnable r1 = () -> System.out.println("ラムダ式でのthis: " + System.identityHashCode(this));
        r1.run();

        Runnable r2 = new Runnable() {
            public void run() {
                System.out.println("匿名クラスでのthis: " + System.identityHashCode(this));
            }
        };
        r2.run();
    }
}
```

ラムダ式のthisはラムダ式を作ったインスタンスですので、インスタンスメソッド内のthisと同じです。匿名クラスのthisは、匿名クラスのインスタンス自身です。結果もそうなっていますね。

```
インスタンスメソッドでのthis: 1392838282
ラムダ式でのthis: 1392838282
匿名クラスでのthis: 1325547227
```

ですから、staticメソッド内で書いたラムダ式では、thisを使えません。なぜなら、staticメソッドの中ではthisが使えないからです。でも、匿名クラスのthisは当然使えます。

```
class LambdaTest {
    public static void main(String[] args) {
        // ↓はコンパイルエラー。staticメソッド内ではthisは使えないため
        Runnable r1 = () -> System.out.println("ラムダ式でのthis:" + this);
        r1.run();

        // ↓はthisを使える。thisが匿名クラスのインスタンスを指すため
        Runnable r2 = new Runnable() {
            public void run() {
                System.out.println("匿名クラスでのthis:" + this);
            }
        };
        r2.run();
    }
}
```

#### ３-５-２．変数のスコープが違う

以下のプログラムでは、ラムダ式はコンパイルエラーになり、匿名クラスはコンパイルが通ります。

```
import java.util.function.Consumer;

class LambdaTest {
    public static void main(String[] args) {
        String s = "ABC";

        Consumer<String> c1 = s -> System.out.println(s);

        Consumer<String> c2 = new Consumer<String>() {
            public void accept(String s) {
                System.out.println(s);
            }
        };
    }
}
```

ラムダ式の引数部のスコープは、ラムダ式が書かれているメソッドのスコープに属します。ここでは、ラムダ式の前でsという変数が宣言済なので、コンパイルエラーになります。

一方、匿名クラスのメソッドの引数の変数名は、匿名クラスが書かれているメソッドのスコープには属しません。ですので、同じ変数名でもコンパイルエラーにはなりません。

なお、Javaにはローカル変数の変数名は、フィールドの変数名より優先されるルールがありますので、以下のようにフィールドと引数の変数名が同じなら、引数が優先されます。

```
import java.util.function.Consumer;

class LambdaTest {
    static String s = "ABC";

    public static void main(String[] args) {
        Consumer<String> c = s -> System.out.println(s);
        c.accept("あいうえお"); // → "あいうえお"
    }
}
```

ちなみに、ラムダ式の後ろでなら、ラムダ式の引数に使った変数名を使えます。

```
import java.util.function.Consumer;

class LambdaTest {
    public static void main(String[] args) {
        Consumer<String> c = s -> System.out.println(s);
        String s = "ABC";
    }
}
```

## ４．ラムダ式の使われ方

<a id="14"></a>
### ４-１．Stream APIの引数

ラムダ式の主な用途はStream APIのメソッドの引数です。そもそも、ラムダ式が導入された主な目的は、Stream APIの各メソッドの引数となる「関数」を簡単に短く書くためなのです。

ここで、関数とカッコつけて言っていますが、Stream APIのメソッドの引数となるのは、単なる関数型インターフェイスのインスタンスでしかなく、特別な何かでは決してないのです。

つまり、以下の二つでやっていることは同じですが、実際には前者の書き方を見ることはありません。同じことをはるかに短く書けるのに、わざわざ複雑で長く書きたい人はいないからです。

```
Predicate<String> predicate = new Predicate<String>() {
    public boolean test(String s) {
        return s.startsWith("A");
    }
};
Function<String, String> function = new Function<String, String>() {
    public String apply(String s) {
        return s.toLowerCase();
    }
};
Consumer<String> consumer = new Consumer<String>() {
    public void accept(String s) {
        System.out.println(s);
    }
};
List<String> list = Arrays.asList("AA", "AB", "BC");
list.stream().filter(predicate).map(function).forEach(consumer);
List<String> list = Arrays.asList("AA", "AB", "BC");
list.stream()
    .filter(s -> s.startsWith("A"))
    .map(s -> s.toLowerCase())
    .forEach(s -> System.out.println(s));
```

#### ４-１-１．Stream APIでは、なぜラムダ式が推奨されるのか?

Stream APIでは、ラムダ式を多用したプログラミングスタイルが推奨されます。なぜなら、ラムダ式を使うことで以下の効果が見込めるからです。

- やっていることが見てすぐわかるので、可読性が向上する
- 一つ一つのラムダ式で行う処理は比較的シンプルになるので、処理全体の意図が掴みやすくなる

なお、<font color="red">Javaでのラムダ式の本質は、ずっとお伝えしてきたとおり、関数型インターフェイスのインスタンスを簡単に生成すること、ただそれだけです。</font>

それでもラムダ式が分かりづらいと思われてしまうのは、Stream APIの文脈でStreamの考え方と一緒にラムダ式を説明しようとすることが多いからです。

確かに、ラムダ式の主な導入目的は、Stream APIで用いることです。しかしその目的と、ラムダ式が実現していることの本質とを、混同すべきではないと感じます。

<a id="15"></a>
### ４-２．スレッド関連APIでのRunnable/Callable

Stream API以外でよく使われるラムダ式は、スレッド関連のRunnableとCallableに対するものでしょう。例えば以下のような感じです。

```
Runnable r = () -> System.out.println("Runnable!");
Thread t1 = new Thread(r);
Thread t2 = new Thread(r);
t1.start();
t2.start();
ExecutorService es = Executors.newFixedThreadPool(2);
Future<?> f1 = es.submit(() -> System.out.println("Runnable!"));
Future<String> f2 = es.submit(() -> "Callable!");
f1.get();
System.out.println(f2.get());
es.shutdown();
```

スレッドで行いたい処理が短ければ、このようにやりたいことを1行で書けてしまいます。匿名クラスでもそこそこ短く書けますが、Java 8以降なら、最も短く簡潔に書けるのはラムダ式です。

<a id="16"></a>
### ４-３．関数型インターフェイスの動作カスタマイズ

Java 8では、関数型インターフェイスとして新しいインターフェイスがたくさん追加され、前からあったインターフェイスの多くも、関数型インターフェイスとして明確に指定されました。

それらの関数型インターフェイスには、大変な数のstatic/defaultメソッドを持つものがあります。そのメソッドの多くは、関数型インターフェイスの動作をカスタマイズするためにあります。

そして、そのカスタマイズにも関数型インターフェイスを使うことがほとんどですので、そこでラムダ式の出番があるのです。

#### ４-３-１．Comparatorのラムダ式でのカスタマイズ

ここでは、文字列のソート順のルールが「文字列が長い順、文字列の長さが同じなら辞書順」のjava.util.Comparatorを、ラムダ式で作ってみましょう。

ごく素直に作ると、以下のようになります。もちろんちゃんと動くものですが、このComparatorはこの仕様でのソート順専用で、他への使いまわしができない「一品物」なのは分かりますか?

```
Comparator<String> c = (s1, s2) -> {
    int diff = s2.length() - s1.length();
    return diff != 0 ? diff : s1.compareTo(s2);
};

List<String> list = Arrays.asList("BB", "ABCDE", "AA");
Collections.sort(list, c);

System.out.println(list); // → [ABCDE, AA, BB]
```

これを、Comparatorをカスタマイズするやり方にしてみましょう。最初に文字列長で比較するComparatorを作り、次に辞書順で比較するComparatorを「追加」してみました。

```
Comparator<String> c1 = (s1, s2) -> s2.length() - s1.length();
Comparator<String> c2 = c1.thenComparing((s1, s2) -> s1.compareTo(s2));

List<String> list = Arrays.asList("BB", "ABCDE", "AA");
Collections.sort(list, c2);

System.out.println(list); // → [ABCDE, AA, BB]
```

つまり、2つの動作をするComparatorをつなぎ合わせるカスタマイズを行いました。ラムダ式で追加したのは「辞書順での比較」だけ。文字列長での比較は、既に作ったものを使っています。

このように、カスタマイズしたいことを関数型インターフェイスで表現できるメソッドが、Java 8以降ではものすごく増えました。ラムダ式のこのような使い方を知っておくと大変便利ですよ。

<a id="17"></a>
### ４-４．その他の使われ方

その他のラムダ式の使われ方も、メソッドのデフォルトの動作からどうカスタマイズするか…の「どうする」を表現するのに使う場合が多いです。

例えば、コンピュータの中のとあるディレクトリにあるファイル・サブディレクトリの中から、特定の条件を満たすファイルを一覧化したいとします。

ファイルを探し出す機能のクラス・メソッドは、Javaの標準ライブラリにもうあります。でも「どんなファイルが欲しい」という抽出条件は、使う側が指定しなければなりません。

その抽出条件をラムダ式で表現してみましょう。そのために使う関数型インターフェイスは、java.util.functions.BiPredicateです。抽出条件は名前が“.java”で終わるものとしてみます。

```
Path root = Paths.get("/a/b/c");
Files.find(root, 10, (entry, attr) -> entry.toString().endsWith(".java"))
    .forEach(file -> System.out.println(file.toString()));
```

パス/a/b/cの配下にあるディレクトリ階層を10階層までたどり、その中で名前が“.java”で終わるものを抽出して、それらの名前をprintしています。

そして、抽出条件を変えたければ、BiPredicateをラムダ式で以下のように新しく作って、ファイルを探すメソッドに「これで探してね」と渡してあげればいいだけなのです。

```
BiPredicate<Path, BasicFileAttributes> java = (entry, attr) -> entry.toString().endsWith(".java");
BiPredicate<Path, BasicFileAttributes> clazz = (entry, attr) -> entry.toString().endsWith(".class");

Path root = Paths.get("/a/b/c");
Files.find(root, 10, java).forEach(file -> System.out.println(file.toString()));
Files.find(root, 10, clazz).forEach(file -> System.out.println(file.toString()));
```

こんな感じで、Java 8以降のJavaプログラミングでは、ラムダ式が使える場所がたくさんあります。メソッドの引数が関数型インターフェイスなら「ラムダ式が使えるぞ!!」と喜びましょう。

## ５．【発展】ラムダ式・関数型インターフェイスと部品化の考え方

この章では、関数型インターフェイスを使った「処理の部品化」について考えていきます。関数型インターフェイスとラムダ式の導入目的にも大きく関係している、大事なことです。

<a id="18"></a>
### ５-１．メソッドによる部品化

Javaで処理を共通化する時に、Javaプログラマが一番最初に考えるのはメソッドでしょう。

例えば、同じ文字列を二つつなげる処理が必要になったなら、きっと以下のようなメソッドを作って呼び出しますよね。

```
String duplicate(String s) {
    return s + s;
}
String s1 = duplicate("ABC");
System.out.println(s1); // → "ABCABC"

String s2 = duplicate("あいうえお");
System.out.println(s2); // → "あいうえおあいうえお"
```

ここで何かの理由でメソッドの名前を変えたいなら、プログラム中でそのメソッドを呼び出しているところをすべて直します。そうしないと、プログラムがコンパイルエラーになりますからね。

このように、Javaのメソッドにはとても強い制約・制限があるのです。Javaのメソッドは、いわゆるファーストクラスオブジェクト(第一級オブジェクト)ではないからでもあります。

<a id="19"></a>
### ５-２．関数型インターフェイスによる部品化

関数型インターフェイスとラムダ式を使って、同じことをする部品を作ってみます。使うのは、引数を一つ取り、何かの戻り値を戻すjava.util.function.Functionです。

```
Function<String, String> duplicate = s -> s + s;

String s1 = duplicate.apply("ABC");
System.out.println(s1); // → "ABCABC"

String s2 = duplicate.apply("あいうえお");
System.out.println(s2); // → "あいうえおあいうえお"
```

このように文字列をつなげるFunctionができました。確かにメソッドと同じ動きをしますが、メソッドと大して変わらない印象です。でも、実は便利なものだということを、次に見ていきます。

<a id="20"></a>
### ５-３．関数型インターフェイスなら処理を差し替えできる

以下のような、引数のStringに対して、Function<String, String>で処理を行った結果を戻すメソッドがあるとします。

```
String applyFunctions(String s, Function<String, String>... functions) {
    String ret = s;

    for (Function<String, String> function : functions) {
        ret = function.apply(ret);
    }

    return ret;
}
```

このapplyFunctionsは、以下のように使えます。applyFunctionsにどんな「処理」を引数で与えるかで、結果が変わります。ここでapplyFunctions自体は何も変えてはいないことに注目です。

```
Function<String, String> firstChar = s -> s.substring(0, 1);
Function<String, String> duplicate = s -> s + s;

String s1 = applyFunctions("ABC", firstChar);
System.out.println(s1); // → "A"

String s2 = applyFunctions("ABC", duplicate);
System.out.println(s2); // → "ABCABC"

String s3 = applyFunctions("ABC", firstChar, duplicate);
System.out.println(s3); // → "AA"
```

applyFunctionsへどんなFunctionを与えるかで、メソッドの動きを呼び出す側が自由に変えられます。つまり、メソッドやクラスの外部から「何をするか(=Function)」を差し替えられるのです。

そして、Functionのインスタンスごとにやることが違います。でも、Functionという型とメソッドの呼び出し方はすべて同じ。同じ型でさえあれば、applyFunctionsで同じように使えるのです。

ここで、プログラムの「抽象度」が上がったのが分かりますか? applyFunctionsの役割は部品のFunctionを決められた手順で実行するだけ、部品はメソッドの外から与えられているからです。

<a id="21"></a>
### ５-４．Javaでの関数は関数型インターフェイスのインスタンス

ラムダ式の文脈では「関数(function)」という言葉が頻繁に使われます。「関数」型インターフェイスであったり、Stream APIは集合に「関数」を適用した結果を得る…とも表現されますね。

この文脈での関数は、数学での「何かを何かに対応づけるもの(射影)」の意味で使われます。言い換えれば、何かを入力にして何かを戻すモノであり、これこそが数学の高い抽象度の源泉です。

関数型インターフェイスと、そのインスタンスを簡単に作るためのラムダ式は、前述の意味での「関数」を活用したプログラミングスタイルを、Javaへ積極的に導入しようというものです。

つまり、何か1つのことができる関数型インターフェイスのインスタンスを、処理が具現化した部品として扱おう、という考え方です。その部品を組み合わせて、複雑な処理を作るのです。

<a id="22"></a>
### ５-５．ラムダ式が関数を作るハードルを下げる

ここまでお伝えしてきたことは、Javaにとって本当の意味で新しいプログラミングスタイルではありません。ラムダ式でなくても、今までも使えた道具…つまりクラスやインターフェイスなどでも「できる」からです。

しかし、今までは「関数」的な考え方でプログラムを作ることへは、Javaプログラマの心理的ハードルやスキル差が立ちはだかっていました。関数的な考え方でプログラムを作れる「可能性」と、その作り方がプログラマたちへ浸透していることとは別なのです。

例えば、多くの行を費やしてクラスを定義したり、インターフェイスを実装したクラスを定義するのは面倒なことでした。プログラムの行数が増えると、それだけで読みづらくもなりますしね。それに、匿名クラスの使い方や考え方を知らないJavaプログラマは、意外に多いのです。

でも、ラムダ式ならやりたい処理だけを手軽に短く書けるので、プログラマが使おうという気になります。ラムダ式こそが、Javaのプログラミングスタイルを大きく変えていく起爆剤なのです。

<a id="23"></a>
## ６．まとめ

<font color="red">Javaのラムダ式は、関数型インターフェイスを実装したクラスのインスタンスを簡単に作るための文法です。</font>これさえ忘れなければ、ラムダ式に惑わされることは少なくなるでしょう。

そして、抽象メソッドが1つだけある関数型インターフェイス、これこそがラムダ式の舞台裏を支える大立役者です。これも絶対確実にその仕組み・意味・意義を覚えておきましょう。

Javaはラムダ式を見つけると、プログラムのどこかから関数型インターフェイスの型を見つけ、1つだけある抽象メソッドを実装したクラスを自動で作り、そのインスタンスを生成します。

ラムダ式の活用方法は、関数型インターフェイスのインスタンスをお手軽に作成して、プログラム中で部品、すなわち関数として扱うことです。関数を全面的に活用する例がStream APIであり、その実用面を支えるのがラムダ式です。

この記事では、ラムダ式と同時に追加され、一心同体でもあるメソッド参照(method reference)には触れませんでした。別の記事でお伝えすることもあるかもしれませんが、ラムダ式が分かればメソッド参照の理解まではもうすぐですよ。