---
layout: post
title:  "Swift - struct, class 차이점"
date:   2019-09-09 15:13:20 +0900
categories: Swift
background: '/img/posts/05.jpg'
---

`struct` 와 `class` 의 차이점을 알기위해 검색하던 중  
차이점이 아닌 공통점은 무엇인가 문득 궁금증이 생겼다.  

`struct` 와 `class` 의 차이점을 설명하기 전에  
공통점에 대해 간략하게 작성하고 넘어가자  

`struct` 와 `class`는 `swift` 에서 이러한 공통점을 가지고 있다.  

* 값을 저장하는 `property` 정의 할 수 있다.
* 기능을 제공하는 `method` 를 정의 할 수 있다.
* `subscribe` 구문을 사용하여 값에 접근 할 수 있는 `subscribe` 을 정의 할 수 있다.
* 초기 상태를 설정하는 `initialize` 함수를 정의 할 수 있다.
* 기본적인 구현을 넘어, 기능을 확장 할 수 있는 `extension` 을 구현 할 수 있다.
* `protocol` 상속이 가능하다.


### 차이점

* `struct`
    * `value type`, `call by value`: 변수 할당 및 `parameter` 전달시 `value copy` 가 일어난다.
    * `stack memory` 할당: 속도가 빠르다.
        * `scope based lifetime`: 컴파일 시점에 컴파일러가 언제 메모리를 할당 / 해제 해야하는지 정확히 알고 있다.(`reference counting` 사용 안함.)
    * `value copy` 구조로 `multi-thread` 환경에서 공유 자원으로 인한 이슈 발생 확율이 낮다.
    * 상속이 불가능하다.(`protocol`은 사용 가능)
    * `deinitializer` 구현 불가능
    * `Type Casting` 불가능
* `class`
    * `reference type`, `call by reference`: 변수 할당 및 `paramter` 전달시 메모리 주소 값만 복사된다.
    * `heap memory` 할당: 속도가 느리다.
        * 런타임에 직접 `alloc` 하며 `reference counting` 을 통해 `dealloc` 이 필요하다.
    * 상속이 가능하다.
    * `Type Casting` 이 가능하다.
    * `deinitializer` 구현이 가능하다.

__*`class` 안에 `struct` 변수를 `property` 로 정의하는것도 가능하고, 반대로 `struct` 의 `property` 로 `class` 인스턴스 변수를 갖는것도 가능하다.  
이 경우 해당 `struct` 변수의 `copy` 가 발생하는 경우 `class property` 는 주소 값만 복사된다.*__

예)
```swift
class TestA {
    var name = "testA"
}

struct TestB {
    var name = "testB"
    var testA = TestA()
    
}


let testB = TestB()
let testB2 = testB

print(testB.testA.name)
// testA

testB2.testA.name = "TestB2"
print(testB.testA.name)
// TestB2
```

### 어떤 상황에서 `struct` 를 사용하고, 어떤 상황에서는 `class` 를 사용해야하는가?

* `struct` 로 사용하면 좋은 케이스
    * 다른 타입으로 상속이 필요하지 않은 경우
    * `property` 로 `class` 인스턴스를 가질 필요가 없는 경우
    * `immutable` 이 필요한 경우
    * 대입보다는 생성되는 경우가 많은 경우
    * 여러곳에서 공유 될 필요가 없는 경우

class 를 사용하면 좋은 케이스는 위 상황의 반대로 설명 할 수 있다.


현재까지 작업한 프로젝트를 기준으로 보면  
`struct` 의 경우 대부분 `model class` 구현시에 사용하고,  
`View Model`, `Controller` 등을 구현하는 경우 `class` 를 사용하고 있었다.

`struct` 의 사용 조건? 을 보면 아주 잘못 사용한것 같지는 않지만  
데이터 공유 등의 상황에서 정확하게 사용했는지는 의문이 든다.  
<br>
<br><br>

======== 작성중 ========
## `struct` 와 `class` 의 차이점과 공통점, 특징을 간략 작성하면서 각각의 특징을 조금 더 자세히 작성할 필요가 있어 보여 아래 내용을 추가 한다.

### `struct` 의 특징

`value type`의 특성 중 가장 기본으로 __*변수 할당 및 `parameter` 전달시 `value copy` 가 일어난다.*__ 를 꼽을 수 있다.

코드 예시로 설명하면 아래와 같다.

```swift
struct TestA {
    var name = "testA"
}

var testA = TestA()
var testB = testA
testB.name = "testB"

print(testA.name)
// testA

print(testB.name)
// testB
```

`name` 이라는 `String` 타입의 `property` 를 가진 `struct TestA` 가 있을때  
`testA` 라는 `TestA struct` 의  변수를 생성하고
`testB` 라는 변수에 `testA` 를 대입한 이후  
`testB` 의 `name` 값을 _"testB"_ 로 변경한다.  
각 변수의 `name` 값을 `print` 해보면

`testA` 변수의 경우 초기 값인 _"testA"_ 가 찍히고,  
`testB` 변수의 경우 변경된 값인 _"testB"_ 가 찍힌다.  

```swift
var testB = testA
```
`testB` 에 `testA` 를 대입해 주는 순간에 `value copy` 가 되어 `testA` 와 `testB`는  
서로 다른 객체가 되기 때문에 이후 `testB` 의 `property` 를 변경해도 `testA` 에는 영향이 가지 않는다.  
<br>
동일한 코드를 `class` 로 변경했을때와 비교해보면 아래와 같다.  

```swift
class TestA {
    var name = "testA"
}

var testA = TestA()
var testB = testA
testB.name = "testB"

print(testA.name)
// testB

print(testB.name)
// testB
```

`struct` 로 구현했을때와 동일한 구조에서 `TestA` 만 `class` 로 변경했는데 `print` 되는 결과물이 다르다.  
`class` 로 구현한 경우
```swift
var testB = testA
```
해당 코드가 호출될때 `reference` 참조가 발생하기 때문에 `testA` 와 `testB` 가 바라보는 객체는  
실질적으로는 동일한 객체를 바라보게 되어 `testB` 의 `name property` 값을 변경하면  
`testA` 객체에도 영향이 가게 된다.