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
プライマリコンストラクタの機能を使うことで上記のコードが次のように書けるようになります。
``` cs
public class UserInfo(string name, int age)
{
    public string Name { get; set; } = name;
    public int Age { get; set; } = age;
}
```
