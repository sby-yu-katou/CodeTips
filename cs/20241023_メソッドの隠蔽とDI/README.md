ここでのコードは .NET8 で書いています。

# メソッドの隠蔽と DI
## メソッドの隠蔽
とにかくこのコードを見てくれ。
``` cs
public class BaseClass
{
  public void M()
  {
    Console.WriteLine("BaseClass.M");
  }

  public virtual void MM()
  {
    Console.WriteLine("BaseClass.MM");
  }
}

public class DerivedClass : BaseClass
{
  new public void M()
  {
    Console.WriteLine("DerivedClass.M");
  }

  public override void MM()
  {
    Console.WriteLine("DerivedClass.MM");
  }
}

  public class Program
{
  public static void Main(string[] args)
  {
    var b = new BaseClass();
    var d = new DerivedClass();
    BaseClass bd = new DerivedClass();

    // メソッドの隠蔽
    b.M();    // BaseClass.M
    d.M();    // DerivedClass.M
    bd.M();   // BaseClass.M (DerivedClass.M が実行されない！)

    Console.WriteLine("----");

    // メソッドのオーバーライド
    b.MM();   // BaseClass.MM
    d.MM();   // DerivedClass.MM
    bd.MM();  // DerivedClass.MM
  }
}
```

メソッドの修飾子に `new` を付けることで基本クラスのメソッドを隠蔽することができる。このとき、派生クラスのオブジェクトでありながら基本クラスとして扱われる場合、`new` された `M` メソッドは呼ばれず、基本クラスの `M` メソッドが呼ばれることになる。
一方、`virtual` 修飾子を付けたメソッドをオーバーライドするようにすると、派生クラスのメソッドが呼ばれるようになる。
この挙動は C# の基礎として当然の挙動である。

## DI でのサービス登録
Dependency Injection（依存関係の注入）については DI コンテナを始めとしてよく知られる実装方法で、Blazor なんかでは気軽にこんな感じでサービスを登録している。
``` cs
// ○○サービス
builder.Services.AddScoped<IBaseClass, BaseClass>();
```
インターフェースとクラスはそれぞれこんな感じである。
``` cs
public interface IBaseClass
{
  bool M();
}

public class BaseClass : IBaseClass
{
  public bool M()
  {
    // 何か処理
    return true;
  }
}
```

この後、`BaseClass` の機能を拡張した `DerivedClass` を作ったとしよう。
``` cs
public class DerivedClass : BaseClass
{
  public bool M()   // CS0108 発生
  {
    var b = base.M();

    // 追加の処理
    return false;
  }
}
```
ところが上記のコードはエラーが起こる。`BaseClass` には既に `M` メソッドが実装されているため、`new` を付けてメソッドの隠蔽をおこなえ、という CS0108 が発生する。それではと `new` を付けてメソッドの隠蔽をしたところで満足し、`Program.cs` のサービス登録を書き換える。
``` cs
public class DerivedClass : BaseClass
{
  new public bool M()   // CS0108 発生を回避
  {
    var b = base.M();

    // 追加の処理
    return false;
  }
}
```

``` cs
// ○○を拡張したサービス
builder.Services.AddScoped<IBaseClass, DerivedClass>();
```

この場合、このサービスを使う側のコードは例えばこうだ。
``` cs
public class Sample(IBaseClass baseClass)
{
  private readonly IBaseClass _baseClass = baseClass;

  public bool DoSomething() => _baseClass.M();
}
```

こうなると、`baseClass` は `DerivedClass` オブジェクトなのにもかかわらず `DerivedClass.M` メソッドが実行されることがなく、「なぜだ！」と延々とデバッグに苦しむことになる。

## 解説
まずメソッドの隠蔽という機能の使いどころを間違えてはいけません。この機能は、例えばサードパーティ製のクラスに対して機能拡張したいときなど、クラス定義を直接変更できない場合に活躍します。自作クラスの機能拡張であれば元のメソッドに `virtual` 修飾子を付けて、派生クラスでオーバーライドすべきです。

メソッドの隠蔽は読んで字のごとくメソッドが隠蔽されることになるため、普通に考えれば真っ向から LSP（リスコフの置換原則）に違反することになります。ただし、元のクラスではなく拡張した後のクラスでしか用途がないのであれば特に問題になることはありません。つまり、`IBaseClass` あるいは `BaseClass` として使うのではなく、必ず `DerivedClass` として使うのであれば問題ないということです。

しかし、それでも `IBaseClass` あるいは `BaseClass` でキャストできてしまう、そして `DrivedClass.M` メソッドを呼ぶつもりが `BaseClass.M` メソッドが呼ばれてしまう危険性は常にはらんでしまうわけですから、基本的にメソッドの隠蔽はおこなわないほうが無難、という結論になります。

インターフェースなどで抽象化している時点でメソッドの隠蔽を必要とするシチュエーションは避けるべきです。