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

メソッドの修飾子に `new` を付けることで基本クラスのメソッドを隠蔽することができる。この挙動は C# の基礎として誰でも知っていることだと思う。ただ、知識と知っているだけで、実が伴っていない方が多いのではなかろうか。

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

こうしてしまうと、`DerivedClass.M` メソッドが実行されることがなく、「なぜだ！」と延々とデバッグに苦しむことになる。
