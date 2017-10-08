---
layout: post
title:  "RxSwift - Variable"
date:   2017-10-08 13:51:20 +0900
categories: RxSwift
---

**Variable**

Variable은 단어 그대로 변수와 같은 동작 방식으로

값을 저장하여 사용하며, 값이 변경될때 마다 이벤트를 발생한다.

BehaviorSubject의 Wrapper 클래스로 더 익숙한 이름으로 사용하기 위해 만들어졌다.

Variable은 이벤트는 발생하지만 Error와 Completed를 발생시키지는 못하고,

deinit되는 순간 자동으로 Completed 이벤트를 발생한다.

-

**Variable 생성**

Generic Type으로 일반 변수 생성과 동일한 방식으로 생성하며 반드시 초기 값을 지정해야한다.

{% highlight swift %}
// Variable 생성
let variable: Variable<String> = Variable<String>("variable")
{% endhighlight %}

**Variable subscribe**

Variable을 subscribe 하기 위해서는

asObservable() 함수를 호출하여 Observable로 형변환을한 이후

Observable의 subscribe과 동일하게 구현을 한다.

Variable의 경우 에러 이벤트를 발생하지 않고,

Completed 이벤트도 Variable이 deinit되는 순간 자동으로 발생하기 때문에

bind(onNext: (Generic Type) -> Void)로 이벤트에 대한 closure만 구현한다.

아래 예시 코드의 경우 Completed가 호출되는 순간을 확인하기 위해 subscribe으로 구현

{% highlight swift %}
// Variable subscribe
variable.asObservable().subscribe(onNext: { text in
	print("bind onNext: \(text)")
}, onError: nil, onCompleted: {
	print("onCompleted")
}, onDisposed: {
	print("onDisposed")
}).disposed(by: disposeBag)

variable.value = "observable"
{% endhighlight %}

**Variable의 값 변경**

Variable의 값을 변경하는 방법은 두가지가 있다.

\#1. variable.value에 값을 대입한다.

{% highlight swift %}
variable.value = "observable"
{% endhighlight %}

\#2. observable.bind(to: Variable<Generic Type>)에 파라미터로 전달한다.

{% highlight swift %}
Observable<String>.just("value").bind(to: variable).disposed(by: disposeBag)
{% endhighlight %}

\#1.의 방법은 일반 변수를 사용하듯 Variable의 값 variable.value에 새로운 값을 대입하는 방법이고

\#2.의 방법은 같은 타입의 Observable에 bind(to: Variable<Generic Type>) 함수의 파라미터로 variable을 전달하는 방법이다.

Observable의 이벤트가 발생하는 경우 해당 이벤트의 값을 Variable에 bind 시키는 방법으로

아래 예시 코드와 같이 observable.subscribe의 onNext closure에서 전달 받은 값을 Variable의 value에 대입 시키는것과 같다.

{% highlight swift %}
observable.subscribe(onNext: { text in
	variable.value = text
},...
{% endhighlight %}

**Variable 이벤트**

Variable을 생성하면서 설정한 초기 값으로 subscribe이 된 후 바로 이벤트가 발생하고,

이후 value가 변경될때마다 이벤트가 발생하게 된다.

{% highlight swift %}
// Variable 생성
let variable: Variable<String> = Variable<String>("variable")

// Variable 구독
variable.asObservable().subscribe(onNext: { text in
	print("bind onNext: \(text)")
}, onError: nil, onCompleted: {
	print("onCompleted")
}, onDisposed: {
	print("onDisposed")
}).disposed(by: disposeBag)
        
variable.value = "observable"

/// 실행 결과

/// Variable 생성시 설정한 초기 값 "variable"
/// "bind onNext: variable"

/// subscribe 설정 후 변경한 값 "observable"
/// "bind onNext: observable"

/// 지역 변수 variable이 deinit 되는 순간 발생한 onCompleted의 print("onCompleted")
/// "onCompleted"

/// 지역 변수 variable이 deinit 되는 순간 발생한 onDisposed의 print("onDisposed")
/// "onDisposed"
*/
{% endhighlight %}