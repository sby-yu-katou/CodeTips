ここでのコードは .NET8 で書いています。

# is 演算子

## 型テストとしての is 演算子
値または式の結果のランタイム型と、指定された型の間に互換性があるかどうかを調べる場合に `is` 演算子を使います。例えば次のような使い方ができます。

``` cs
// 数値リテラルの型推論の既定値は int なので value は int になる
var value = 10;

if (value is int)
{
  // value が int 型と互換性がある場合に何か処理をする
}

if (value is long)
{
  // `var value = 10` という宣言文から、数値リテラル `10` に対して型推論 `var` が適用されている
  // 数値リテラルの型推論は int なので `value` は int 型となる
  // したがってここは処理されない
  // int は long に変換できるが、is 演算子では数値変換まで考慮しない
}

long longValue = 10;
if (longValue is int)
{
  // 明確に long なのでここは処理されない
}
```

`is` 演算子は互換性を考慮するため、派生クラスに対して次のような挙動になります。

``` cs
// 動物を表すクラス
public class Animal { }

// キリンを表すクラス
public class Giraffe : Animal { }

public static class CsSample
{
  public static void Main()
  {
    var giraffe = new Giraffe();
    Console.WriteLine(giraffe is Animal);   // output: True

    Console.WriteLine(giraffe is Giraffe);  // output: True
  }
}
```

厳格な型チェックをしたい場合は `GetType` メソッドを用いた `typeof` 演算子による型テストをおこなう必要があります。
``` cs
// 動物を表すクラス
public class Animal { }

// キリンを表すクラス
public class Giraffe : Animal { }

public static class CsSample
{
  public static void Main()
  {
    var giraffe = new Giraffe();
    Console.WriteLine(giraffe.GetType() == typeof(Animal));   // output: False

    Console.WriteLine(giraffe.GetType() == typeof(Giraffe));  // output: True
  }
}
```

## パターンマッチングとしての is 演算子
### 宣言パターン
``` cs
object str = "Hello, world!";
if (str is string message)
{
  // 変数 message は string なので ToLower メソッドにアクセスできる
  Console.WriteLine(message.ToLower());
}
```

### var パターン
式の結果を受けるときに型推論を利用して `var` による宣言パターンを書くこともできます。この場合、式の結果がどんな型であるかはあらかじめ分かっているため、`if` 文のステートメントで用いるより、その後の処理で使用する計算の中間結果を保持する一時変数としての用途に向いています。
``` cs
public static class Program
{
  private static int SimulateRandomValue()
  {
    var random = new Random();
    return random.Next(0, 100);
  }

  // メソッドの戻り値によって処理を分岐するが、
  // その戻り値を var パターンで受け取ることで式を簡略化している
  private static bool IsAcceptable() =>
    SimulateRandomValue() is var value
    && value > 50
    && value < 75;

  public static void Main()
  {
    Console.WriteLine(IsAcceptable() ? "Accept!" : "Oops.");
  }
}
```

`is` 演算子を使わない場合、計算の中間結果を保持する変数を明確に宣言するため、ステートメントが複数行になります。
``` cs
  private static bool IsAcceptableOld()
  {
    var value = SimulateRandomValue();
    return (value > 50) && (value < 75);
  }
```

宣言パターンで紹介したコードに対して `var` を使用してもコンパイルできますが、変数名が `str` から `message` に変わるだけで意味がないので、このような使い方はしません。
``` cs
object str = "Hello, world!";
if (str is var message)
{
  // 変数 message は object なので ToLower メソッドにアクセスできず、コンパイルエラーになる
  Console.WriteLine(message.ToLower());
}
```

### リストパターン
リストの各要素に対してそれぞれ `is` 演算子による比較をおこない、すべての要素がそれぞれ一致するかどうかを調べることができます。
``` cs
int[] numbers = [1, 2, 3];

Console.WriteLine(numbers is [1, 2, 3]);            // True
Console.WriteLine(numbers is [1, 2, 4]);            // False
Console.WriteLine(numbers is [1, 2, 3, 4]);         // False
Console.WriteLine(numbers is [0 or 1, <= 2, >= 3]); // True
```
この場合の `is` 演算子は定数との比較になるため、参照型の場合は `null` との比較以外では使えません。
``` cs
var a = new Person();
var b = new Person();
var c = new Person();
List<Person> people = [a, b, c];

Console.WriteLine(numbers is [null, null, null]);   // False
// Console.WriteLine(numbers is [a, b, c]);         // a, b, c は定数ではないのでコンパイルエラー

public class Person { }
```
リストパターンについては次のような要素を抽出するときに威力を発揮するのではないでしょうか。
``` cs
List<int> numbers = [1, 2, 3];

if (numbers is [_, var second, _])
{
    Console.WriteLine($"3 つの要素を持つコレクションの 2 番目の要素は {second} です.");
}
```

``` cs
List<int> numbers = [1, 2, 3, 4, 5, 6, 7];

if (numbers is [.., var number, _])
{
  Console.WriteLine($"{numbers.Count} 個の要素を持つコレクションの後ろから 2 番目の要素は {number} です.");
}
```
### プロパティパターン
プロパティパターンはプロパティに対して再帰的にパターンマッチングする機能です。
``` cs
// Name と Age というプロパティを持つオブジェクト
var person = new Person() { Name = "John", Age = 23 };

// Name プロパティに対するマッチング
if (person is { Name: "Aimy" })
{
  Console.WriteLine("Hi, Aimy.");
}

// Age プロパティに対するマッチング
if (person is { Age: > 18 })
{
  Console.WriteLine("You are an adult.");
}

// Name と Age 両方のプロパティに対するマッチング
// 数値については比較演算子を使うこともできる
if (person is { Name: "John", Age: > 18 })
{
  Console.WriteLine("Hi, John. You are an adult.");
}

// これは定数比較ではないのでコンパイルエラーになる
// var expectedName = "John";
// if (person is { Name: expectedName })
// {
//   Console.WriteLine($"Hi, {expectedName}.");
// }

public class Person
{
  public required string Name { get; set; }
  public int Age { get; set; }
}
```
どうしても変数を使って比較したい場合は、var パターンと組み合わせることで対応できます。
``` cs
var expectedName = "John";
if ((person is { Name: var name }) && (name == expectedName))
{
  Console.WriteLine($"Hi, {expectedName}.");
}
```
ただ、このサンプルコードの場合 `person.Name == expectedName` という比較式を使う方がシンプルです。他にも比較したいプロパティが複数ある場合は、プロパティパターンを使うことでコードが簡潔になることもあります。

### 否定パターン
パターンの前に `not` を付けることで比較の意味を否定することができます。
``` cs
private void M(Person? person)
{
  if (person is not null)
  {
    // person が null ではないので何か処理をする
  }
}
```
これまでは `!=` 演算子を使って比較していましたが、`==` および `!=` 演算子はオーバーロードを定義することができるため、`is` や `is not` を使った方が常に意図通りに比較されますし、パフォーマンスが良くなることもあります。また、`!=` は視認性が悪いので単に `is not` を好む場合もあります。
また、`is not` による Eary Return をおこなう場合、宣言パターンと組み合わせることで、その後続処理で宣言パターンで宣言した変数を使うことができます。
``` cs
private void M(Person person)
{
  if (person is not Baby baby) return;

  // person が Baby オブジェクトの場合に baby 変数を使って処理をする
  // if (person is Baby baby) { } と同じ意味だが、これだとネストが一つ深くなる
}

public class Person { }
public class Baby : Person { }
```
### 論理パターン
`and` や `or` を使った論理パターンを記述することができます。`()` による優先度を付けることもできます。
``` cs
var value = 23;
if (value is (> 18 and < 20) or 23)
{
  // value が 19 または 23 の場合に何か処理をする
}
```

## 補足

### プロパティパターンを利用した `null` チェック
プロパティパターンを利用してオブジェクトが `null` ではないことを判定することもできます。
``` cs
if (person is { })
{

}
```
プロパティパターンはそもそもオブジェクトが `null` ではないことを確認するところから始まります。なのでプロパティパターンを指定していない上記のコードは `null` チェックだけをおこなうことになります。

さらに応用すると、こんなこともできます。
``` cs
if (person?.Child is { } child)
{
  Console.WriteLine($"Hi, {child.Name}.");
}
```
プロパティパターンによって `null` チェックを通過した `person?.Child` オブジェクトに対して宣言パターンによって `child` という変数に格納することで、以降のスコープ内では `child` 変数が使えるようになります。

### その他のパターンマッチング
`is` 演算子の紹介としてパターンマッチングを説明しましたが、パターンマッチングとしては `is` 演算子を用いる他、`switch` 演算子を用いた定数パターンや型パターン、リレーショナルパターンもあります。
