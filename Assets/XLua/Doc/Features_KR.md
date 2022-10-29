# 특징

## 개요

* Lua 가상 머신 지원
  * Lua-5.3
  * LuaJIT-2.1
* Unity3D 버전 지원
  * 모든 버전 지원중
* 플랫폼 지원
  * Windows 64/32
  * Android
  * iOS 64/32/bitcode
  * MacOS X
  * UWP
  * WebGL
* 주요 기술
  * 어댑터 코드 생성
  * 리플렉션(Reflection)
* 사용 편의
  * 압축 해제후 사용 가능
  * 코드 작성 중 빌드 필요 없음
  * 작성한 코드와 리플렉션 사이 매끄러운 전환
  * 단순한 GC 로부터 자유로운 API
  * 이해하기 쉬운 메뉴
  * 다중 복제, 모듈 분리, 타겟 타입에 Attribute 라벨을 붙일 수 있는 설정 제공
  * 코드 클리핑(Clipping) 방지를 위한 link.xml 자동 생성
  * 쉬운 사용을 위해 cmake 로 컴파일된 플러그인 파트
  * 코어 코드가 생성된 코드에 의존적이지 않아 언제든지 생성된 폴더 삭제 가능함
* 성능
  * 사용하지 않는 타입으로 인한 오버헤드를 줄이기 위한 지연 로딩 기술
  * Lua 함수는 C# 델리게이트, Lua 테이블은 인터페이스에 맵핑되어 있음. 인터페이스 레벨에서 C# GC alloc 오버헤드가 없음
  * 모든 기본 타입, 모든 열거형, 구조체 필드등을 Lua 에서 C# 으로 전달시 GC alloc 없음
  * LuaTable, LuaFunction 는 GC 접근 인터페이스를 통하지 않음
  * 정적 분석을 통해 최적화된 코드를 생성함
  * C# 과 Lua 사이 포인터 전달 지원
  * UnityEngine.Object 가 파괴될 경우 자동으로 레퍼런스 해제됨
* 확장
  * Lua 써드 파티 확장은 코드 변경없이 추가할 수 있음
  * 세컨더리 개발을 위한 엔진 생성 인터페이스 제공

## C# 구현 패치 제공

* 컨스트럭터(Constructor)
* 데스트럭터(Destructor)
* 멤버 메서드(MemberMethods)
* 스태틱 메서드(StaticMethods)
* 제너릭 메서드(GenericMethods)
* 오퍼레이터 오버로딩(OperatorOverloading)
* 멤버 프로퍼티(MemberProperties)
* 스태틱 프로퍼티(StaticProperties)
* 이벤트(event)

## Lua 코드 로딩

* string 로드
  * 로딩 후 즉시 실행 지원
  * 로딩 후 델리게이트(delegate)나 LuaFunction 리턴 지원. 델리게이트(delegate)나 LuaFunction 호출 후 스크립트 파라미터 전달 가능
* 리소스 디렉토리내 파일
  * 직접 require 가능
* 커스텀 로더
  * Lua 내 require 를 통해 작동
  * require 파라미터가 로더로 전달되면 로더는 Lua 코드를 읽고 리턴함
* Lua 오리지널 방식
  * Lua 기본 기본 작동 방식 그대로 실행됨

## Lua 에서 C# 호출

* C# 오브젝트(Object) 생성
* C# 스태틱 프로퍼티와 필드(StatisProperties & StaticFields)
* C# 스태틱 메서드(StatisMethods)
* C# 멤버 프로퍼티와 필드(MemberProperties & MemberFields)
* C# 멤버 메서드(MemberMethods)
* C# 상속
  * 서브 클래스 오브젝트는 직접적으로 부모 클래스 메서드를 호출하고 프로퍼티에 접근함
  * 서브 클래스 모듈은 직접적으로 부모 클래스의 스태틱 메서드(StaticMethods)와 스태틱 프로퍼티(StatisProperties)에 접근함
* 익스텐션 메서드(ExtensionMethods)
  * 일반적인 멤버 메서드(MemberMethods)와 동일하게 사용됨
* 파라미터 입출력 속성(out, ref)
  * out lua 리턴으로 처리됨
  * ref lua 파라미터와 리턴으로 처리됨
* 메서드 오버로딩(MethodOverloading)
  * 오버로딩(Overloading) 지원
  * Lua 데이터 타입은 C# 에 비해 훨씬 적기 때문에 판단이 어려워 확장 메소드(ExtensionMethods)로 호출될 수 있음
* 오퍼레이터 오버로딩(OperatorOverloading)
  * 지원 되는 오퍼레이터(Operator): +, -, *, /, ==, unary -, <, <=, %, []
  * 다른 오퍼레이터는 익스텐션 메서드(ExtensionMethods)를 통해 호출됨
* 디폴트 파라미터(default value
  * C# 파라미터는 기본 값을 가질 수 있으나 lua 에 전달되지는 않음
* 가변 파라미터(VariableParameters)
  * 가변 파라미터에 해당하는 부분은 직접 입력하면 되므로 확장할 필요는 없음
* 제너릭 메서드(GenericMethod) 호출
  * 스태틱 메서드(StaticMethods)는 그대로 캡슐화됨
  * 멤버 메서드(MemberMethods)는 익스텐션 메서드(ExtensionMethods)를 사용해 캡슐화됨
* 열거형(enum)
  * 숫자와 문자열 enum 변환 지원
* 델리게이트(delegate)
  * C# 델리게이트(delegate) 호출
  * `+` 오퍼레이터(operator) 지원
  * `-` 오퍼레이터(operator) 지원
  * C# 델리게이트(delegate) 처럼 Lua 함수를 C# 에 전달함
* 이벤트
  * 이벤트 콜백 추가
  * 이벤트 콜백 삭제
* 64 비트 정수
  * GC 와 정밀도 손상 없이 전달
  * Lua-5.3 네이티브 64 비트 지원
  * number 사용 가능
  * java 부호 없는 64 비트 정수 지원
* C# 복합 타입을 위한 테이블 자동 변환
  * obj.complexField = {a = 1, b = {c = 1}}, obj 는 a C# 오브젝트, complexField 는 2 레벨 중첩 struct 혹은 클래스
* typeof
  * C# typeof 대응. Type 오브젝트 리턴
* Lua 에서 직접 클론
* 부동 소수점(Decimal)
  * GC 와 정밀도 손상 없이 전달

## C# 에서 Lua 호출

* Lua 함수 호출
  * delegate 모드로 Lua 함수 호출
  * LuaFunction 을 사용해 Lua 함수 호출
* Lua 테이블 접근
  * LuaTable 일반적인 Get/Set 인터페이스 제공. GC 없이 호출, Key 와 Value 타입 정의 가능
  * CSharpCallLua 마킹시 인터페이스 접근
  * struct 과 class 에 값 복사

## Lua 가상 머신 (VirtualMachine)

* 가상 머신 GC 파라미터 읽기와 설정

## 툴 체인 (Toolchain)

* Lua 프로파일러(Profiler)
  * 함수 호출 전체 시간, 각 호출 평균 시간 호출 횟숫에 따라 정렬 가능
  * Lua 함수 이름과 함수가 위치한 파일내 라인 번호 표시
  * C# 함수 표시
* 리얼 머신 디버깅 지원
