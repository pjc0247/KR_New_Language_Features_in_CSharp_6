Original Source : https://github.com/dotnet/roslyn/wiki/New-Language-Features-in-C%23-6

New Language Features in C# 6
====
이 문서에서는 C# 6에서 새롭게 추가된 기능들에 대해서 설명합니다. 이곳에 작성된 모든 기능들은 VS 2015에서 구현되어 있습니다.

향상된 Auto-property
==========================


Auto-properties 초기화
--------------------------------

이제 field에 하던 것 처럼 auto-property에 초기화 구문을 작성할 수 있습니다.

``` c#
public class Customer
{
    public string First { get; set; } = "Jane";
    public string Last { get; set; } = "Doe";
}
```

작성된 생성자는 setter를 거치지 않고 backing field를 직접적으로 초기화하는 방식으로 작동합니다. 이 생성자들은 이전에 field들이 하던 것 처럼  쓰여진 순서대로 실행됩니다.

field 생성자처럼, auto-property의 생성자는 'this'를 가리킬 수 없습니다. - 이들은 오브젝트가 초기화 되기 전에 실행됩니다.

Getter-only auto-properties
---------------------------

이제 setter 없는 auto-property를 선언하는것이 가능합니다.

``` c#
public class Customer
{
    public string First { get; } = "Jane";
    public string Last { get; } = "Doe";
}
```

getter-only auto-property의 backing field는 'readonly' 속성으로 선언됩니다. (이걸 신경써야 할 경우는 리플렉션을 수행할 경우밖에 없습니다.) 이러한 getter-only auto-property는 아래의 예제에서 나오듯이 생성자에서 초기화를 수행할 수 있습니다. Also, a getter-only property can be assigned to in the declaring type’s constructor body, which causes the value to be assigned directly to the underlying field:


``` c#
public class Customer
{
    public string Name { get; }
    public Customer(string first, string last)
    {
        Name = first + " " + last;
    }
}
```

This is about expressing types more concisely, but note that it also removes an important difference in the language between mutable and immutable types: auto-properties were a shorthand available only if you were willing to make your class mutable, and so the temptation to default to that was great. Now, with getter-only auto-properties, the playing field has been leveled between mutable and immutable.


Expression-bodied function members
==================================

람다 표현식은 {}로 이루어진 일반적인 함수형의 body 또는 표현식 타입의 body로 선언할 수 있었습니다. 새로운 Expression-bodied func members 기능은 클래스 멤버에도 이러한 편의성을 제공합니다.

Expression bodies on method-like members
----------------------------------------

메소드와 유저가 정의한 오퍼레이터들 그리고 타입 변환기기는 표현식 타입의 body로 선언할 수 있게 되었습니다. 이 기능을 사용하려면 람다에서 사용하는 => 화살표를 이용합니다.

``` c#
public Point Move(int dx, int dy) => new Point(x + dx, y + dy); 
public static Complex operator +(Complex a, Complex b) => a.Add(b);
public static implicit operator string(Person p) => p.First + " " + p.Last;
```

이러한 방법은 메소드가 하나의 return 구문으로 이루어진 형태와 완전히 동일합니다.

void를 반환하는 메소드와, 'Task'를 반환하는 메소드들에 대해서는 이 화살표 형태의 문법은 여전히 적용되지만, 뒤에 따라오는 표현식들은 statement expression 이어야 합니다. (람다가 그렇듯이)

``` c#
public void Print() => Console.WriteLine(First + " " + Last);
```


Expression bodies on property-like function members
---------------------------------------------------

프로퍼티나 인덱서는 문법적으로 지원되는 getter와 setter를 가질 수 있었습니다. 이제부턴 getter-only의 프로퍼티나 인덱서에 표현식 타입의 body 선언을 사용할 수 있습니다.

``` c#
public string Name => First + " " + Last;
public Customer this[long id] => store.LookupCustomer(id); 
```

표현식 타입의 body로 만들어진 프로퍼티나 인덱서에는 'get' 키워드가 사용되지 않는다는 점을 기억하세요.


Using static
============

이 기능은 클래스의 static 멤버를 importㅎ하여, 접두사 없이 사용이 가능하도록 만들어줍니다. 아래의 코드에 그 예제가 있습니다.

``` c#
using static System.Console;
using static System.Math;
using static System.DayOfWeek;
class Program
{
    static void Main()
    {
        WriteLine(Sqrt(3*3 + 4*4)); 
        WriteLine(Friday - Monday); 
    }
}
```

이 기능은 특정한 domain 아래의 함수들의 집합을 가지고 있는데, 그걸 계속해서 사용해야 할 경우에 매우 유용합니다. 'System.Math'가 그 좋은 예입니다. 또한 이 기능은 enum의 각각의 이름들에 접두사 없이 접근할 수 있도록 해줍니다. 아래의 코드는 'System.DayOfWeek' enum을 사용한 예제를 보여줍니다.


Extension methods
-----------------

Extension methods are static methods, but are intended to be used as instance methods. Instead of bringing extension methods into the global scope, the using static feature makes the extension methods of the type available as extension methods:

``` c#
using static System.Linq.Enumerable; // The type, not the namespace
class Program
{
    static void Main()
    {
        var range = Range(5, 17);                // Ok: not extension
        var odd = Where(range, i => i % 2 == 1); // Error, not in scope
        var even = range.Where(i => i % 2 == 0); // Ok
    }
}
```

This does mean that it can now be a breaking change to turn an ordinary static method into an extension method, which was not the case before. But extension methods are generally only called as static methods in the rare cases where there is an ambiguity. In those cases, it seems right to require full qualification of the method anyway.



Null-conditional 연산자
==========================

가끔 코드가 null-checking 덕분에 지저분해지는 것을 볼 수 있습니다. 새로운 null-conditional 연산자는 값이 null이 아닐 때만 요소 또는 멤버에 접근할 수 있도록 해줍니다. 만약 값이 null이라면 null값을 반환합니다.

``` c#
int? length = customers?.Length; // customers가 null이라면 length에도 null이 들어갑니다.
Customer first = customers?[0];  // customers가 null이라면 first에도 null이 들어갑니다.
```

null-conditional 연산자는 '??' 연산자와 같이 사용하면 편리합니다.

``` c#
int length = customers?.Length ?? 0; // customers가 null이면 length는 0이 들어갑니다.
```

null-conditional 연산자 뒤에 오는 체인은 값이 null이 아닐 때만 실행됩니다.

``` c#
int? first = customers?[0].Orders.Count();
```

위의 예제는 아래와 동일한 의미를 가집니다.

``` c#
int? first = (customers != null) ? customers[0].Orders.Count() : null;
```

Except that `customers` is only evaluated once. None of the member accesses, element accesses and invocations immediately following the `?` are executed unless `customers` has a non-null value.


당연하게도 null-conditional 연산자끼리도 체인될 수 있습니다. 아래의 코드에서는 체인을 사용하여 두 번의 null 체크를 하는것을 보여줍니다.

``` c#
int? first = customers?[0].Orders?.Count();
```

Note that an invocation (a parenthesized argument list) cannot immediately follow the `?` operator – that would lead to too many syntactic ambiguities. Thus, the straightforward way of calling a delegate *only* if it’s there does not work. However, you can do it via the `Invoke` method on the delegate:

``` c#
if (predicate?.Invoke(e) ?? false) { … }
```

우리는 이 패턴이 이벤트를 트리거하는데에 있어서 널리 쓰이기를 기대합니다.

``` c#
PropertyChanged?.Invoke(this, args);
```

이것은 이벤트를 트리거하기 전에 null을 체크하기 위한 간편하면서도 스레드에 안전한 방법입니다. 이 기능이 스레드에 안전할 수 있는 이유는 '?" 왼쪽의 값을 처음 딱 한번 판단한 후 임시 변수에 담아서 사용하기 때문입니다.



문자열 보간
====================

'String.Foramt'류의 기능은 매우 유용하고 다재다능했지만, 모양새가 좋지 않앗고 에러의 소지 또한 있엇습니다. 특히 짜증나는건 '{0}' 처럼 생긴 플레이스홀더들이었는데, 이건 포멧 스트링과 별도의 인자(argument)로 줄줄이 전달되어야 했습니다.

``` c#
var s = String.Format("{0} is {1} year{{s}} old", p.Name, p.Age);
```

새로운 문자열 보간은 표현식이 들어가야 할 올바른 자리에 위치할 수 있도록 해줍니다. 

``` c#
var s = $"{p.Name} is {p.Age} year{{s}} old";
```

기존의 'String.Format'에서 하던 것 처럼, 정렬 옵션과 포멧 지정자(specifier)를 사용할 수 있습니다.

``` c#
var s = $"{p.Name,20} is {p.Age:D3} year{{s}} old";
```

{} 안의 내용에는 아무 표현식이나 위치할 수 있으며, 문자열을 넣는 것도 가능합니다.

``` c#
var s = $"{p.Name} is {p.Age} year{(p.Age == 1 ? "" : "s")} old";
```

위의 코드에서 삼항 연산자 식은 괄호() 안에 포함되어 있습니다. 그렇기 때문에 `: "s"`는 포멧 지정자(specifier)와 혼동되지 않습니다.


nameof expressions
==================

가끔 프로그램의 요소의 이름을 string 형태로 가져오고 싶을 때가 있습니다. 'ArgumentNullException'에서 문제가 되는 인자의 이름을 찾고 싶을 때, 'PropertyChanged' 이벤트에서 변경된 프로퍼티의 이름을 가져오고 싶을 때 처럼 말입니다.

위의 경우에 단순히 스트링 리터럴을 사용하는 것은 매우 간단한 방법이지만 문제의 소지가 있었습니다. 오타를 낼 수도 있었고, 리팩토링 기능을 이용할 때 실제 변수의 이름은 바뀌었지만 문자열 내용까지 바꾸지 않는 문제도 있었습니다. nameof는 이러한 요구에 대해 깔끔하게 처리할 수 있는 방법을 제공합니다. 

``` c#
if (x == null) throw new ArgumentNullException(nameof(x));
```

nameof를 사용할 때 '.'이 들어간 표현식을 사용할 수 있습니다. 하지만 이건 단순히 컴파일러가 어디를 찾아보아야 할지를 알려주는 것 뿐이고, 마지막에 사용된 식별자만이 사용됩니다.

``` c#
WriteLine(nameof(person.Address.ZipCode)); // prints "ZipCode"
```


Index initializers
==================

오브젝트와 콜렉션의 생성자들은 오브젝트의 필드와 프로퍼티, 그리고 콜렉션의 초기 값들을 설정하는 편리한 방법을 제공하였습니다. 하지만 Dictionary와 다른 오브젝트들의 indxer를 이용하여 초기화 하는 방법은 매우 깔끔하지 못했는데, 이를 해결하기 위해 우리는 오브젝트가 가지고 있는 인덱서를 사용하여 값을 설정할 수 있는 새로운 문법을 추가하였습니다.

``` c#
var numbers = new Dictionary<int, string> {
    [7] = "seven",
    [9] = "nine",
    [13] = "thirteen"
};
```


Exception filters
=================

VB는 원래 가지고 있었고, F# 또한 있었습니다. 이제 C#에도 탑재될 차례입니다. 

``` c#
try { … }
catch (MyException e) when (myfilter(e))
{
    …
}
```

만약 괄호 안의 값이 true로 판정되면 catch 블록이 실행됩니다. 그렇지 않으면 exception은 다른 catch를 찾아 계속 진행합니다.

익셉션 필터는 catch 후 다시 throw 하는 기존의 방법보다 좋은데, 익셉션 필터를 사용하면 stack이 손상되지 않은(unharmed) 채로 남아있기 때문입니다. 익셥션이 발생한 후 스택을 덤프했을 때, catch 후 rethrow하는 방법은 단순히 어느곳에서 rethrow 햇는지밖에 알 수 없었지만, 익셉션 필터는 익셉션이 어디서 왔는지를 온전하게 보존합니다. 

It is also a common and accepted form of “abuse” to use exception filters for side effects; e.g. logging. They can inspect an exception “flying by” without intercepting its course. In those cases, the filter will often be a call to a false-returning helper function which executes the side effects:
아래의 예제는 익셉션 필터를 특이하게 이용하여 응용하는 방법을 보여줍니다. 이러한 방법을 사용하면 익셉션의 처리 경로를 변경하지 않으면서도, 익셉션 객체를 볼 수 있습니다.

``` c#
private static bool Log(Exception e) { /* log it */ ; return false; }
…
try { … } catch (Exception e) when (Log(e)) {}
```


Await in catch and finally blocks
=================================

C# 5에서는 'await' 키워드를 catch와 finally 블록 안에서 사용하는것이 허용되지 않았습니다. 그 당시에 우리는 그걸 구현하는것은 불가능하다고 확신했엇지만, 결국 불가능하지 않은 일로 만들어냈습니다. 

이건 매우 심각한 제약사항이었고, 사람들은 이 문제를 해결하기 위해 보기 흉한 방법으로 해결책을 만들어야 했습니다. 하지만 이런 괴상한 플랜 B는 더이상 필요 없게 되었습니다.

``` c#
Resource res = null;
try
{
    res = await Resource.OpenAsync(…);       // 원래부터 이렇게 할 수 있었습니다.
    …
} 
catch(ResourceException e)
{
    await Resource.LogAsync(res, e);         // 이제 이런 것도 할 수 있습니다.
}
finally
{
    if (res != null) await res.CloseAsync(); // … 이것도
}
```

이것의 구현은 매우 복잡하지만, 사용자는 이것에 대해서 전혀 신경 쓰지 않아도 됩니다. 그게 바로 async가 있는 언어에서의 중요한 점 입니다.


Extension Add methods in collection initializers
================================================


우리가 C#의 콜렉션 생성자를 처음 구현했을 때, extension으로 확장된 'Add' 메소드는 사용할 수 없었던 버그가 있었습니다. VB에서는 처음부터 올바르게 동작하였기 때문에, 우리가 C#에 대해서는 잊고 있었던 것 같습니다. 이 버그는 수정되엇고, 이제 문제 없이 오브젝트의 생성자에서 확장된 'Add' 메소드를 사용할 수 있습니다. 이건 대단한 기능은 아니면서도 가끔 유용하게 사용됩니다. 우리는 이에 해당하는 부분을 새로운 컴파일러에서 수정하여, 문제 없이 작동하도록 하였습니다.


Improved overload resolution
============================

오버로딩에 있어서 몇가지 개선 사항이 있었습니다. 이 개선 사항들은 컴파일러로 하여금 주어진 인자들을 가지고 올바른 메소드를 선택할 수 있도록 해, 좀 더 사용자가 기대한 결과대로 동작하도록 해줍니다.

눈치 챌 수 있는 변경 사항은, when choosing between overloads taking nullable value types. Another is when passing method groups (as opposed to lambdas) to overloads expecting delegates. 여기에 자세한 사항을 적을 만큼 가치 있는 변화는 아니었지만, 그저 무언가가 개선 사항이 있었다는 것을 알려주고 싶었습니다.<br>
(예제 코드를 안적어놔서 완벽하게 이해도 안되고 함부러 번역이 안됨)
