# XLua 설정

모든 xLua 설정은 태깅(tagging), 스태틱 리스트(StaticList), 다이나믹 리스트(DynamicList) 3 가지 방식을 지원합니다:

설정을 위해서는 2가지 요구 조건과 2가치 권장 사항이 있습니다.

* 리스트 모드는 스태틱 필드(StatisFields)나 스태틱 프로퍼티(StaticProperties)를 반드시 사용해야 합니다.
* 리스트 모드는 스태틱 타입으로 배치되어야 합니다.
* 태깅 사용은 권장하지 않습니다.
* 리스트 모드 설정은 에디터(Editor) 디렉토리안에 두는 것이 좋습니다.

**태깅(Tagging)**

XLua 는 코드 생성 허용 목록을 사용하며, 허용 목록은 속성(Attribute)로 설정됩니다. 예를 들어 Lua 에서 C# 타입을 호출하거나 어댑터 코드를 생성하려면 LuaCallCSharp 태그를 타입 위에 추가해 줍니다.

~~~csharp
[LuaCallCSharp]
public ClassA
{

}
~~~

태깅(Tagging) 방식은 편리하지만 il2cpp 코드량을 증가시키므로 권장하지 않습니다.

**스태틱 리스트(StaticList)**

시스템 API 나 소스 코드 없는 라이브러리, 인스턴스화된 제너릭 타입처럼 직접적으로 태깅할 수 없는 경우 정적 타입으로 정적 필드를 선언하게 됩니다. 정적 필드는 `IEnumerable<Type>`으로 구현되는 제외 목록(BlackList)이나 추가 프로퍼티(AdditionalProperties)외에는 어떤 타입이든 사용할 수 있습니다. (해당 예외 상황에 대해서는 뒤에서 다시 설명하겠습니다.)
다음은 필드에 태그를 추가하는 방법입니다.

~~~csharp
[LuaCallCSharp]
public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
{
    typeof(GameObject),
    typeof(Dictionary<string, int>),
};
~~~

This field needs to be placed in a **static type** and placing it in the **Editor directory** is recommended.

해당 필드는 **스태틱 타입(StaticType)**으로 배치할 필요가 있고 **에디터(Editor) 디렉토리**안에 두는 것을 추천합니다.

**다이나믹 리스트(Dynamic list)**

아래처럼 스태틱 프로퍼티(StaticProperty)와 태그(Tag) 선언합니다.

~~~csharp
[Hotfix]
public static List<Type> by_property
{
    get
    {
        return (from type in Assembly.Load("Assembly-CSharp").GetTypes()
                where type.Namespace == "XXXX"
                select type).ToList();
    }
}
~~~

Getter 는 코드입니다. 네임스페이스(Namespace) 설정, 어셈블리 설정 등과 같이 많은 결과가 필요한 구현에 사용할 수 있습니다.

해당 프로퍼티는 **스태틱 타입(StaticType)**으로 배치할 필요가 있고 **에디터(Editor) 디렉토리**안에 두는 것을 추천합니다.

### XLua.LuaCallCSharp

C# 타입에 LuaCallCSharp 설정을 적용하면 xLua 는 해당 타입에 대한 (타입 인스턴스 생성, 멤버 프로퍼티&메서드 접근, 스태틱 프로퍼티&메서드 접근 등을 포함한) 어댑터(Adaptor) 코드를 생성합니다. 설정이 없다면 낮은 성능을 보여주는 리플렉션 모들 사용해 접근을 시도합니다.

타입의 익스텐션 메소드(ExtensionMethods)에 설정하면 어댑터(Adaptor) 코드가 생성되고 확장된 타입 멤버 메서드에 추가됩니다.

XLua 는 이 설정으로 로드된 타입만 코드를 생성하며 부모 타입 어댑터(Adaptor)코드는 자동으로 생성하지는 않습니다. 자식 타입 오브젝트의 부모 타입 메서드에 접근할 때 부모 타입이 LuaCallCSharp 설정을 가지고 있다면 부모 타입 어댑터 코드를 실행합니다.  설정이 없다면 리플렉션 모드를 사용해 접근을 시도합니다. 

리플렉션 모드 억세스는 느리게 작동하며 il2cpp 에서 접근 실패가 발생할 수 있습니다. 해당 문제는 아래에서 설명할 ReflectionUse 태그를 통해 방지할 수 있습니다.

### XLua.ReflectionUse

C# 타입에 ReflectionUse 설정을 적용하면 xLua 는 link.xml 을 생성해서 il2cpp 에서 코드 스트립(Stripping)을 차단합니다.

익스텐션 메서드(ExtensionMethods)를 사용하려면 반드시 LuaCallCSharp 혹은 ReflectionUse 를 추가해 접근할 수 있도록 만들어 줘야 합니다.

모든 플랫폼에서 제대로된 작동을 기대하기 위해서는 Lua 에서 접근하는 모든 타입에 대해 LuaCallCSharp 이나 ReflectionUse 태그를 설정하는 것이 좋습니다.

### XLua.DoNotGen

DoNotGen 설정은 타입의 일부 함수, 필드, 프로퍼티에 대한 코드가 생성되지 않고 리플렉션 모드를 통해서만 접근되는 것을 말합니다.

`Dictionary<Type, List<string>>`안에 있는 필드와 프로퍼티에만 사용할 수 있습니다. 키는 유효한 타입을 나타냅니다. 코드 생성되지 않는 함수, 필드, 프로퍼티 이름을 설정합니다.

ReflectionUse 와의 차이점은 다음과 같습니다.

  1. ReflectionUse 는 전체 타입을 지정합니다.
  2. 함수 (필드, 프로퍼티) 접근시 ReflectionUse 는 전체 타입을 랩핑하고, DoNotGen 은 해당 함수 (필드, 속성)만 랩핑합니다. 다시 말하자면 DoNotGen 은 좀 더 지연된 작동을 합니다.

제외 목록(BlackList) 와의 차이점은 다음과 같습니다.

  1. 제외 목록(BlackList)는 설정시 사용할 수 없습니다.
  2. 제외 목록(BlackList)는 오버로드된 함수를 지정할 수 있지만, DoNotGen 은 지정할 수 없습니다.

### XLua.CSharpCallLua

CSharpCallLua 설정을 사용하면 Lua 함수를 C# 델리게이트(delegate)에 적용할 수 있습니다. C# 쪽에 UI 이벤트, 델리게이트(delegate) 파라미터, `List<T>:ForEach` 같은 다양한 콜백을 사용하는 상황, LuaTable 의 Get 함수를 사용해 Lua 함수가 델리게이트(delegate)에 바인딩하는 상황, Lua 테이블을 C# 인터페이스에 적용하는 상황등에 사용될 수 있습니다. 델리게이트(delegate)나 인터페이스(interface)에 설정이 필요합니다.

### XLua.GCOptimize

GCOptimize 설정은 C# 순수한 value 타입 (주의: 오직 value 타입만을 가지고 있는 struct 을 말합니다)이나 C# 열거형 값에 사용됩니다. xLua 는 해당 유형에 GC 최적화된 코드를 생성합니다. 결과적으로 value 타입은 Lua 와 C# 사이에서 주고받을 때 C# gc alloc 발생하지 않으며 해당 타입 배열 접근시에도 GC 가 발생하지 않습니다. GC 에 자유로운 다양한 시나리오는 05\_NoGc 예제를 참고하시기 바랍니다.

열거형(enum)을 제외한 (파라미터 없는 생성자를 가지고 있는 복합 타입을 포함한) 모든 타입은 Lua 테이블을 생성하며 수정된 타입의 1차원 배열로 되어 있는 변환 코드를 생성합니다. 이로인해 gc alloc 이 감소하며 변환 성능이 좋아집니다.

### XLua.AdditionalProperties

AdditionalProperties 설정은 GCOptimize 의 확장 설정입니다. 필드(Field)를 프라이빗(private)로 만들고 프로퍼티(Property)를 통해 필드에 접근하게 하는 구조체에 필요한 설정입니다. 기본적으로 GCOptimize 는 퍼블릭(public) 필드(Field)만 패킷화/디패킷화합니다.

태깅 모드에서는 상대적으로 단순하게 지정할 수 있고 설정 모드에서는 복잡합니다. `Dictionary<Type, List<string>>` 타입을 필요로 하는데 Dictionary 의 Key 는 유효한 타입이고 Value 는 프로퍼티 리스트입니다. UnityEngine value 타입과 SysGCoptimize 타입에 대한 xLua 설정을 참고하시기 바랍니다.

### XLua.BlackList

BlackList 설정은 지정한 타입의 멤버 어댑터 코드를 생성하지 않으려고 할 때 사용합니다.

태깅 모드에서는 상대적으로 단순하게 지정할 수 있으며 대응되는 멤버를 추가할 수 있습니다.

제외 목록(BalckList)에 오버로드 함수들 중 하나만 추가해야 한다는 점까지 고려하면 설정은 좀 더 복잡해집니다. 타입은 `List<List<string>>` 이며 string 리스트를 가지고 있는 리스트입니다. 첫번째 문자열은 해당 타입의 전체 경로이고, 두번째 문자열은 멤버 이름입니다. 멤버가 메서드라면 세번째 문자열에는 파라미터 타입들의 전체 경로가 필요합니다.

예를 들어 다음은 GameObject 의 프로퍼티와 FileInfo 의 메소드를 제외 목록(BlackList)에 추가하는 방법입니다.

~~~csharp
[BlackList]
public static List<List<string>> BlackList = new List<List<string>>()  {
    new List<string>(){"UnityEngine.GameObject", "networkView"},
    //new List<string>(){ typeof(UnityEngine.GameObject).FullName, "networkView"},
    new List<string>(){"System.IO.FileInfo", "GetAccessControl", "System.Security.AccessControl.AccessControlSections"},
    //new List<string>(){ typeof(System.IO.FileInfo).FullName, "GetAccessControl",typeof(System.Security.AccessControl.AccessControlSections).FullName },
};
~~~

### 다음은 제너레이터 설정이며 반드시 에디터 디렉토리내 위치해야 합니다.

### CSObjectWrapEditor.GenPath

코드 생성 경로를 string 타입으로 설정합니다. 기본값은 `"Assets/XLua/Gen" 입니다.

### CSObjectWrapEditor.GenCodeMenu

이 설정은 빌드 엔진 2차 개발에 사용됩니다. 해당 태그를 파라미터 없는 함수에 추가하면 `"XLua/Generate COde" 메뉴를 실행할 때 호출됩니다.
