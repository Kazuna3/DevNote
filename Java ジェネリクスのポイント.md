出所：[Java ジェネリクスのポイント #Java - Qiita](https://qiita.com/pebblip/items/1206f866980f2ff91e77)

最終更新日：2015(H27).07.22

取得年月日：2023(R5).10.27

# Java ジェネリクスのポイント

久しぶりにジェネリクスを多用するライブラリを作成していて、忘れていたことやはまりどころがあったので、今更ですが備忘録的にポイントをまとめておきます。

# 用語

ジェネリクスは似たような用語があって混乱しやすいので、まずは用語をきちんと抑えておく。
以下は、[Effective Java](http://www.amazon.co.jp/Effective-Java-プログラミング言語ガイド-Joshua-Bloch/dp/4894714361)の項目23からの抜粋である。

| 用語                                              | 例                     |
| :------------------------------------------------ | :--------------------- |
| ジェネリック型（generic type）                    | List<E>                |
| 仮型パラメータ（type parameter）                  | E                      |
| パラメータ化された型（parameterized type）        | List<String>           |
| 実型パラメータ (actual type parameter)            | String                 |
| 原型（raw type）                                  | List                   |
| 境界型パラメータ（bounded type parameter）        | <E extends Number>     |
| 非境界ワイルドカード型（unbounded wildcard type） | List<?>                |
| 境界ワイルドカード型（bounded wildcard type）     | List<? extends Number> |



# 変性

ジェネリクス型 `X<T>` において、型Aが型Bのサブタイプであるとき、`X<A>` が `X<B>`　のサブタイプであれば、Xは型パラメータTについて **共変（covariant）** という。逆に、`X<B>` が `X<A>` のサブタイプとなるなら、**反変（contravariant）** という。いずれも成り立たない場合、これを**不変（invariant）**という。ジェネリクス型はこれらいずれかの **変性** を持つ。以下に例を示す。

| 変性 | 例                                                   |
| :--- | :--------------------------------------------------- |
| 共変 | `List<Integer>` が `List<Number>` のサブタイプとなる |
| 反変 | `List<Number>` が `List<Integer>`のサブタイプとなる  |
| 非変 | `List<Number>`と`List<Integer>`の継承関係はない      |



# Javaのジェネリクスは非変である

Javaのジェネリクス型は非変である。このため、IntegerはNumberのサブタイプだが、`List<Integer>` は`List<Number>` のサブタイプではない。

以下のサンプルコードに例を示す。

```
    public void hoge() {
        List<Integer> intList = new ArrayList<>();
        List<Number> numList = new ArrayList<>();
        List<Number> anotherNumList = new ArrayList<>();

        numList = anotherNumList; //OK
        numList = intList; //コンパイルエラー       
    }
```

## 配列は共変

対照的に、Javaの配列は共変である。したがって、以下のような危険なコードが書ける。

```
    public static void foo() {
        Object[] objects;
        Integer[] integers = new Integer[]{1,2,3};
        objects = integers;
        objects[0] = "error"; //実行時例外 ArrayStoreExceptioin
    }
```

# コンパイル時に型情報は消去される

パラメータ化された型や、型パラメータの持つ型情報はコンパイラによって消去される。これは型消去（type erasure)と呼ばれる。

例えば、以下の型変数Tを持つジェネリック型を考える。
ContainerはIntegerやBigDecimalなどのNumber型のみの値を保持するコンテナクラス。

```
class Container<T extends Number> {
    private T value;    
    public Container(T value) {
        this.value = value;
    }   
    public T get() {
        return value;
    }
}
```

コンパイラは、上記のクラスから型情報を消去し、以下のクラスと同等のクラスを生成する。

```
class Container {
    private Number value;
    public Container(Number value) {
        this.value = value;
    }
    public Number get() {
        return value;
    }
}
```

型情報といっても実際に消去するわけではなく、型パラメータはその上限境界の型（上の例ではNumber）に置換される。
このように、型情報はコンパイラによって消去されるため、**実行時には型情報を取得することはできない**。

# `new T()` できない

型パラメータは消去されるため、型Tのインスタンスを生成することはできない。

```
class Illegal<T> {
    public T create() {
        return new T();
    }
}
```

どうしてもやりたい場合は、以下のようにするしかない。

```
class Legal<T> {
    public T create(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }
}
```

# `new T[]` できない

ジェネリクス型の場合は、以下のように仮型パラメータTを型とするインスタンスを生成でる。

```
class Sample<T> {
    public List<T> createList(int size) {
        //TのArrayListを生成できる
        return new ArrayList<T>(size);
    }
}
```

一方、型パラメータTを要素とする配列は生成できない。

```
class Sample<T> {   
    public T[] createArray(int size) {
        return new T[size];
    }   
}
```

すでに説明したように、型パラメータTの型情報は実行時に消去される。ジェネリクス型の場合は、型情報を含まない原型を生成するため、実行時に型情報を必要としない。一方、配列の生成には実行時に型情報が必要となるため（正確には配列を生成するバイトコードが型を必要とする）、型パラメータTを要素とする配列は生成できない。

# `List<?>` は任意の要素型をもつListを表す

`List<?>` は、なんらかの型の要素を持つListを表す。このような型を非境界ワイルドカード型という。`List<?>`の変数には任意のListを代入できる。したがって、以下のようなコードが記述できる。

```
    public void wildcardHasAnyType() {
        List<String> stringList = new ArrayList<>();
        List<Integer> integerList = new ArrayList<>();

        boolean b1 = contains(stringList, "a"); //List<String>
        boolean b2 = contains(integerList, 3); //List<Integer>
    }

    public boolean contains(List<?> list, Object o) {
        return list.contains(o);
    }
```

これは、共変の性質により`List<Object>` の変数には `List<String>` を代入できなかったことと対照的である。このように、ワイルドカード型を利用すると、ジェネリックの不変の制約を緩めることができる。

# `List<?>` に要素をaddできない

`List<?>` は、任意の要素を持つListであるが、具体的な要素型については不定である。したがて、null値を除くいかなる値も渡すことはできない。したがって、以下のようなコードはエラーとなる。

```
    public void foo(List<?> anyList) {
        anyList.add("error"); //コンパイルエラー
    }
```

より、一般的に言うなら、型パラメータが引数に現れる非境界ワイルドカード型のメソッドを呼び出すことはできない。

```
interface UnaryFunction<T> {
    public T apply(T value);
}

class Illegal {
    public void foo(UnaryFunction<?> f) {
        f.apply("a"); //コンパイルエラー
    }
}
```

# `List<?>` からgetした要素はObjectである

非境界ワイルドカード型の要素はどのような型にでもなりえるため、取得した要素型はもっとも汎用的な型であるObjectとなる。

```
    public void bar(List<?> list) {
        Object o = list.get(0);
        String s = list.get(0); //コンパイルエラー
    }
```

# `List<? extends T>` は共変性を持つ

`List<? extends T>` は **上限付きの境界ワイルドカード型** である。これは、型Tのなんらかのサブタイプを要素とするListを表す。上限付きワイルドカード型を使うと、以下のサンプルコードのように、本来非変であるジェネリクスに共変性を導入することができる。

```
    public void foo() {
        List<Number> numList = new ArrayList<>();
        List<? extends Number> wildCardNumList = new ArrayList<>();
        List<Integer> intList = new ArrayList<>();

        wildCardNumList = intList; //OK
        numList = intList; //コンパイルエラー
    }
```

# `List<? super T>` は反変性を持つ

`List<? super T>` は **下限付きの境界ワイルドカード型** である。これは、型Tのなんらかのスーパータイプを要素とするListを表す。下限付きワイルドカード型を使うと、以下のサンプルコードのように、本来非変であるジェネリクスに反変性を導入することができる。

```
    public void bar() {
        List<Integer> intList = new ArrayList<>();
        List<? super Integer> wildCardIntList = new ArrayList<>();
        List<Number> numList = new ArrayList<>();

        wildCardIntList = numList; //OK
        intList = numList; //コンパイルエラー
    }
```

# Put/Get原則

[Effective Java](http://www.amazon.co.jp/Effective-Java-プログラミング言語ガイド-Joshua-Bloch/dp/4894714361)の項目28では、以下の略語からPECSとも呼ばれる。

> PECSは、プロデューサー（producer）-extends, コンシューマー（consumer）-superを表しています。

Put/Get原則あるいはPECSは、以下の２つからなるワイルドカード型の基本原則である。

- 型変数Tでパラメータ化された型が、プロデューサーであれば、`<? extends T>` を利用する。
- 型変数Tでパラメータ化された型が、コンシューマであれば、`<? super T>` を利用する。

Put/Get原則は柔軟性のあるAPIを実現するためのキーとなる。以下で、サンプルコードを通してPut/Get原則の有用性について示す。

まずは、Put/Get原則を適用していない、以下のようなインターフェースを考える。コードの一部は、[ここ](http://www.ibm.com/developerworks/jp/java/library/j-jtp07018.html)から引用している。

```
//型Tの値を消費する関数
interface Consumer<T> {
    void apply(T value);
}

//型Tの値をもつコンテナ
interface Box<T> {
    //保持する値を返す
    T get();
    //値を設定する
    void put(T element);
    //別のコンテナの値を設定する
    void put(Box<T> box);
    //Consumer関数を適用する
    void applyConsumer(Consumer<T> function);
}
```

ここで、以下のコードはコンパイルエラーとなる。

```
class InvariantSample {
    public void foo(Box<Number> numBox, Box<Integer> intBox) {
         numBox.put(1); //OK
         numBox.put(intBox); //コンパイルエラー      
    }
}
```

`Box<Number>` のputメソッドは `Box<Number>` を引数として受けとるが、非変性により、`Box<Integer>` は `Box<Number>`のサブタイプではないため、`Box<Integer>`の引数を適用するとエラーとなる。

`put(1)` により、Integer型の要素を設定できるにもかかわらず、`Box<Integer>` を設定できないのは、APIとして柔軟性に欠けている。

次に、以下のコードを考える。

```
    public void foo(Box<Integer> intBox, Consumer<Integer> intConsumer, Consumer<Number> numConsumer) {
        intBox.applyConsumer(intConsumer); //OK
        intBox.applyConsumer(numConsumer); //コンパイルエラー
    }
```

Integer型の要素を消費する `Consumer<Integer>` 型の関数を適用できるにもかかわらず、Number型の要素を消費する`Consumer<Integer>` の関数を適用できないのは直感に反している。

上記のインターフェースについて、Put/Get原則を適用してみる。適用できる箇所は以下の２つのメソッドである。

- `void put(Box<T> box)` の引数 `Box<T>` は値のプロデューサーである。
- `void applyConsumer(Consumer<T> function)` の引数 `Consumer<T>` は値のコンシューマーである

よって、これら２つのメソッドにPut/Get原則を適用すると以下のようになる。

```
//型Tの値をもつコンテナ
interface Box<T> {
    //保持する値を返す
    T get();
    //値を設定する
    void put(T element);
    //別のコンテナの値を設定する
    void put(Box<? extends T> box);
    //Consumer関数を適用する
    void applyConsumer(Consumer<? super T> function);
}
```

この結果、Put/Get原則の適用前のコンパイルエラーは全て解消される。