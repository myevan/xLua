# xLua 튜토리얼

## Lua 실행하기

1. Lua 문자열 실행

    가장 간단한 방법은 LuaEnv.DoString 을 사용해 문자열을 실행하는 것입니다. 문자열은 Lua 문법에 맞아야 합니다.

    예시:

       luaenv.DoString("print('hello world')")

    XLua\Tutorial\LoadLuaScript\ByString 디렉토리내 전체 코드를 확인해보세요.

    > 하지만 문자열 실행 모드보다는 이후 나오는 방식들을 추천합니다.

2. Lua 파일 불러오기

    Lua require 함수를 사용합니다.

       DoString("require 'byfile'")

    XLua\Tutorial\LoadLuaScript\ByFile 디렉토리내 전체 코드를 확인해보세요.

   The require actually calls the loaders one by one to load the file. The loading process stops if one loader succeeds. If all loaders fail, no file found will be reported. 

    require 는 파일을 불러오기 위해 하나씩 로더를 호출합니다. 로딩 프로세스는 로더가 하나씩 성공할 때마다 종료됩니다. 모든 로더가 실패하면 어떤 파일도 찾을 수 없다고 리포트 됩니다.

    xLua 는 네이티브 로더 이외 리소스 로더를 추가합니다. 다만 리소스 로더는 제한된 확장자만 지원하므로 Resources 안에 있는 Lua 파일은 .txt 확장자를 붙여줘야 합니다. (첨부된 예제를 확인해주세요.)

    Lua 스크립트를 불러오는 추천 방식입니다: 전체 프로그램이 `DoString("require 'main'")`인지 확인한 다음 (커맨드 라인에서 Lua 스크립트를 실행하는 방법: lua main.lua 와 동일하게) main.lua 에서 다른 스크립트들을 불러옵니다.

    몇몇 분들은 다운로드 받은 Lua 파일, 커스텀된 Lua 파일, 암호화된 Lua 파일을 실행하기 위한 방법을 문의 주셨습니다. xLua 커스텀 로더는 다양한 상황에 대응할 수 있습니다.

3. 커스텀 로더

   xLua 에서는 로더를 간단하게 커스터마이즈 할 수 있습니다. 관련된 인터페이스는 단 1 개뿐입니다:

       public delegate byte[] CustomLoader(ref string filepath);
       public void LuaEnv.AddLoader(CustomLoader loader)

    AddLoader 를 사용하면 콜백을 등록할 수 있습니다. 파라미터는 string 이며 Lua 코드에서 require 가 호출될 때 그대로 전달 됩니다. 콜백은 전달 받은 파라미터에 따라 지정한 파일들을 로드하게 됩니다. 디버깅이 필요할 경우 파일 경로를 실제 경로로 수정해서 전달해야 합니다. 콜백의 리턴값은 바이트 배열로 Null 은 로딩에 실패했다는 것을 의미합니다. 성공했다면 Lua 파일 내용을 리턴하게 됩니다.

IIPS IFS 를 로드할 수 있을까요? 네, 로더가 IIPS 인터페이스를 통해 파일 내용을 읽어드리도록 작성하시면 됩니다.

암호화된 파일을 읽을 수 있을까요? 네, 로더가 파읽을 읽고 복호화 후 리턴하면 됩니다.

 XLua\Tutorial\LoadLuaScript\Loader 디렉토리에 있는 예제를 참고하시기 바랍니다.

## C# 에서 Lua 접근하기

C# 에서 Lua 데이터 구조를 다루는 방법을 시작해보도록 하겠습니다.
이번 챕터에서 설명하는 예제들은 XLua\Tutorial\CSharpCallLua 디렉토리에서 찾아 볼 수 있습니다.

1. 전역 기본 자료형(GlobalBasicDataType) 얻기

    LuaEnv.Global 에서 `GEt<T>` 메서드를 사용하면 리턴 받는 유형을 정할 수 있습니다.

        luaenv.Global.Get<int>("a")
        luaenv.Global.Get<string>("b")
        luaenv.Global.Get<bool>("c")

2. 전역 테이블(GlobalTable) 접근

    위에 나온 Get 메서드를 사용할 경우 어떤 클래스를 지정해야할까요?

    1. 일반 클래스나 구조체에 맵핑됩니다.

        테이블에 대응하는 필드의 퍼블리 프로퍼티가 있는 클래스를 정의합니다. 파라미터 생성자가 있던 없던 상관 없습니다. 예를 들어 `{f1 = 100, f2 = 100}`이라면 `public int f1; public int f2;`를 포함하는 클래스를 정의합니다. xLua 는 새로운 인스턴스를 만들어서 필드에 대응하는데 도움을 줄 수 있습니다.

        테이블 프로퍼티들은 클래스 프로퍼티보다 많거나 적을 수 있습니다. 다른 복합 클래스(ComplexClass)들을 중첩시킬 수 도 있습니다. 값들을 복사하는 프로세스이기 때문에 복합 클래스(ComplexClass)들은 비용이 꽤 큽니다. 필드 값을 수정하더라도 테이블과 동기화 되지 않으며 반대 상황도 동일합니다.

        이 기능은 GCOptimize 에 클래스를 추가하면 생성 비용을 줄일 수 있습니다. 자세한 사항은 설정 문서를 참고하시기 바랍니다.

        레퍼런스 모드에서 맵핑을 사용할 수 있을까요? 네, 아래에서 설명할 방법 중 하나를 사용할 수 있습니다.

    2. 인터페이스에 맵핑

        인터페이스 맵핑 모드는 생성된 코드에 의존합니다. (코드가 생성되지 않았다면 InvalidCastException 에러가 리포트 됩니다.) 코드 제너레이터는 인터페이스 인스턴스를 생성합니다. 프로퍼티를 얻을려고 하면 대응되는 테이블 필드를 가져오는 코드를 생성합니다. 마찬가지로 프로퍼티를 설정할 때는 대응되는 테이블 필드를 설정하게 됩니다. 인터페이스를 통해 Lua 함수에 억세스 할 수도 있습니다.

    3. 값 모드 경량화: Dictionary<> 와 List<>에 맵핑

        타입이나 인터페이스를 정의하지 않은 경우 사용할 수 있습니다. 전제는 테이블 키와 값이 같은 유형이어야 합니다.

    4. ref 모드 작동 방법: LuaTable 클래스에 맵핑

        ref 모드의 장점은 코드를 생성할 필요가 없다는 점입니다. 다만 몇가지 문제가 있는데 예를 들면 인터페이스보다 매우 많이 느리며 클래스 체크를 하지 않습니다.

3. 전역 함수(GlobalFunction) 접근

    전역 함수 접근 역시 Get 메서드를 사용하지만 다른 클래스에 맵핑됩니다.

   1. delegate 에 맵핑하기

        성능과 안정성 면에서 가장 추천되는 방식입니다. 이 메서드의 단점은 생성된 코드라는 점입니다. (아무런 코드가 생성되지 않는다면 InvalidCastException 에러가 리포트됩니다.)

        delegate 를 어떻게 선언하나요?
        함수 각 파라미터에 입력 타입 파라미터를 선언합니다.

        여러 개 리턴 값은 어떻게 처리하나요?
        C# 출력 파라미터들을 왼쪽에서 오른쪽으로 맵핑합니다. 출력 파라미터들에는 리턴 값, out 파라미터, ref 파라미터가 포 포함됩니다.

        파라미터와 리턴 값은 어떤 타입들이 지원 되나요?
        모든 타입이 지원 됩니다. 다양한 복합 타입(ComplexType)들이 리턴 될 수 있습니다. out, ref, 심지어 다른 delegate 까지 리턴할 수 있습니다.

        delegate 를 사용하는 것이 가장 간단하며, 함수처럼 사용할 수 있습니다.

   2. LuaFunction 에 맵핑하기

        LuaFunction 에 맵핑하는 방법은 delegate 에 맵핑하는 방법과 장단점이 안전히 반대입니다.

        사용법은 간단합니다. LuaFunction 에는 가변인자를 가진 Call 함수가 있으며 어떤 타입이든 몇개던지 전달할 수 있습니다. 리턴 값은 object 배열로 Lua 에서 리턴되는 여러 값들입니다.

4. 추천 사용 방법

    1. Lua 글로벌 데이터, 특히 테이블과 함수에 접근하는 비용은 매우 높습니다. 우리는 가능하면 최소화 할 것을 추천합니다. 에를 들어 초기화 동안 나중에 (delegate 로 맵핑해서) 호출할 함수를 가져와 저장해 둔 다음 delegate 를 통해 직접적으로 호출할 수 있습니다. (테이블도 비슷합니다.)

    2. Lua 의 모든 구현이 delegate 와 interface 모드인 경우 xLua 에서 완전히 분리될 수 있습니다. 분리된 모듈은 xLua 초기화와 delegate 와 interface 맵핑을 담당합니다. 맵핑된 delegate 와 interface 들은 사용할 곳에 위치시킬 수 있습니다.

### Lua 에서 C# 호출하기

> 이번 섹션에서 다루는 모든 예제는 XLua\Tutorial\LuaCallCSharp 에 들어 있습니다.

#### 새로운 C# 오브젝트 만들기

새로운 C# 오브젝트를 생성할 수 있습니다.

    ```c#
    var newGameObj = new UnityEngine.GameObject();
    ```

Lua 에서 C# 오브젝트를 생성하는 방법입니다.

    ```lua
    local newGameObj = CS.UnityEngine.GameObject()
    ```

몇 가지 경우를 제외하고 동일합니다.

    1. Lua 에는 new 키워드가 없습니다.
    2. C# 과 관련된 모든 컨텐츠들은 CS 테이블에 위치합니다. 생성자, 스태틱 멤버 프로퍼티와 메서드들이 들어 있습니다.

다양한 생성자에 대응하려면 어떻게 해야 하나요? 문제 없습니;다. xLua 는 오버로드를 지원합니다. 예를 들어 만약 1개의 string 파라미터를 가진 GameObject 생성자를 호출한다면 아래처럼 작성할 수 있습니다.

    ```lua
    local newGameObj2 = CS.UnityEngine.GameObject('helloworld')
    ```

#### C# 스태틱 프로퍼티와 메서드 접근하기

##### 스태틱 프로퍼티 읽기

    ```lua
    CS.UnityEngine.Time.deltaTime
    ```

##### 스태틱 프로퍼티 쓰기

    ```lua
    CS.UnityEngine.Time.timeScale = 0.5
    ```

##### 스태틱 메서드 호츨

    ```lua
    CS.UnityEngine.GameObject.Find('helloworld')
    ```

팁: 자주 접근하는 타입에 대해서 호출하기 전에 로컬 변수에 레퍼런스해둔다면 타이핑 시간을 줄이고 성능을 개선할 수 있습니다.

    ```lua
    local GameObject = CS.UnityEngine.GameObject
    GameObject.Find('helloworld')
    ```

#### C# 멤버 프로퍼티와 메서드 접근하기

##### 멤버 프로퍼티 읽기

    ```lua
    testobj.DMF
    ```

##### 멤버 프로퍼티 쓰기

    ```lua
    testobj.DMF = 1024
    ```

##### 멤버 메서드 호출

알림: 멤버 메서드를 호출할 때 첫번째 파라미터는 현재 오브젝트를 리턴하게 됩니다. 우리는 다음과 같은 `:` 이라는 문법적 설탕(syntactic sugar) 사용을 추천합니다:

    ```lua
    testobj:DMFunc()
    ```

##### 부모 프로퍼티와 메서드

XLua 는 (derived 타입을 통해) base 타입의 스태틱 프로퍼티와 스태틱 메서드 접근과 (derived 타입 인스턴스를 통해) base 타입의 멤버 프로퍼티와 멤버 메서드를 접근을 지원합니다.

##### 파라미터 입력과 출력 속성(out 과 ref)

Lua 호출 파라미터 규칙: C# 의 보통 파라미터는 입력 파라미터입니다. ref 는 입력 파라미터이지만 out 은 아닙니다. 나머지는 Lua 에서 호출되는 실제 파라미터 목록에 왼쪽부터 오른쪽 순서로 대응됩니다.

Lua 호출 리턴 규칙: C# 함수의(리턴 값이 존재할 경우) 리턴 값을 리턴합니다. out 도 리턴 값이고 ref 도 리턴 값입니다. 나머지는 루아에서 리턴된 값들이 왼쪽에서 오른쪽 순서로 대응됩니다.

##### 메서드 오버로드

예를 들어 다음과 같은 방식으로 다른 파라미터 타입들로 오버로드된 함수에 접근할 수 있습니다.

    ```lua
    testobj:TestFunc(100)
    testobj:TestFunc('hello')
    ```

TestFunc 의 int 파라미터와 string 파라미터는 구분되서 접근됩니다.

알림: xLua 오버로드는 범위가 한정됩니다. Lua 는 C# 에 비해 지원되는 타입이 적기 때문에 일대다 상황이 발생합니다. 예를 들어 C# 의 int, float, double 타입은 Lua 의 number 타입에 대등됩니다. 예시에서 나온 TestFunc 에서는 이런 타입들을 구분할 수 없으며 이들 중 1개(생성된 코드 중 첫번째)만 호출할 수 있습니다.

##### 오퍼레이터

지원되는 오퍼레이터: `+, -, *, /, ==, unary-, <, <=, %[]`

##### 디폴트 값이 있는 파라미터 메서드

C# 이 디폴트 값이 있는 함수를 호출하는 것과 동일합니다. 주어진 실제 파라미터가 정식 파라미터 숫자보다 적을 경우 디폴트 값이 추가됩니다.

##### params 변수 메서드

C# 파라미터

    ```C#
    void VariableParamsFunc(int a, params string[] strs)
    ```

Lua 호출 방법

    ```Lua
    testobj:VariableParamsFunc(5, 'hello', 'john')
    ```

##### 확장 메서드 사용하기

C# 에 정의하면 Lua 에서 사용할 수 있습니다.

##### 제너릭 (템플릿) 메서드

제너릭 메서드를 직접적으로 지원하지는 않지만 확장 메서드 기능을 통해 패키징하면 호출 할 수 있습니다.

##### 열거형

열거형 값은 열거형 스태틱 프로퍼티와 동일합니다.

    ```lua
    testobj:EnumTestFunc(CS.Tutorial.TestEnum.E1)
    ```

EnumTestFunc 함수 파라미터는 Tutorial.TestEnum 타입입니다.

Enum 은 __CastFrom 메서드를 가지고 있습니다. 정수나 문자열 값을 열거형으로 변환해줍니다.

    ```lua
    CS.Tutorial.TestEnum.__CastFrom(1)
    CS.Tutorial.TestEnum.__CastFrom('E1')
    ```

##### delegate 사용 (call, +, -)

C# delegate 호출: lUa 함수를 호출하는 것과 동일합니다.

`+` 오퍼레이터: C# + 오퍼레이터에 대응합니다.. 2개 호출을 콜 체인으로 결합하며 오른쪽 오퍼랜드는 동일한 탈입의 C# delegate 거나 Lua 함수여야 합니다.

`-` 오퍼레이터: + 와 반대로 콜 체인에서 delegate 를 삭제합니다.

> PS: delegate 프로퍼티는 Lua 함수에서 설정할 수 있습니다.

##### 이벤트

예를 들어 testobj 에 event 정의가 있다면: `public event Action TestEvent;`

이벤트 콜백 추가

    ```lua
    testobj:TestEvent('+', lua_event_callback)
    ```

이벤트 콜백 삭제

    ```lua
    testobj:TestEvent('-', lua_event_callback)
    ```

##### 64 비트 정수 지원

    Lua 5.3 에서 64 비트 정수(long, ulong)은 네이티브 64 비트 정수에 맵핑됩니다. LuaJIT 버전(Lua 5.1 버전도 동일)은 64 비트 정수를 지원하지 않지만 xLua 는 64bit 지원 확장 라이브러리를 제공합니다. C# long 과 ulong 정수들은 userdata 에 맵핑됩니다.

    64 비트 연산, 비교, print 지원, Lua number 연산은 Lua 에서 지원 합니다. 64 비트 확장 라이브러리에서는 비교만 지원되는데, int64 와 ulong 정수 타입은 long 타입으로 강제 변환된 후 Lua 로 전송됩니다. 몇몇 ulong 연산들은 비교를 위해 Java 와 동일한 모드를 사용할 수 있습니다: API 세트가 제공되며 자세한 사항은 API 문서를 참고해주시기 바랍니다.

##### C# 복합 타입(ComplexType)과 테이블 사이 자동 변환

파라미터가 없는 생성자를 가진 C# 타입은 Lua 테이블로 대체됩니다. 테이블은 복합 타입(ComplexType)의 퍼블릭 필드(PublicField)로 대응됩니다. 함수 파라미터 전달, 프로퍼티 대입 등을 지원합니다.

예시: C# B 구조체 정의(타입 지원)

    ```C#
    public struct A
    {
    public int a;
    }
    
    public struct B
    {
    public A b;
    public double c;
    }
    ```

A 타입은 멤버 함수를 가지고 있습니다.

    ```C #
    void Foo(B b)
    ```

Lua 는 B 타입 파라미터를 아래처럼 호출하게 됩니다.

    ```lua
    obj:Foo({b = {a = 100}, c = 200})
    ```

##### Get 타입 (C# 의 typeof 와 동일)

예를 들어 다음과 같은 방식으로 UnityEngine.ParticleSystem 타입 정보를 가져올 수 있습니다.

##### "strong" 변환

Lua 는 typed language 가 아니기 때문에 strongly typed languaged 처럼 "strong" 변환이 없습니다.
다만 비슷하게 사용 가능한 방법이 있습니다: xLua 가 생성된 코드로 object 를 호출하면 됩니다.
어떤 상황에 사용할 수 있을까요? 경우에 따라 써드 파티 라이브러리들은 interface 나 abstract 타입을 노출하는 경우가 있습니다. 구현이 숨겨져 있기 때문에 구현 타입에 대한 코드를 생성할 수 없습니다. xLua 는 이러한 구현 타입을 생성되지 않은 코드로 식별하며 리플렉션을 통해 접근하게 됩니다. 이런 타입들이 자주 호출될 경우 성능에 큰 영향을 미치게 됩니다. interface 나 abstract 타입을 생성된 코드에 추가해 오브젝트 접근시 사용할 수 있습니다:

    ```lua
    cast(calc, typeof(CS.Tutorial.Calc))
    ```

위 예시에서는 CS.Tutorial.Calc 생성된 코드를 사용해 calc 오브젝트에 접근하는 방법입니다.