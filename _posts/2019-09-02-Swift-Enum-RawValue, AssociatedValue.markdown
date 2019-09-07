---
layout: post
title:  "Enum - RawValue, AssociatedValue"
date:   2019-09-02 14:42:20 +0900
categories: Swift
background: '/img/posts/03.jpg'
---

<!-- # Enum - RawValue, AssociatedValue -->

### Enum의 값(RawValue)

`enum`은 각각의 `case` 에 대해서 값을 지정할 수 있다.  
`C언어`에서는 `enum`에 정수만 지정할 수 있었지만  
`Swift`의 경우에는 훨씬 유연하게 타입 지정이 가능하다.  

```swift
enum Direction: Int {
    case left = 0
    case right = 1
    case top = 2
    case bottom = 3
}

print(Direction.top.rawValue)
// 2

enum ViewType: String {
    case button = "Button"
    case label = "Label"
    case imageView = "imageView"
}

print(ViewType.label.rawValue)
// Label

enum Constants: Double {
    case pi = 3.14
    case e = 2.71828
}

print(Constants.e.rawValue)
// 2.71828
```

`enum Direction: Int`, `enum ViewType: String` 과 같이 `enum` 선언,  `Direction` 네이밍 다음에 `: Type` 으로 원하는 타입을 지정 할 수 있습니다.  

`String` 이나 `Int`의 경우 `value` 표기를 지정하지 않아도 컴파일러가 알아서 채워 준다.  

```swift
// Mercury = 1, Venus = 2, ... Neptune = 8
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}

print(Planet.uranus.rawValue)
// 7

// North = "North", ... West = "West"
enum CompassPoint: String {
    case north, south, east, west
}

print(CompassPoint.east.rawValue)
// east
```

* 실수형(`Float`, `Double`)의 경우 `Int` 처럼(`1.0...8.0`) 자동으로 채워준다.  

Swift에서는 정수(Integer), 실수(Floating point), 문자열(String), 불리언(Bool) 값을 `Enum` 에 연결할 수 있으나, `CGPoint` 와 같은 타입을 `Enum` 에 지정 할 수 없다.  

이유는 `Enum`의 타입 지정의 경우 각 `case`에 연결 된 값(`rawValue`)를 통해  
비교문 `if variable == Enum.Type`이 성립되기 때문에  
`Equatable Protocol` 및 다음 `Protocol` 중 하나를 준수해야한다.  
정수 리터럴의 경우 `ExpressibleByIntegerLiteral`,  
부동 소수점 리터럴의 경우 `ExpressibleByFloatLiteral`,  
부동 소수점 리터럴의 경우 `ExpressibleByStringLiteral`,  
임의의 수의 문자를 포함하는 문자열 리터럴의 경우 `ExpressibleByStringLiteral`,  
단일 문자 만 포함하는 문자열 리터럴의 경우 `ExpressibleByUnicodeScalarLiteral` 또는 `ExpressibleByExtendedGraphemeClusterLiteral`  

```swift
class SomeType: Equatable & ExpressibleByIntegerLiteral {
    let value: Int
    
    static func == (lhs: SomeType, rhs: SomeType) -> Bool {
        return lhs.value == rhs.value
    }
    
    required init(integerLiteral value: Int) {
        self.value = value
    }
    
}

enum SomeSome: SomeType {
    case a = 1
    case b = 2
}

print(SomeSome.a.rawValue.value)
// 1
```

타입 지정이 된 `enum`의 연결된 값을 사용하려면 `rawValue` 프로퍼티를 통해 사용하면 된다.

```swift
print(Planet.uranus.rawValue)
// 7

print(SomeSome.a.rawValue.value)
// 1
```

`rawValue` 란 `enum` `case` 각각의 고유 값이라고 할 수 있다.  
즉, 프로퍼티를 통해 값을 읽어오는것 뿐만 아니라 반대로 rawValue를 통해  
`enum` `case`를 생성 하는 것도 가능하다.  

```swift
enum Direction: Int {
    case left = 0
    case right = 1
    case top = 2
    case bottom = 3
}


let kDirection = Direction(rawValue: 1)
print(kDirection?.rawValue)
// Optional(1)
```

타입 지정된 `enum`의 경우 기본적으로 `rawValue` 생성자 호출이 가능하며  
각 `case`에 연결이 안 된 예외적인 `rawValue`를 `parameter`로 받을 수 있기 때문에  
기본적으로 `init?(rawValue)` 옵셔널 생성자 함수 이다.  
단, 직접 생성자 함수를 구현하여 예외 케이스에 대한 처리를 해줄수도 있다.  

```swift
enum Direction: Int {
    case left = 0
    case right = 1
    case top = 2
    case bottom = 3
    
    case none = 4
    
    init(rawValue: Int) {
        switch rawValue {
        case 0:
            self = .left
        case 1:
            self = .right
        case 2:
            self = .top
        case 3:
            self = .bottom
        default:
            self = .none
        }
    }
}


let kDirection = Direction(rawValue: 1)
print(kDirection.rawValue)
// 1
```  
<br>

### Enum의 Associated Value

Associated Value는 `enum` 의 `case` 에 추가 정보를 담을 수 있는 독특한 방법이다.  
`enum` 을 사용하게 된다면 자주 활용 하게 되는 기법이고, 실제로 가장 최근 프로젝트에서도 사용했다.  

Alert을 표시할때 기본적인 표시 패턴으로 `title`, `message`와 `Ok` 버튼의 구조가 있을 수 있고  
`Ok`, `Cancel` 와 같은 버튼 구조일 수도 있다. 또한,  
커스터마이징 된 Alert으로 기본 UIAlert과 다른 구조로 표시해야하는 경우도 있을 수 있다.
이런 경우 `enum`의 `Associated Value`를 사용 할 수 있는데  

```swift
enum AlertType {
    case normal(title: String?, message: String?, ok: String?, cancel: String?)
    case banner(image: UIImage)
    ...
}

class AlertViewController {
    func configure() {
        switch alertType {
            case .normal(let title, let message, let ok, let cancel):
            ...
            break
            case .banner(image: UIImage?):
            ...
            break
        }
    }
}
```

상기된 코드처럼 `AlertType`이라는 `enum` 을 구현하고,  
`case` 로 기본적인 `Alert` 구현이 가능한 `normal` 타입과,  
이미지가 표시되는 `banner` 타입을 만들고  

실제 구현되야하는 `Custom Alert` 에서는 `AlertType`을 `parameter` 로 받는  
`init` 생성자를 구현, 구현해야하는 `AlertType` 을 저장해둔다.

이후 `viewDidLoad` 호출 시점에 전달 받은 `AlertType` 을 기준으로 UI를 구성한다.  

`Assoicated Value` 가 적용된 `enum case` 의 패턴 매칭 방법은 아래와 같다.  
```swift
enum Type {
    case a(title: String)
    case b(value: Int)
}

let kValueA = Type.a(title: "title")
let kValueB = Type.b(value: 2)

switch kValueB {
case .a(let title):
    print(title)
    break
case .b(let value):
    print(value)
    break
}
// 2

if case let Type.a(title) = kValueA {
    print(title)
}
// title
```

즉 `enum`의 `rawValue`는 타입 지정을 통해 `case` 별 값을 미리 지정해두는 방법 이고,  
`Associated Value`는 `case` 생성 시점에 값을 지정해주는 방법 이다.  

`Associated Value` 를 사용하는 경우 타입 지정을 하여 `rawValue` 를 사용 할 수 없고,  
`rawValue` 의 특성 중 하나인 `Equatable Protocol` 을 구현하지 않기 때문에 `==` 비교 문을 사용 할 수 없다.  

단, `rawValue`의 생성자 함수 구현 처럼 직접 `rawValue` 값을 구현하여 사용 할 수도 있다.

```swift
enum Type {
    case a(title: String)
    case b(value: Int)
    
    var rawValue: Int {
        get {
            switch self {
            case .a:
                return 0
            case .b:
                return 1
            }
        }
    }
}

print(Type.a(title: "Title").rawValue)
// 0
```

하지만 `rawValue` 자체를 지정해줬을 뿐 `Equatable Protocol` 을 구현한것은 아니기 때문에  
`==` 직접 비교는 불가능 하다. 


`Swift` 의 `enum`은 유연성이 뛰어나기 때문에 변수, 함수를 구현 하는 등 확장성이 뛰어난 방법으로  
실제 프로젝트에서 다양한 방법으로 활용 가능 하다.  