出所：[【第一回】Stream APIを使う前にジェネリクスの基礎を理解する #java8 - Qiita](https://qiita.com/tech_newbie/items/6adcb7dc5f59165f3fb9)

取得年月日：2023(R5).11.13

[@tech_newbie(tech newbie)](https://qiita.com/tech_newbie)

# 【第一回】Stream APIを使う前にジェネリクスの基礎を理解する

- [java8](https://qiita.com/tags/java8)
- [Java入門](https://qiita.com/tags/java入門)
- [StreamAPI](https://qiita.com/tags/streamapi)
- [ジェネリクス](https://qiita.com/tags/ジェネリクス)

最終更新日 2019年11月30日投稿日 2019年11月17日

# はじめに

この投稿は、Java8におけるStream APIを活用する為の事前知識として書いています。
その為、深いところまでは突っ込まず基礎的な内容に留めていきます。

Streamを理解するためには複数の知識が必要なので、下記流れで投稿していきます！

- 第一回　[ジェネリクス](https://qiita.com/tech_newbie/items/6adcb7dc5f59165f3fb9)
- 第二回　[関数型インターフェース](https://qiita.com/tech_newbie/items/083cf1ffcfa9744b0c88)
- 第三回　[ラムダ式とメソッド参照](https://qiita.com/tech_newbie/items/2588970f3d2ce08a1b16)
- 第四回　[Stream API](https://qiita.com/tech_newbie/items/3ee76c639f6a1a311174)

まだまだエンジニアとして初心者なので、間違いがあったらご指摘下さい！

# 目的

- ジェネリクスを活用できるようになる
- ジェネリクスによる型安全を理解する
- 仮型パラメータを活用できるようになる

# ジェネリクスとは

こんなコードは参考書とかでもよく目にしますよね。

```
List<String> sampleList = new ArrayList<String>();
```

この`<String>`の部分が**ジェネリクス**です。

`<String>`と記述することで、このListの中には**String型の要素だけが入るべき**ということを宣言しています。
よって、このListの中に`Integer`といった他の型の要素を入れることはできません。
この**String型の要素が入っている**ということが保証されることで型安全を図ることができます。

# ジェネリクスによる型安全

もしジェネリクスが指定されていないListがあったとしましょう。

原型のList

```
List sampleList = new ArrayList();
```

このコードはコンパイルエラーにはなりません。
ジェネリクスが指定されていないので、このListにはどんなオブジェクトでも入れることができます。
こんな形は**原型(raw type)**とも呼ばれていたりします。

ただ、単純に考えてどんなオブジェクトでも入れられてしまうListって怖いですよね？
例えばこんなListがあったとしましょう。

ClassCastException例

```
//原型のList
List sampleList = new ArrayList();

sampleList.add(new Person(/*省略*/)); // -> 0番目の要素として追加
sampleList.add(new Dog(/*省略*/)); // -> 1番目の要素として追加

//値を取り出してDog型にcast
Dog Pochi = (Dog)sampleList.get(0); // -> ClassCastExceptionの発生
```

このように、色々な要素がごっちゃになって入っていたら、
ClassCastExceptionを発生させない為にも値を取り出す際は注意深くcastしなくてはいけません。

ジェネリクスを使ってた場合はどうでしょうか

```
//ジェネリクスを用いたList
List<Dog> sampleList = new ArrayList<>();
```

このListにはDog型のオブジェクトしか入っていないことを宣言しています。
これならListの要素を取り出す時でもPerson型だっけ？Dog型だっけ？と考えずに済みますね。
コンパイル時に型検査を行ってくれるので、ClassCastExceptionが発生する可能性も大幅に減ります。

# 仮型パラメータ

ジェネリクスは型安全を保証すること以外にも活用方法があります。
それが仮型パラメータを使用することです。

説明する前に、まずは仮型パラメータを使っていない場合のコードを見て下さい。

StrData.java

```
public class StrData {
    private String data;

    public String getData() {
        return data;
    }
    public void setData(String data) {
        this.data = data;
    }
}
```

このクラスにはgetterとsetterしかありません。

しかし、これでは不便な場合が出てきます。
というのも、このクラスではStringしか扱うことができません。
型は違うけど同じ処理を行うクラスを作りたいという場合、いちいち別で定義しなきゃいけなくなります。

これを仮型パラメータを使って改善してみましょう！

Data.java

```
public class Data<T> {
    private T data;

    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```

ここではクラス名の後に`<T>`という記述を用いて宣言されています。
この`T`が**仮型パラメータ**です。

この形で宣言することで、このクラスでは`"T型の要素"`を扱うことが可能になります。
`<T>`を使用することで、このクラスではString型でもInteger型でも他の型でも扱うことができます。

というのもこの時点では`<T>`の型が何になるかは決まっていないからです。
実際にどの型にするかはこのクラスを利用する際に決定します。

イメージが付きにくいと思うのでコードで見てみましょう！

GenSample.java

```
public static void main(String[] args) {
    Data<String> d1 = new Data<>();
    Data<Integer> d2 = new Data<>();

    //要素をセット
    d1.setData("サンプル");
    d2.setData(100);

    //要素を取得し代入
    String str = d1.getData(); // <String>を指定したのでコンパイルエラーが起きない
    Integer num = d2.getData(); // <Integer>を指定したのでコンパイルエラーが起きない

    //要素の確認
    System.out.println(str + "テキスト"); // -> サンプルテキスト ※文字列として繋がっている
    System.out.println(num + 1); // -> 101 ※数値として計算されている
}
```

ここではインスタンスを生成する際に、ジェネリクスの型を指定することで仮型パラメータ`T`の型を決めています。
指定した型で読み替えられ、処理が行われているわけですね。

# なんでTなの？

実はこの仮型パラメータの`T`というアルファベット自体には特に意味は持ちません。
というのも`A`でも`B`でもなんでもいいことになっています。
しかし、実際に使われる際は作法というか一般的なパターンが存在しています！

| 文字の種類 | 示す内容       |
| :--------- | :------------- |
| E          | Element (要素) |
| T          | Type (タイプ)  |
| K          | Key (キー)     |
| V          | Value (値)     |
| N          | Number (数字)  |

個人的に、メソッドで使われる場合は次のケースが多い気がします。

- `T` = 1つ目の引数
- `U` = 2つ目の引数
- `R` = 戻り値

# 定義方法のまとめ

### クラスの場合

`Class<T>`の形で宣言すると、その仮型パラメータ`T`をクラスのフィールドやメソッドで使用することができます。

```
public class SampleClass<T>{
    private T field;
}
```

### メソッドの場合

戻り値の前に`<T>`を記述することで、そのその仮型パラメータ`T`をメソッド内で使用することができます。

```
public <T> T getT(T t) {
    return t;
}
```

# ジェネリクスの継承関係

**Javaにおけるジェネリクスの性質は不変とされています。**

これがどういった継承関係の意味を指しているかを説明していきます。

```
Integer intNum = 1;
Number num = intNum;
```

このコードは問題なく実行することができます。
というのも`Number`は`Integer`のスーパークラスであるからです。
公式のリファレンスを見てもそう定義されていますよね。

公式のAPIリファレンスより引用

> public final class Integer extends Number

**狭い型（Integer）から広い型（Number）へ変換することができているので、この場合の性質は"共変"**となります。

次のコードはどうでしょうか。

```
List<Number> numList = new ArrayList<Integer>();
```

実行すると下記エラーが発生します。

> 型の不一致: ArrayList<Integer> から List<Number> には変換できません

ここではジェネリクスを用いてますが、最初に述べたようにJavaにおいて**ジェネリクスの性質は不変**です。
**不変とは狭い型（Integer）から広い型（Number）へ変換ができず、互換性がないこと**を意味しています。
なので、`List<Integer>`と`List<Number>`の間にはなにも関係を持ちません。

ジェネリクスを使用する目的として、型安全を図ることがあるので性質は不変となっています。

# 型パラメータにワイルドカードを指定する

とはいえ、ジェネリクスが一切継承関係を表現できないという訳ではありません。
表現するには`?`**(ワイルドカード)**を使用します。

先程のリストを改良したコードを見てみましょう

```
List<?> numList = new ArrayList<Integer>();
```

これで、コンパイルエラーは起きなくなります。
この<?>だけを使用したのを**非境界型ワイルドカード**といいます。

具体的な利用方法ですが、例えば
任意の型のListを受け取って、その要素全てを出力するメソッドを見てみます。

```
public static void main(String[] args) {
    List<Integer> numList = new ArrayList<>();
    List<String> strList = new ArrayList<>();

    numList = Arrays.asList(1,2,3);
    strList = Arrays.asList("サンプル","文字列");
    printAll(numList);
    printAll(strList);
}
public static void printAll(List<?> list) {
    for (Object o : list) {
        System.out.println(o);
    }
}
//実行結果 -> 1,2,3,サンプル,文字列  ※改行を,に置き換えて記述
```

結果としてintListとstrListの両方の要素が出力されます。
両者は型は違いますが、受け取り側のメソッドでワイルドカードを指定している為、
どんな型のListでも受け取ることができています。

# 非境界型ワイルドカードの制約

原型と似ていますが、非境界型ワイルドカードにはジェネリクスとしての型安全を保障するために、
以下の制約があります。

- 要素を追加できない
  - 要素を追加(add)しようとしても型パラメータが決まらないためコンパイルエラーになる

```
//要素の追加は不可
public static void addElement(List<?> list) {
    list.add("要素"); // -> コンパイルエラー
}
```

- 取得した要素はObject型
  - 型パラメータは不明であっても、すべての参照型のスーパータイプであるObject型ならClassCastExceptionは発生しないため

```
List<Integer> numList = Arrays.asList(1,2,3);
List<?> list = numList;

//取得した要素はObject型
Integer num = list.get(0); // -> コンパイルエラー　※<?>が<Integer>だという確証がないため
Object obj = list.get(0); // -> OK
```

# 境界型ワイルドカード

どんな型でも代入できた非境界型ワイルドカードに継承関係の制限をつけてみましょう。
制限を付けるには**境界型ワイルドカード**を使用します。

| 記述例            | 呼び方                   | 拡張範囲                                            |
| :---------------- | :----------------------- | :-------------------------------------------------- |
| <? **extends** T> | 上限境界ワイルドカード型 | 自分自身(T)もしくはその**サブクラスの型に拡張**     |
| <? **super** T>   | 下限境界ワイルドカード型 | 自分自身(T)もしくはその**スーパークラスの型に拡張** |

実際に上限境界ワイルドカード型を用いてさっきのコードを改良してみます

```
//代入不可 --上記でコンパイルエラーが出た例
List<Number> numList1 = new ArrayList<Integer>();

//代入可能 --上限境界ワイルドカード型を用いた例
List<? extends Number> numList2 = new ArrayList<Integer>(); 
```

1つ目：ジェネリクスを用いた<Number>と<Integer>は何も関係がない（不変の）為、代入不可です。
2つ目：NumberはIntegerのスーパークラスなので代入可能です。

superも含めた境界型ワイルドカードをまとめると以下になります。

```
//extends
List<? extends Number> extends1 = new ArrayList<Object>(); // -> コンパイルエラー
List<? extends Number> extends2 = new ArrayList<Integer>(); // -> OK

//super
List<? super Number> super1 = new ArrayList<Object>(); // -> OK
List<? super Number> super2 = new ArrayList<Integer>(); // -> コンパイルエラー
```

- extends1
  - ObjectはNumberのサブクラスでないのでコンパイルエラー（代入不可）
- super2
  - IntegerはNumberのスーパークラスでないのでコンパイルエラー（代入不可）

ここでは詳しく書きませんが、extends, superのどちらを使うかということで、
PECS, Put/Get原則といった基準となる考え方があったりします。

# おわりに

ここまでを覚えることで**`Function<T,R>`**といった関数型インターフェースを使いこなせるのではないでしょうか。

Effective Javaとかではもっと有効に活用するためのヒントが書いてあるので、自分自身も勉強していければと思います！