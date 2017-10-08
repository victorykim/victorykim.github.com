---
layout: post
title:  "RxSwift - Observable"
date:   2017-10-06 16:47:20 +0900
categories: RxSwift
---

**Observable 생성**

{% highlight swift %}

// #1. Generic observable 생성
let observable: Observable<String> = Observable<String>.create({ observer in
	observer.onNext("Next")             // #2. 이벤트 전송
	observer.onError(error)             // #3. 에러 전송
	observer.onCompleted()              // #4. observable completed 전송
	return Disposables.create()         // #5. disposable 리턴
})
{% endhighlight %}

**\#1. String 타입 Observable 생성**

{% highlight swift %}
Observable<String>.create(subscribe: (AnyObserver<String>) -> Disposable)

#=>subscribe: observer를 parameter로 받고 Disposable을 리턴하는 closure parameter
{% endhighlight %}

**\#2. 이벤트 전송**

{% highlight swift %}
observer.onNext("Next")
#=>"Next" 라는 문자열 이벤트를 전송
{% endhighlight %}

**\#3. 에러 전송**

{% highlight swift %}
observer.onError(error)
#=>Error 전송
{% endhighlight %}

**\#4. observable completed 전송**

{% highlight swift %}
observer.onCompleted()
#=>Completed 전송
{% endhighlight %}

**\#5. Disposable**

{% highlight swift %}
return Disposables.create()
#=>Disposable 리턴
{% endhighlight %}


Observable의 기본 Cycle을 요약하면

1. Generic Type으로 Observable을 생성한다.
2. Observer를 parameter로 받고 Disposable을 리턴하는 Closure를 구현
3. 정상적인 이벤트 전송의 경우 observer.onNext(Generic Type)을 통해 전송
4. 에러가 발생한 경우 observer.onError(error)를 통해 에러 전송
5. Observable의 완료를 원하는 경우 observer.onCompleted() 실행
6. 이벤트 완료 후 Disposable 리턴

-

**Subscribe**

{% highlight swift %}
// #1. Observable subscribe 설정
observable.subscribe(onNext: { text in
	// #2. 이벤트 발생시 동작 구현
	print("onNext: \(text)")
}, onError: { error in
	// #3. 에러 발생시 동작 구현
	print("onError: \(error.localizedDescription)")
}, onCompleted: {
	// #4. Completed 발생시 동작 구현
	print("onCompleted")
}, onDisposed: {
	// #5. Disposed 발생시 동작 구현
	print("disposed")
}).disposed(by: disposeBag)	// #6. subscribe 해제를 위해 disposeBag에 등록
{% endhighlight %}

**\#1. Observable subscribe 설정**

{% highlight swift %}
observable.subscribe(onNext: ((String) -> Void)?, onError: ((Error) -> Void)?, onCompleted: (() -> Void)?, onDisposed: (() -> Void)?)
#=>onNext: 이벤트 발생하는 경우 실행할 Observable 생성시 설정한 Generic Type을 parameter로 받는 Optional Closure parameter
#=>onError: 에러 발생시 실행할 Error를 parameter로 받는 Optional Closure parameter
#=>onCompleted: complted 발생시 실행할 Void Type Optional Closure paramter
#=>onDisposed: disposed 발생시 실행할 Void Type Optional Closure paramter
{% endhighlight %}

**\#2. Generic Type 이벤트 발생시 동작하는 Closure**

Optional Type closure로 구현을 안하고 nil을 넘겨주는것이 가능

Observable 생성시 구현한 onNext 이벤트 발생시 호출되는 closure

이벤트가 발생하는 경우 원하는 동작을 구현, 예시 코드의 경우

전달 받은 문자열을 print 하도록 구현

{% highlight swift %}
print("onNext: \(text)")
#=> "onNext: Next" 출력
{% endhighlight %}

**\#3. 에러 발생시 동작하는 Closure**

Optional Type closure로 구현을 안하고 nil을 넘겨주는것이 가능

Observable 생성시 구현한 onError 이벤트 발생시 호출되는 closure

이벤트가 발생하는 경우 원하는 동작을 구현, 예시 코드의 경우

전달 받은 에러의 localizedDescription을 print 하도록 구현

{% highlight swift %}
print("onError: \(error.localizedDescription)")
#=> "onError: (에러 도메인, 에러 코드 등)" 출력
{% endhighlight %}

**\#4. Completed 발생시 동작하는 Closure**

Optional Type closure로 구현을 안하고 nil을 넘겨주는것이 가능

Observable 생성시 구현한 onCompleted 이벤트 발생시 호출되는 closure

이벤트가 발생하는 경우 원하는 동작을 구현, 예시 코드의 경우

문자열 "onCompleted" print 하도록 구현

{% highlight swift %}
print("onCompleted")
#=> "onCompleted" 출력
{% endhighlight %}

**\#5. Disposed 발생시 동작하는 Closure**

Optional Type closure로 구현을 안하고 nil을 넘겨주는것이 가능

Observable의 subscribe이 disposed 되는 경우 호출되는 closure

이벤트가 발생하는 경우 원하는 동작을 구현, 예시 코드의 경우

문자열 "disposed" print 하도록 구현

{% highlight swift %}
print("disposed")
#=> "disposed" 출력
{% endhighlight %}

**\#6. subscribe 해제를 위해 disposeBag에 등록**

Observable 이벤트 종료되는 경우 subscribe을 해제하고 메모리 해제를 위해 DisposeBag 객체에
Observable을 등록하는 부분


subscribe의 기본 Cycle을 요약하면

1. Observable의 subscribe을 설정한다.
2. onNext 이벤트 발생시 실행할 closure를 구현한다.
3. onError 이벤트 발생시 실행할 closure를 구현한다.
4. onCompleted 이벤트 발생시 실행할 closure를 구현한다.
5. onDisposed 이벤트 발생시 실행할 closure를 구현한다.
6. subscribe 해제를 위해 Disposed를 설정한다.

일반적으로 Observable의 subscribe의 경우 아래와 같이 onNext의 동작만 구현을 하는
bind를 주로 사용한다.
{% highlight swift %}
observable.bind(onNext: (String) -> Void)
{% endhighlight %}

-

**Disposed**

Observable의 subscribe 해제 시점은 disposed(by: DisposeBag) 호출시

parameter로 전달한 DisposeBag 객체가 deinit되는 경우, 즉 DisposeBag 객체가

메모리에서 해제 되는 경우에 subscribe이 해제 되고 Observable 또한 메모리에서 해제 된다.

보통의 경우 UIViewController의 전역 변수로

{% highlight swift %}
private let disposeBag: DisposeBag = DisposeBag()
{% endhighlight %}

구현하고 해당 ViewController가 deinit 되는 경우 DisposeBag 객체가 deinit되면서

메모리에서 해제가 된다.

원하는 시점에 강제로 disposed 하길 원하는 경우

let이 아닌 var로 구현한 후

{% highlight swift %}
disposeBag = DisposeBag()
{% endhighlight %}

새로운 DisposeBag 객체를 생성해주면 기존 DisposeBag이 메모리에서 해제되며

subscribe 또한 해제가 된다.