![](Assets/XLua/Doc/xLua.png)

[![license](http://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/Tencent/xLua/blob/master/LICENSE.TXT)
[![release](https://img.shields.io/badge/release-v2.1.15-blue.svg)](https://github.com/Tencent/xLua/releases)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blue.svg)](https://github.com/Tencent/xLua/pulls)
[![Build status](https://travis-ci.org/Tencent/xLua.svg?branch=master)](https://travis-ci.org/Tencent/xLua)

[(English Documents Available)](README_EN.md)

## C# 을 위한 Lua 프로그래밍 솔루션

xLua 는 Unity, .Net, Mono 및 기타 C# 환경에 Lua 스크립트 기능을 추가합니다. xLua 와 함께라면 Lua 와 C# 코드를 쉽게 연동할 수 있습니다.

## xLua 의 뛰어난 기능

xLua 는 기능, 성능, 편의성에 있어 많은 비약적인 발전을 이루었습니다. 주요한 기능들은 다음과 같습니다.

* C# 구현(메소드, 오퍼레이터, 프로퍼티, 이벤트 등)을 런타임상에서 Lua 구현으로 대체할 수 있습니다.
* C# 과 Lua 사이 오브젝트들을 주고 받을 때 GC 최적화, 맞춤형 구조, C# 가비지 컬렉팅이 없습니다. 
* 에디터 모드에서 코드를 작성하지 않아도 되는 가벼운 개발 방식을 제공합니다.

좀 더 자세한 기능에 대해 알고 싶다면 [특징](Assets/XLua/Doc/Features_KR.md)문서를 참고해주세요.

## 설치

zip 패키지를 언팩하면 Unity 프로젝트 Assets 디렉토리를 확인할 수 있습니다. 여러분의 유니티 프로젝트에도 동일한 디렉토리 구조로 넣어주시면 됩니다.

다른 디렉토리에 설치하고 싶다면 [FAQ](Assets/XLua/Doc/Faq_KR.md)를 참고 부탁 드립니다.

## 문서

* [FAQ](Assets/XLua/Doc/Faq_KR.md): 자주 문의되는 질문들에 대한 답변을 정리해놓았습니다. 처음 시작할 때 생기는 궁금한 점들을 찾아볼 수 있습니다.
* (필독!) [XLua 튜토리얼](Assets/XLua/Doc/XLua_Tutorial_KR.md): 튜토리얼입니다. 참고할만한 코드는 [여기](Assets/XLua/Tutorial/)에서 찾아보실 수 있습니다.
* (필독!) [XLua 설정](Assets/XLua/Doc/Configure_KR.md): xLua 를 설정하는 방법입니다.
* [핫픽스 가이드](Assets/XLua/Doc/Hotfix_EN.md): 핫픽스 기능 설명서입니다.
* [xLua 써드 파티 Lua 라이브러리 추가 및 삭제](Assets/XLua/Doc/Add_Remove_Lua_Lib.md): 써드 파티 Lua 확장 라이브러리 추가나 삭제하는 방법입니다.
* [xLua API](Assets/XLua/Doc/XLua_API_EN.md): API 매뉴얼입니다.
* [빌드 엔진 세컨더리 개발](Assets/XLua/Doc/Custom_Generate_EN.md): 빌드 엔진의 세컨더리 개발 방법입니다.

## 빠른 시작

단 3 줄 코드로 구성된 완벽한 예제입니다.

xLua 를 설치하고, MonoBehaviour 를 생성한 다음 아래 코드를 작성해보세요.

```csharp
XLua.LuaEnv luaenv = new XLua.LuaEnv();
luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
luaenv.Dispose();
```

1. DoString 파라미터는 string 이며 어떤 Lua 코드도 넣어 볼 수 있습니다. 이번 예제에서는 로그를 출력하기 위해 Lua 에서 C# 의 UnityEngine.Debug.Log 를 호출했습니다.

2. LuaEnv 인스턴스는 Lua 가상 머신입니다. 오버헤드 이슈로 Lua 가상 머신은 전역적으로 단일하게 유지하는 것을 추천합니다.

C# 에서 Lua 를 호출하는 방법도 간단합니다. Lua 시스템 함수를 호출하는 괜찮은 방법을 예를 들어보겠습니다.

* 선언

```csharp
[XLua.CSharpCallLua]
public delegate double LuaMax(double a, double b);
```

* 연동

```csharp
var max = luaenv.Global.GetInPath<LuaMax>("math.max");
```

* 호출

```csharp
Debug.Log("max:" + max(32, 12));
```

한번 연동해둔 다음 재사용하는 것을 추천합니다. 코드를 생성해두면 max 를 호출할 때 gc alloc 을 하지 않습니다.

## 핫픽스

* 오래된 코드의 원본 코드를 어떤 변경도 없이 핫픽스를 적용할 수 있습니다.
* 런타임에 거의 영향을 주지 않으며 핫픽스를 적용하지 않은 코드와 거의 동일하게 작동합니다.
* 문제가 발생했을 때 Lua 코드 패치를 사용할 수 있습니다. 이후 Lua 코드 로직이 작동하게 됩니다.

사용 방법은 [핫픽스](Assets/XLua/Doc/Hotfix_EN.md)를 참고하세요.

## 추가 예제

* [01_Helloworld](Assets/XLua/Examples/01_Helloworld/): 빠른 시작 예제입니다.
* [02_U3DScripting](Assets/XLua/Examples/02_U3DScripting/): MonoBehaviour 를 작성해 Mono 를 사용하는 예제입니다.
* [03_UIEvent](Assets/XLua/Examples/03_UIEvent/): Lua 를 사용해 UI 로직을 작성하는 예제입니다. 
* [04_LuaObjectOrented](Assets/XLua/Examples/04_LuaObjectOrented/): Lua 와 C# 사이 OOP 를 연동하는 예제입니다.
* [05_NoGc](Assets/XLua/Examples/05_NoGc/): value 타입 GC 를 피하는 예제입니다.
* [06_Coroutine](Assets/XLua/Examples/06_Coroutine/): Unity 코루틴을 사용해 Lua 코루틴을 작동시키는 예제입니다.
* [07_AsyncTest](Assets/XLua/Examples/07_AsyncTest/): Lua 코루틴을 사용해 비동기 로직을 동기화하는 예제입니다.
* [08_Hotfix](Assets/XLua/Examples/08_Hotfix/): 핫픽스 예제입니다. (자세한 사항은 [핫픽스](Assets/XLua/Doc/Hotfix_EN.md)를 참고해주세요.)
* [09_GenericMethod](Assets/XLua/Examples/09_GenericMethod/): 제너리 함수에 대한 데모입니다.
* [10_SignatureLoader](Assets/XLua/Examples/10_SignatureLoader/): 전자 서명이 있는 Lua 스크립트를 읽는 예제입니다. 자세한 사항은 [전자 서명](Assets/XLua/Doc/signature.md)을 참고해주세요.
* [11_RawObject](Assets/XLua/Examples/11_RawObject/): C# 파라미터가 object 일 때 boxing 되어 있는 Lua 숫자를 전달하는 예제입니다.
* [12_ReImplementInLua](Assets/XLua/Examples/12_ReImplementInLua/): Lua 구현에 복잡한 값을 변경해보는 예제입니다.

## 기술 지원 

QQ 그룹 1: 612705778 (만료)

QQ 그룹 2: 703073338 (만료)

QQ 그룹 3: 811246782

문제 상황을 발견했다면 [FAQ](Assets/XLua/Doc/faq_EN.md) 를 먼저 확인해주세요.

