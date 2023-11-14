出所：[ジェネリクス、PECS、Get&Put原則 | ならのブログ (ameblo.jp)](https://ameblo.jp/narazumono0321/entry-11459198426.html)

取得年月日：2023(R5).11.13

# ジェネリクス、PECS、Get&Put原則

メソッドがパラメータ化された型を引数や返却値として扱う場合、パラメータ化された型は、ワイルドカードを利用した形式でほぼ必ず表現されるべき。

このとき、上限境界を用いるのか下限境界を用いるのか。

producer -extends consumer -super から、PECS と表現されたり、
Get&Put原則と表現されたりするがいまいちピンとこない。

結論から言うと、
　・引数は -extends
　・返却値は -super
とするのがほぼ正しい。

例えば
public List<Integer> getList()
というジェネリクスメソッドがあったとする、これはこのままだとInteger型のリストしか返却することができないし、当然、受け取る側のコードもList<Integer>でしか受ける事ができない。
ジェネリクスに限らず、呼び出しもとの変数がスーパータイプで受け取るという場面はオブジェクト指向プログラミングなら良くある事。このままだとそういった柔軟性が発揮できない。
次のように上限境界を用いることで解決できる。
public List<? super Integer> getList()
これで受け取る側のコードはList<? super Integer>型の変数で柔軟に受け取る事が可能となる。

public void setList(List<Number> list)
というジェネリクスメソッドがあったとする、これはこのままだとNumber型のリストしか受け入れることができず、Integer型のリストは受け入れる事ができない。
次のように下限境界を用いることで解決できる。
public void setList(List<? extends Number> list)