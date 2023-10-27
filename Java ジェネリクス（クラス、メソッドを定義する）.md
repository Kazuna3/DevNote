出所：[Java ジェネリクス（クラス、メソッドを定義する） #Java - Qiita](https://qiita.com/rodentia6/items/b36d134fa24867ba4d63)

最終更新日：2021(R3).06.04

取得年月日：2023(R5).10.27

# Java ジェネリクス（クラス、メソッドを定義する）

Javaのジェネリッククラスやジェネリックメソッドを定義する上での文法について。
用語に関してはここを参照 → [Java ジェネリクスのポイント](http://qiita.com/pebblip/items/1206f866980f2ff91e77)

# 仮型パラメータ

例えば`List<E>`であれば、山カッコで括られた`E`が仮型パラメータである。
文法上は2文字以上や小文字混じり等でも問題ないが、慣例的には1文字が多く使われる。
`<E>`や`<T>`はそれぞれ*Element*や*Type*の略だったりする。

## 仮型パラメータに対する制約

型パラメータの情報は実行時に消去されるので、いくつかの制約がある。

- `new T()`できない
- `new T[]`できない
- `T.class`は参照できない
- `obj instanceof T`の評価はできない

# ジェネリッククラスの定義方法

　`SomeClass<T>`と書くと、その型パラメータ`T`をクラスのフィールドや（引数、戻り値を含めた）メソッド内で使用することができる。

```
//                     ↓ここに型パラメータを書く
public class SomeClass<T> {
  private T t;  // フィールド署名にTを利用できる
  private List<T> list = new ArrayList<T>();  // ジェネリクスを使用したフィールドにTが使用できる
  public void set(T arg){  // インスタンスメソッドの署名にTが使用できる
    this.t = arg;
  }
  public T get(){
    return this.t;
  }
}
```

なお、以下のようにジェネリッククラスの仮型パラメータをそのクラス内のstaticメソッドやstatic文の中で使用することはできない。
staticな文脈で利用したい場合は後述のジェネリックメソッドで解決しよう。

```
public class Hoge<T>{
  static {
    T t; // コンパイルエラー
  }
  public static void hoge(T t){ // コンパイルエラー
  }
}
```

# ジェネリックメソッドの定義方法

メソッドの戻り値の型の手前に`<T>`を書くと、その型パラメータ`T`をメソッド内で使用することができる。

```
  //             ↓ここに型パラメータを書く
  public static <T> ArrayList<T> someMethod(T arg){
    ArrayList<T> list = new ArrayList<T>();  // メソッド内でTが使用できる
    list.add(arg);
    return list;
  }
```

------

ジェネリッククラスの定義やメソッド定義の基本的なところは以上で、あとはケースによって仮型パラメータである`<T>`の中身を変えていく。

------

# いろいろな例

## 複数の型パラメータ

カンマ区切りで複数の型パラメータを使用することができる。

```
  public static <K, V> HashMap<K, V> newHashMap(){
    return new HashMap<K, V>();
  }
  // ※<K,V>の型は左辺から推測してくれる
  Map<String, Integer> map = newHashMap();
```

## 境界型パラメータ

`<T extends クラスやインターフェース>`のように書くと、`T`が指定したクラスのサブクラス（または指定したクラスそのもの）であることを呼び出し側に要求することができる。

```
  // Number型のサブクラスもしくはNumber型でのみインスタンス化できるArrayList
  public class NumArrayList<T extends Number> extends ArrayList<T> {}
  List<Number> numberList = new NumArrayList<Number>(); // ok
  List<Double> doubleList = new NumArrayList<Double>(); // ok
  List<String> stringList = new NumArrayList<String>(); // コンパイルエラー
```

## 再帰型境界

この例ではEがComparableの実装クラスであることを呼び出し側に要求している。

```
  // コレクション内から最大値を取り出すメソッド
  public static <E extends Comparable<E>> E max(Collection<E> collection) {
    // コード省略。EはcompareTo(E)メソッドを持つので、
    // それを利用して要素の最大値を求めるアルゴリズムを実装することができる
  }
```

## 複数の継承指定

複数のインターフェースの実装クラスであることを指定するには、親クラスやインターフェース同士を`&`で繋ぐ。

```
// ComparableインターフェースとSerializableインターフェースの両方の実装をEに要求している
public class Fuga<E extends Comparable<E> & Serializable>{}
```

（`&`はあまり見慣れない記号だが、型パラメータの区切り文字として`,`が既に使われているために`&`を使用しているのだと思われる）