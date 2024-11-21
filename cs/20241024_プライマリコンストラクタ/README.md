ここでのコードは .NET8 で書いています。

# プライマリコンストラクタ
例えばこんなクラスがあったとして、
``` cs
public class UserInfo
{
  public UserInfo(string name, int age)
  {
    Name = name;
    Age = age;
  }

  public string Name { get; set; }
  public int Age { get; set; }
}
```
プライマリコンストラクタの機能を使うことで上記のコードが次のようにコンストラクタを省略して書けるようになります。
``` cs
public class UserInfo(string name, int age)
{
  public string Name { get; set; } = name;
  public int Age { get; set; } = age;
}
```
次のように書くことで引数なしのコンストラクタに対しても初期値を設定できます。
``` cs
public class UserInfo(string name, int age)
{
  public UserInfo()
    : this("名無し", 0)
  {
  }

  public string Name { get; set; } = name;
  public int Age { get; set; } = age;
}
```

プライマリコンストラクタを使うことで、そのクラスが何に依存しているのかが一目で分かるようになります。
例えばこんなコードについて、
``` cs
public class LoginController
{
  public LoginController(IAuthenticationService authenticationService)
  {
    _authenticationService = authenticationService;
  }

  private readonly IAuthenticationService _authenticationService;
}
```
これだけ短いコードならすぐに `IAuthenticationService` インターフェースに依存していることが分かりますが、実際のコードはもう少し見にくくなってくるので、そう簡単には分かりません。これがプライマリコンストラクタを使うと次のように書けるようになります。
``` cs
public class LoginController(IAuthenticationService authenticationService)
{
  private readonly IAuthenticationService _authenticationService = authenticationService;
}
```
こう書くとほんの少しの差なのであまり実感できないかもしれませんが、実際にコードを目の当たりにするとじわじわと実感できると思います。

実は、プライマリコンストラクタは `record` の定義の書き方を `class` にも拡張したもので、その恩恵は `record` のほうで発揮されます。
