---
title: Kotlin in action 06. 코틀린 타입 시스템
author: jaeeun
date: 2024-01-21 00:00:00 +0800
categories: [Study, "Kotlin"]
tags: ["Kotlin in action"]
render_with_liquid: false
---

# chapter06. 코틀린 타입 시스템

## 6.1 널 가능성

### 6.1.1 널이 될 수 있는 타입: `?`

코틀린 타입 시스템은 널이 될 수 있는 타입을 명시적으로 지원한다.

기본적으로 코틀린의 타입은 null 참조를 저장할 수 없다.

#### 널 허용 타입 만들기
**널과 명시된 타입을 인자**로 받을 수 있게 하려면 **타입 뒤에 물음표(?)를 명시**해야 한다.

널이 될 수 있는 타입의 변수가 있다면 그에 대해 수행할 수 있는 연산이 제한되며

해당 타입은 **널 이 아님을 확신할 수 있는 영역에서
널이 될 수 없는 타입의 값처럼 사용**할 수 있다.

```kotlin
fun strLenSafe(s: String?) : Int = if (s != null) s.length else 0
```

### 6.1.3 안전한 호출 연산자: `?.`

호출 연산자 `?.`는 **null 체크**에 대해서 유용하다.
이 연산자는 객체가 null이 아닌 경우에만 해당 메서드나 프로퍼티에 접근하도록 도와준다.
- 호출하려는 값이 **null이 아니라면** ?.은 **일반 메서드 호출**처럼 동작
- 호출하려는 값이 **null이면** 호출은 무시되고 **null을 반환**한다.

```kotlin
fun strLen(s: String?) : Int? = s?.length
```
```
                --- foo != null ---> foo.strLen()
foo?.strLen() -|
                --- foo == null ---> null
```

### 6.1.4 엘비스 연산자 `?:`

엘비스 연산자 `?:`는 **null 대신 사용할 디폴트 값을 지정**할 때 사용할 수 있다.

```kotlin
fun strLenSafe(s: String?) : Int = s?.length ?: 0
```

### 6.1.5 안전한 캐스트 `as?`

`as?` 연산자는 어떤 **값을 지정한 타입으로 캐스트**한다. as?는 값을 대상 타입으로 반환할 수 없으면 null을 반환한다.
```kotlin
val stringValue: Any = "Hello"

// 안전한 형변환 시도
val castedString: String? = stringValue as? String

// 엘비스 연산자를 사용하여 출력
val message = castedString ?: "Casting failed."
println("Casting result: $message")
```

### 6.1.6 널 아님 단언 `!!`
`!!` 연산자를 사용하여 널이 될 수 있는 타입을 **널이 될 수 없는 타입으로 강제 변환**할 수 있다.

그러나 사용에 주의해야 하며, 만약 해당 변수가 실제로 **널이라면 NullPointerException**이 발생할 수 있다.

```kotlin
val nullableValue: String? = "Hello"
val nonNullValue: String = nullableValue!!
```

### 6.1.7 `let` 함수

`let` 함수를 사용하여 널이 될 수 있는 값에 대해 안전하게 연산을 수행할 수 있다.

let 함수는 람다 함수를 호출하면서 수신 객체를 람다의 인자로 전달한다.
이를 통해 널 체크를 진행하고, 널이 아닐 때만 람다 블록을 실행할 수 있다.

```kotlin
val nullableValue: String? = "Hello"

// let 함수를 사용하여 널 체크 및 안전한 블록 실행
nullableValue?.let { 
    println("The value is not null: $it")
}
```

### 6.1.8 나중에 초기화 `lateinit`

`lateinit` 변경자를 붙여 프로퍼티를 나중에 초기화할 수 있다.

lateinit은 주로 non-null 타입의 프로퍼티를 선언할 때 사용되며, 초기화를 나중에 수행할 수 있도록 한다.

> 의존성 주입 시 유용하게 사용된다. 그런 시나리오에서는 lateinit 프로퍼티 값을 DI 프레임워크가 외부에서 설정해준다.

```kotlin
@SpringBootTest
class Example {

    @Autowired lateinit var testHelper: TestHelper
}
```

### 6.1.9 널이 될 수 있는 타입 확장

널이 될 수 있는 타입에 대한 확장 함수를 정의하여 해당 타입의 변수에 대해 메서드를 호출할 때, 확장 함수가 널을 처리하게 할 수 있다.

```kotlin
// String? 타입에 대한 확장 함수 정의
fun String?.safeUpperCase(): String {
    // 널 체크 후 안전하게 대문자로 변환
    return this?.toUpperCase() ?: "NULL"
}

fun main() {
    // 널이 될 수 있는 String 타입의 변수
    val nullableString: String? = "hello"
    
    // 확장 함수를 호출하면서 널 안전성을 활용
    val result = nullableString.safeUpperCase()
    
    println(result) // 출력: HELLO

    // 널이 될 수 있는 변수에 대해서도 확장 함수 호출이 가능하며, 널을 처리할 수 있다.
    val nullValue: String? = null
    val resultNull = nullValue.safeUpperCase()
    
    println(resultNull) // 출력: NULL
}
```

코틀린에서는 String을 확장해 정의된 `isEmpty`와 `isBlank`가 있다.
- isEmpty : 빈 문자열 검사
- isBlank : 공백 문자로만 구성되어 있는지 확인

또한 String? 타입의 수신 객체에 대해 호출할 수 있는 `isNullOrEmpty`, `isNullOrBlank` 메서드가 있다.
- isNullOrEmpty : 문자열이 null이거나 비어 있는지 확인
- isNullOrBlank : 문자열이 null이거나 비어 있거나 공백 문자로만 구성되어 있는지 확인

### 6.1.10 타입 파라미터의 널 가능성
타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 **이름 끝에 물음표가 없더라도 T가 널이 될 수 있는 타입**이다.

만약 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한을 지정해야 한다.

### 6.1.11 널 가능성과 자바
코틀린에서 플랫폼 타입(Platform Type)은 널 관련 정보를 알 수 없는 타입을 나타낸다.

이는 자바와의 상호 운용성을 감안하여 도입된 개념이다.

자바에서는 모든 참조 타입이 기본적으로 널이 될 수 있기 때문에, 코틀린에서 자바 타입을 사용할 때 널 관련 정보를 정확하게 알 수 없다.
이런 경우에는 코틀린 컴파일러는 널 처리를 강제하지 않는다.

```
⌜---------⌝   ⌜----------------⌝
|  Type   | = | Type? 또는 Type |
⌞-- 자바 --⌟   ⌞----- 코틀린 -----⌟
```

## 6.2 코틀린의 원시 타입

코틀린은 **원시 타입과 래퍼 타입을 구분하지 않으며** 실행 시점에 숫자 타입은 가능한 한 가장 효율적인 방식으로 표현된다.

코틀린의 숫자 타입 (예: Int, Long, Double 등)은 **가능한 경우에는 자바의 기본 타입에 해당하는 타입으로 컴파일**된다.
예를 들어, 대부분의 경우에는 Int는 자바의 int로, Long은 자바의 long으로 컴파일된다.

그러나 코틀린은 **제네릭과 같은 특정 상황**에서는 기본 타입을 사용할 수 없으며, 이때는 **해당 타입의 래퍼 타입에 해당하는 객체가 생성**된다.
예를 들어, 코틀린의 List<Int>는 자바에서는 List<Integer>로 컴파일된다.

### 6.2.3 숫자 변환

코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다.
```kotlin
val i = 1
val l: Long = i // Error: type: missmatch 컴파일 오류 발생

// 직접 변환 메서드 호출
val l: Long = i.toLong()
```

### 6.2.4 `Any`, `Any?` : 최상위 타입

Any: 모든 타입의 조상이며, 모든 타입의 값을 가질 수 있는 최상위 타입이다.

Any?: Any와 달리 널(null)도 허용하는 최상위 타입이다.

### 6.2.5 `Unit` 타입 : 코틀린의 void

Unit 타입은 코틀린에서의 반환 값이 없음을 나타내는 타입으로, 자바의 void와 같은 역할을 한다.

함수가 어떠한 값을 반환하지 않을 때, 해당 함수의 반환 타입으로 Unit을 사용하며 생략 가능하다.

### 6.2.6 Nothing 타입 : 이 함수는 정상적으로 끝나지 않는다.

Nothing 타입은 아무 값도 포함하지 않으므로 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

## 6.3 컬렉션과 배열

### 6.3.2 읽기 전용과 변경 가능한 컬렉션

코틀린에서는 **컬렉션 안의 데이터에 접근**하는 인터페이스(Immutable)와 **컬렉션 안의 데이터를 변경**하는 인터페이스(Mutable)를 분리하여 제공한다.

- **읽기 전용 인터페이스**: Collection, List, Set, Map ...
- **변경 가능 인터페이스**: MutableCollection, MutableList, MutableSet, MutableMap ...

이 때 어떤 동일한 컬렉션 객체를 가리키는 읽기 전용 컬렉션 타입의 참조와 변경 가능한 컬렉션 타입의 참조가 있는 경우
이런 상황에서는 읽기 전용 컬렉션이 스레드 안전하지는 않다.

[읽기 전용 컬렉션의 thread unsafe 예제](./code/ThreadUnsafeCollection.kt)

```
+------------------------------------------------+
|               sharedList                       |
|              +---+---+---+                     |
|              | 1 | 2 | 3 |                     |
|              +---+---+---+                     |
|             ▲            ▲                     |
|             |            |                     |
|  +-----------+        +---------------------+  |
|  | List<Int> |        | MutableList<String> |  | 
|  +-----------+        +---------------------+  |
|                                                |
+------------------------------------------------+
```
