---
layout: post
title:  "Hot Observable, Cold Observable"
date:   2019-09-01 11:03:20 +0900
categories: RxSwift
background: '/img/posts/02.jpg'
---

# Hot Observable, Cold Observable

Observable은 이벤트 방출(emit) 시점에 따라
Hot Observable과 Cold Observable 로 구분 할 수 있다.

## Hot Observable
* 생성과 동시에 이벤트를 방출(emit) 한다.
* Subscribe 시점과 상관 없이 Observer에게 이벤트를 중간부터 전송한다.

예시)
~~~Swift
let kHotObservable = BehaviorRelay<Int>(value: 1)
kHotObservable.accept(2)

kHotObservable.subscribe(onNext: { value in
    print("next(\(value))")
}).disposed(by: disposeBag)

kHotObservable.accept(3)


let kHotObservable2 = PublishRelay<Int>()

kHotObservable2.accept(1)

kHotObservable2.subscribe(onNext: { value in
    print("next(\(value))")
}).disposed(by: disposeBag)

kHotObservable2.accept(2)
kHotObservable2.accept(3)

/* 결과
next(2)
next(3)
22222: next(3)
*/
~~~

Subscribe 시점과 상관 없이 중간부터 이벤트를 방출 한다.  

첫번째 Subscribe의 경우 subscribe 되기 전 accept된 값 2가  
subscribe 되고 바로 호출되는것을 확인 할 수 있고,  
subscribe 이후 accept 된 값 3도 호출되는 것을 확인 할 수 있다.

두번째 Subscribe의 경우 Subscribe 된 이후  
accept된 값 2, 3에 대한 이벤트가 호출 되는 것을 확인 할 수 있다.  

## Cold Observable
* Subscribe 되는 시점부터 이벤트를 생성해 방출한다.

~~~Swift
let kColdObservable = Observable<Int>.create({ observer in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)
    observer.onCompleted()
    
    return Disposables.create()
})

kColdObservable.subscribe(onNext: { value in
    print("next(\(value))")
}, onCompleted: {
    print("onCompleted")
}, onDisposed: {
    print("onDisposed")
}).disposed(by: disposeBag)


kColdObservable.subscribe(onNext: { value in
    print("222: next(\(value))")
}, onCompleted: {
    print("222: onCompleted")
}, onDisposed: {
    print("222: onDisposed")
}).disposed(by: disposeBag)

/* 결과
next(1)
next(2)
next(3)
onCompleted
onDisposed
222: next(1)
222: next(2)
222: next(3)
222: onCompleted
222: onDisposed
*/
~~~

Cold Observable은 이벤트를 방출하기전에 Observer가 Subscribe 할 때 까지 기다린다.  
따라서 Observer는 시작부터 이벤트 전체를 호출하는 것을 확인 할 수 있다.  