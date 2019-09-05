---
layout: post
title:  "Swift - flatMap, compactMap"
date:   2019-09-05 13:48:20 +0900
categories: Swift
background: '/img/posts/04.jpg'
---

# Swift - flatMap, compactMap

### flatMap

`Swift 4.1` 부터 `flatMap` 이 `deprecated` 되고, `compactMap` 을 사용하도록 변경되었다.

<!-- 변경점이 무엇이며 왜 변경되었는지 -->

#### What

기존의 `flatMap` 정의는 총 세가지로 아래와 같다.

~~~Swift
Sequence.flatMap<S>(_: (Element) -> S) -> [S.Element] where S : Sequence
Optional.flatMap<U>(_: (Wrapped) -> U?) -> U?
Sequence.flatMap<U>(_: (Element) -> U?) -> [U]
~~~

`Sequence` 에 대한 `optional` 값을 `unwrapping` 하여 `Element` 를 `flat` 하게 `mapping` 해주는  
~~~Swift
Sequence.flatMap<U>(_: (Element) -> U?) -> [U]
~~~

이 정의가 `compactMap` 으로 분리되었다.  

코드로 예시를 들면 아래와 같다.

~~~Swift
let kArr = ["2", nil, "4", nil, "6"]
print(kArr.flatMap({ $0 }))

/*
    ["2", "4", "6"] 의 값이 print 되지만
    'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value
    Use 'compactMap(_:)' instead 라는 Swift Compiler Warning 이 발생한다.
*/
~~~

~~~Swift
let kArr = ["2", nil, "4", nil, "6"]
print(kArr.compactMap({ $0 }))

/*
    ["2", "4", "6"] 의 값이 print 되고 warning 이 발생하지 않는다.
*/
~~~


### Why

그렇다면 왜 `flatMap` 의 정의 중 마지막 정의만 `deprecated` 되어 `compactMap` 으로 변경 되었을까?

[Proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0187-introduce-filtermap.md#motivation) 문서에서 이유를 확인 할 수 있었다.

__*The last one, despite being useful in certain situations, can be (and often is) misused*__

즉, 해당 정의는 특정 상황에서 유용한 기능이지만 `map` 으로 처리 가능한 동작을 `flatMap` 으로 오용 하는 상황을 방지하기 위해 변경되었다고 말하고 있다.

예)
~~~Swift
struct Person {
  var age: Int
  var name: String
}

func getAges(people: [Person]) -> [Int] {
  return people.flatMap { $0.age }
}
~~~

`getAges` 함수가 호출되면 `parameter` 로 받은 `[Person]` 배열을 `[Int]` 배열로 변경하여 리턴해주기 위해  
`flatMap` 내부적으로 `optional` 이 아닌 값에 대해 불필요한 `wrapping`, `unwrapping` 을 하게 된다.  
아래와 같이 `map` 을 사용하면 쉽게 불필요한 `wrapping`, `unwrapping` 과정을 생략 할 수 있다.

~~~Swift
func getAges(people: [Person]) -> [Int] {
  return people.map { $0.age }
}
~~~

<!-- flatMap 의 다른 두가지 정의에 대한 post 는 따로 작성 -->