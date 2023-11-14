出所：[【第四回】Stream APIの基礎的な使い方 #java8 - Qiita](https://qiita.com/tech_newbie/items/3ee76c639f6a1a311174)

取得年月日：2023(R5).11.13

[@tech_newbie(tech newbie)](https://qiita.com/tech_newbie)

# 【第四回】Stream APIの基礎的な使い方

- [java8](https://qiita.com/tags/java8)
- [Java入門](https://qiita.com/tags/java入門)
- [StreamAPI](https://qiita.com/tags/streamapi)

投稿日 2019年11月30日

# はじめに

この投稿は、Java8におけるStream APIを扱うための基礎知識を書いています。
その為、性能的な側面や複雑な処理については扱いません。

また、事前に以前の投稿である、下記3項目を理解しているものとして進めていきます。

- 第一回　[ジェネリクス](https://qiita.com/tech_newbie/items/6adcb7dc5f59165f3fb9)
- 第二回　[関数型インターフェース](https://qiita.com/tech_newbie/items/083cf1ffcfa9744b0c88)
- 第三回　[ラムダ式とメソッド参照](https://qiita.com/tech_newbie/items/2588970f3d2ce08a1b16)

途中で詰まった場合は、ぜひ以前の投稿を参考にしてみて下さい！

# Stream APIってなに？

**`Stream API`**は、公式から引用すると次のように説明されています

> コレクションに対するマップ-リデュース変換など、
> 要素のストリームに対する関数型の操作をサポートするクラスです。

要は`List`や`Map`といったコレクションで何らかの処理を行う際に`Stream API`を用いると、

- **簡潔に記述できたり**
- **可読性が向上したり**
- **性能が向上したり**

といったメリットがあります。

調べると**「for文の代わりに使おう！」「for文禁止」**と書かれている記事もよく見受けられます。
私自身もStream APIを知ってから**ほぼfor文を書かなくなりました。**それぐらい便利です！

# 従来までのfor文によるコーディング

ForSample.java

```
public static void main(String[] args) {
    List<String> animals = Arrays.asList("ape", "bear", "cat", "cow", "camel", "dog");
    List<String> startsWithC = new ArrayList<>();

    for (int i = 0; i < animals.size(); i++) {
        String str = animals.get(i);
        if (str.startsWith("c")) {
            startsWithC.add(str.toUpperCase());
        }
    }
    for (int i = 0; i < startsWithC.size(); i++) {
        System.out.println(startsWithC.get(i)); // CAT,COW,CAMEL
    }
}
```

一番原始的と思われる方法で書いてみました。

このコードで行いたい処理としては
**1. animalsというリストから頭文字が'c'で始まる動物の文字列を抽出する**
**2. その文字列を大文字に変換する**
**3. 出力**

というものです。
あえて拡張for文を使わずに記述していますが、冗長でどんな処理を行っているかが一目で把握し辛いですよね。
このレベルでの処理だけならいいですが、仮にこの処理に2つ追加したとします。

**・文字列を自然順序でソートして**
**・上位2件に来る文字列を抽出する**

余計ゴチャゴチャになるのが想像できるのではないでしょうか？

**こういった可読性が低く冗長な記述をStream APIは解決してくれます。**

# Stream APIによるコーディング

実際にStream APIを使用していきましょう。
処理内容は最初に書いたものに、2つの処理を追加したので次の通りです。

**1. animalsというリストから頭文字が'c'で始まる動物の文字列を抽出する**
**2. その文字列を大文字に変換する**
**3. 文字列を自然順序でソートして**
**4. 上位2件に来る文字列を抽出する**
**5. 出力**

コードを見てみましょう。

StreamSample.java

```
public static void main(String[] args) {
    List<String> animals = Arrays.asList("ape", "bear", "cat", "cow", "camel", "dog");

    animals.stream()                              //Streamを取得
            .filter(s -> s.startsWith("c"))       //'c'から始まる動物の文字列を抽出
            .map(s -> s.toUpperCase())            //大文字に変換
            .sorted()                             //自然順序でソート
            .limit(2)                             //上位2件を抽出
            .forEach(System.out::println);        //出力 -> CAMEL, CAT
}
```

`filter`,`map`,`sorted`,`limit`,`forEach`といった中間/終端操作については後に記述しますが、
`map`以外はメソッド名を見るだけでもなんとなく処理内容がイメージつくのではないでしょうか！

先程追加した処理内容を流暢に表現できますし、
**「ここは i < と i <= どっちだったっけ...」**といったfor文内における余計なコーディングミスも防止できます。

こういった点もStream APIを利用する大きなメリットかと思います。

# Streamの記述方法

Streamを扱う場合は、3段階に別れます。

```
{Stream生成} → {中間操作} → {終端操作}
```

※中間操作は複数持てるのに対し、終端操作は1つしか持てません。

# Streamの生成

Streamを生成することで、先程のような中間操作や終端操作を伴う処理が可能となります。
Streamの生成方法はいくつかありますが、代表的な例を紹介します。

## Stream.ofからのStream生成

**`Stream.of()`**内に要素を列挙し、Streamを生成します。

```
Stream<String> strStream = Stream.of("cat","dog","cow");
strStream.filter(s -> s.startsWith("c")).forEach(System.out::println);
```

## 配列からのStream生成

**`Arrays.stream`**メソッドを使用して、配列からStreamを生成します。

```
String[] arrays = {"cat","dog","cow"};
Stream<String> arraysStream = Arrays.stream(arrays);
arraysStream.filter(s -> s.startsWith("c")).forEach(System.out::println);
```

## CollectionからのStream生成

Java8よりCollectionインターフェースに**`stream`**メソッドが追加されました。
これを利用することで、`List`や`Map`といった既存のコレクションからStreamを生成することができます。

```
List<String> animals = Arrays.asList("cat", "dog", "cow");
animals.stream()
        .filter(s -> s.startsWith("c"))
        .forEach(System.out::println);
```

利用方法としてはこの形が一番多いかと思います！

# 中間操作

**中間操作は、値を変換したり、条件に満たない要素を除外したりといった処理を行います。**
生成したストリームに対しては、**0個以上の中間操作を行う必要があります。**

**中間操作は戻り値が`Stream`となり、その処理結果が次の処理の入力となります。**

`A処理()`での処理結果が`B処理()`にて入力し、
`B処理()`での処理結果が`C処理()`にて入力されます。
この**`.`**で繰り返し処理を繋いでいく方法を**パイプライン処理**と言います。

※**中間操作A( ).中間操作B( ).中間操作C( )**といった形です

ここからは中間操作の代表的なメソッドを紹介します。

## filter

| メソッド名 | 引数         | 戻り値    | 実装が必要な抽象メソッド | 処理説明                                                     |
| :--------- | :----------- | :-------- | :----------------------- | :----------------------------------------------------------- |
| **filter** | Predicate<T> | Stream<T> | boolean test(T t)        | Predicateの定義がtrueになる要素のみを抽出し、そのストリームを返す |

```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
numbers.stream()
        .filter(n -> n % 2 == 0)
        .forEach(System.out::println); // 2 4 6 8 10
```

表だけ見ると逆にわかりづらいと感じるかもしれません。

最初なので細かく見ていくと、流れとしては
・`filter`メソッドは引数として(関数型インターフェース)`Predicate<T>`を受け取る
・**`n -> n % 2 == 0`** の部分がラムダ式になっており、Predicateの抽象メソッド`test`を実装
・条件式に合致する(trueになる)要素を抽出する(ここでは偶数の数字)
・filterメソッドは戻り値にStreamを返すので、ここでは偶数の数字のみとなったStreamを返却する
・そのStreamを終端操作である`forEach`メソッドに渡して出力

こんなイメージになるかと思います。

参考までにラムダ式でなく無名クラスを使った場合のコードを載せておきます。
こちらの方がイメージがつきやすいかもしれませんね。

```
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    numbers.stream()
            .filter(new Predicate<Integer>() {
                //関数型インターフェースPredicateの抽象メソッド(test)を実装
                @Override
                public boolean test(Integer n) {
                    return n % 2 == 0;
                }
            })
            .forEach(System.out::println); // 2 4 6 8 10
}
```

## map

| メソッド名 | 引数          | 戻り値    | 実装が必要な抽象メソッド | 処理説明                                                     |
| :--------- | :------------ | :-------- | :----------------------- | :----------------------------------------------------------- |
| **map**    | Function<T,R> | Stream<R> | R apply(T t)             | ストリームの要素に指定された関数を適用した結果から構成されるストリームを返す |

・ T型の`Stream`を受け取り
・ その要素をR型に変換し
・ R型の`Stream`を返却する

ということになります。
`T`と`R`に関しては同じ型でも問題ありません。

その為、
**元の値に対して、値や型を別のものに置き換える処理**と考えて問題ないかと思います。

```
List<Integer> numbers = Arrays.asList(1,2,3,4,5);
numbers.stream()
        .map(n -> n * 10)
        .forEach(System.out::println); // 10 20 30 40 50
```

ここでのmapメソッドの処理は、
Integer型のStreamを受け取って、その値を10倍にしたStreamを返却しています。

## distinct

| メソッド名   | 引数 | 戻り値    | 処理説明                                                     |
| :----------- | :--: | :-------- | :----------------------------------------------------------- |
| **distinct** |  -   | Stream<T> | このストリームの重複を除いた要素から構成されるストリームを返す |

**重複の除外処理です。**
裏側的な処理としては、**Object.equals**を用いて各要素が比較され、重複の判定が行われています。

```
List<Integer> numbers = Arrays.asList(1, 1, 2, 2, 3, 4, 5);
numbers.stream()
        .distinct()
        .forEach(System.out::println); // 1 2 3 4 5
```

## limit

| メソッド名 | 引数 | 戻り値    | 処理説明                                                     |
| :--------- | :--- | :-------- | :----------------------------------------------------------- |
| **limit**  | long | Stream<T> | このストリームの要素をmaxSize以内の長さに切り詰めた結果から成るストリームを返す |

**引数で渡されたn個分の要素を抽出する処理です。**

```
List<String> animals = Arrays.asList("cat", "dog", "cow");
animals.stream()
        .limit(2)
        .forEach(System.out::println); // cat dog
```

## sorted ①

| メソッド名 | 引数 | 戻り値    | 処理説明                                                     |
| :--------- | :--: | :-------- | :----------------------------------------------------------- |
| sorted     |  -   | Stream<R> | このストリームの要素を自然順序に従ってソートした結果から構成されるストリームを返す |

大雑把に言うと、
**自然順序によってソートしてくれる処理です。**

```
List<Integer> numbers = Arrays.asList(5,2,3,4,1);
    numbers.stream()
            .sorted()
            .forEach(System.out::println); //1 2 3 4 5
```

## sorted ②

実はsortedには2種類存在します。

| メソッド名 | 引数          | 戻り値    | 処理説明                                                     |
| :--------- | :------------ | :-------- | :----------------------------------------------------------- |
| sorted     | Comparator<T> | Stream<R> | ストリームの要素に指定された関数を適用した結果から構成されるストリームを返す |

こちらは引数としてインターフェースのComparatorを渡します。
これによって、ソート方法を指定することが可能となります。

抽象メソッドだけでなくデフォルトメソッドによる書き方もあったり、説明が長くなるのでここでは記述しません。
深く知りたいという方はこちらの方の投稿を見て頂くのがいいと思います。
https://qiita.com/tag1216/items/50ecf6a7bc10218ee889

他にも様々な中間操作がありますが、基礎知識としては一旦ここまでにします。

# 終端操作

**終端操作は1つのStreamパイプラインにおいて、1つしか持つことができません。**
中間操作と違い、Stream以外の戻り値を返すことで、Streamの処理を終了させています。

終端操作の代表的なメソッドを紹介します。

## forEach

| メソッド名 | 引数        | 戻り値 | 処理説明                                           |
| :--------- | :---------- | :----- | :------------------------------------------------- |
| forEach    | Consumer<T> | void   | このストリームの各要素に対してアクションを実行する |

何度も出てきているので説明はいらないかもしれません。
Stream内の各要素に対して引数で渡された処理（アクション）を実行します。

## allMatch, anyMatch, noneMatch

| メソッド名 | 引数         | 戻り値  | 処理説明                                                     |
| :--------- | :----------- | :------ | :----------------------------------------------------------- |
| allMatch   | Predicate<T> | boolean | 引数として渡された値が、このストリーム内の**すべての要素と一致する場合trueを返す** |
| anyMatch   | Predicate<T> | boolean | 引数として渡された値が、このストリーム内の**いずれかの要素と一致する場合trueを返す** |
| noneMatch  | Predicate<T> | boolean | 引数として渡された値が、このストリーム内に**存在しない場合trueを返す** |

条件を判定する為のメソッドです。
引数としては戻り値としてbooleanを返す関数型インターフェースのPredicateが用いられています。

```
List<String> animals = Arrays.asList("ape", "bear", "cat", "cow", "camel", "dog");

//allMatch
boolean all = animals.stream().allMatch(s -> s.equals("dog"));
System.out.println(all); //false
//anyMatch
boolean any = animals.stream().anyMatch(s -> s.equals("dog"));
System.out.println(any); //true
//noneMatch
boolean none = animals.stream().noneMatch(s -> s.equals("dog"));
System.out.println(none); //false
```

## min, max, count

| メソッド名 | 引数          | 戻り値      | 処理説明                           |
| :--------- | :------------ | :---------- | :--------------------------------- |
| min        | Comparator<T> | Optional<T> | ストリーム内の**最小要素**を返す   |
| max        | Comparator<T> | Optional<T> | ストリーム内の**最大要素**を返す   |
| count      | -             | Long        | ストリーム内の**要素の個数**を返す |

ここまで来るとExcelの関数に近いですね。
Excelではセル内でしたが、StreamではStream内です。

minとmaxに関しての補足ですが、
引数は、比較に用いられるインターフェースの`Comparator`
戻り値は、`Optional`を用います。

この2つに関しては長くなるので別の機会で投稿を行いたいと思いますが、ざっくりとだけ説明すると、
インターフェースのComparatorでStreamの要素を比較し、min/maxメソッドの処理内容に従った要素を返しています。
この要素はnullの可能性があるので、Optionalでラップしてnullかもしれないことを許容する形をとっています。

```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 10);
//min
Optional<Integer> minNum = numbers.stream().min(Comparator.naturalOrder());
System.out.println(minNum.get()); // 1
//max
Optional<Integer> maxNum = numbers.stream().max(Comparator.naturalOrder());
System.out.println(maxNum.get()); // 10
//count
long countNum = numbers.stream().count();
System.out.println(countNum); // 6
```

## collect

| メソッド名 | 引数             | 戻り値 | 処理説明                                          |
| :--------- | :--------------- | :----- | :------------------------------------------------ |
| collect    | Collector<T,A,R> | R      | Streamの要素に特定の処理を行って1つにまとめる操作 |

collectも様々な処理が行なえますが、基本的によく使うのは、
Collectorsクラスのstaticメソッドを用いる形です。

Collectorsは引数であるCollectorを実装しているクラスで、様々なメソッドが用意されています。
ここでは一部のメソッドをご紹介します。

### `Collectors.toList`

**Streamの処理結果を最終的にListに変換します。**

```
List<String> animals = Arrays.asList("ape", "bear", "cat", "cow", "camel", "dog");
List<String> results = animals.stream()
                             .filter(s -> s.startsWith("c"))
                             .map(s -> s.toUpperCase())
                             .collect(Collectors.toList());
//resultsの要素確認
results.forEach(System.out::println); //CAT COW CAMEL
```

同じコレクションを扱うものとして、SetやMapに変換するメソッド（`toSet`,`toMap`)も用意されています。

### `Collectors.joining`

**文字列を区切り文字で連結し、指定された接頭辞と接尾辞等を付加できます。**

```
List<String> str = Arrays.asList("one", "two", "three");
String results = str.stream().collect(Collectors.joining("-","[","]"));
System.out.println(results); //[one-two-three]
```

※ここでは、
"-" が区切り文字
"[" が接頭辞
"]" が接尾辞

### `Collectors.groupingBy`

**Streamの要素をグループ化した結果を返します。**

血液型でグルーピングするコード例を紹介します。

Person.java

```
public class Person {

    private String name;
    private int age;
    private String bloodType;

    Person(String name, int age, String bloodType) {
        this.name = name;
        this.age = age;
        this.bloodType = bloodType;
    }

    //getterとsetterは省略

    @Override
    public String toString() {
        return this.getName() + "(" + this.getAge() + "歳 " + this.getBloodType() + "型)";
    }
}
```

StreamSample.java

```
public static void main(String[] args) {
    List<Person> personList = new ArrayList<Person>();

    personList.add(new Person("John", 20, "A")); //名前,年齢,血液型
    personList.add(new Person("Tom", 25, "O"));
    personList.add(new Person("Mike", 30, "B"));
    personList.add(new Person("Sara", 20, "O"));
    personList.add(new Person("Kate", 25, "A"));

    Map<String, List<Person>> grpByBloodType = personList.stream()
            .collect(Collectors.groupingBy(Person::getBloodType));

    grpByBloodType.entrySet().forEach(entry -> {
        System.out.println("Key: " + entry.getKey() + " Value: " + entry.getValue());
    });

    /*
    実行結果
    Key: A Value: [John(20歳 A型), Kate(25歳 A型)]
    Key: B Value: [Mike(30歳 B型)]
    Key: O Value: [Tom(25歳 O型), Sara(20歳 O型)]
    */
}
```

# まとめ

これまで4回に分けてStream API関連の投稿を行ってきました。
Stream APIは奥が深くてここでは本当に基礎の部分しか書いていませんが、まだまだ便利な中間操作/終端操作が多数存在します！
そういった深い部分も後々は投稿していきたいと思います。

Stream APIを活用するには様々な知識が必要なので、ぜひ過去の投稿もご覧頂ければと思います！