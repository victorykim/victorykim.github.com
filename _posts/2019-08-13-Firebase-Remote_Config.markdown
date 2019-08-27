---
layout: post
title:  "Firebase - Remote Config"
date:   2019-08-13 15:53:20 +0900
categories: Firebase
background: '/img/posts/06.jpg'
---

최근 진행하던 프로젝트의 앱스토어 심사 과정에서  
발생한 문제와 해결 방법에대 기록하는 글  
<br>

개발 중이던 앱의 서비스 지역이 한국 고정이고 회원가입, 로그인 방식이  
Nice 본인인증만으로 동작하기 때문에 한국 통신사를 사용하는 사람만   
회원가입, 로그인이 가능하지만 리뷰어의 경우 해외 거주 외국인으로  
로그인이 불가능 하기 때문에 데모 계정이 필요하다는 사유로 리젝되는 상황

해당 문제를 해결하기 위해 처음에 수정한 방식은

{% highlight swift %}
if Locale.current.regionCode != "KR" && Date() < "2019-08-13".date(format: "yyyy-MM-dd")! {
    // 리뷰어용 로그인 화면 호출
} else {
    // 일반 사용자용 로그인 화면(Nice 본인인증 웹뷰) 호출
}
{% endhighlight %}

이런 방식으로 Locale.current.region 값이 "KR"이 아니고  
오늘 날짜가 특정 날짜 이전인 경우에 Native로 만든 리뷰어용 로그인 화면을 띄우고  

{% highlight swift %}
guard kId == "ReviewerId" && kPw == "ReviewerPassword" else {
    // 로그인 실패 처리
    return
}
{% endhighlight %}

리뷰어용 로그인 화면에서는 입력된 ID, Password를 로컬에서 비교,  
리뷰어용 정보인 경우 테스트 유저 정보의 key 값으로 로그인 시도 하도록 구현 했으나  

위 방법의 경우 아래와 같은 문제 점이 있어 좋은 방법이 아닌 상태

1. 일반 사용자의 경우에도 지역설정을 변경하는 경우 리뷰어용 로그인 화면에 진입이 가능
2. 앱스토어 심사 기간이 정확하지 않기 때문에 제한 설정한 날짜 이내에 심사 통과가 안되는 경우 리뷰어의 로그인이 불가능
3. 로컬에서 ID, Password를 비교하기 때문에 보안상 위험
4. 3번과 같은 원인, 로컬에서 테스트 유저 정보의 key를 전송하기 때문에 위험

여러가지 방법을 찾아 봤지만 상기된 문제를 모두 회피 할 방법을 찾기 어려워 보였지만  
지인을 통해 Remote Config라는 방법을 알게 되었고  
개발중인 프로젝트에서 MLKit, Cloud Messaging 등 여러가지 기능 구현 문제로  
Firebase를 적극적으로 사용 중이였는데 Firebase가 Remote Config 기능을 지원하여  
Firebase의 Remote Config를 적용하고 다시 앱스토어 심사 제출했고  
무사히 통과해서 앱스토어 출시까지 성공,  
일반 사용자에게는 리뷰어용 로그인 화면이 노출 안되는것까지 확인 할 수 있었다.
<br>

**Remote Config에 대한 설명과 적용 방법**

Firebase Reference 문서에서는 Remote Config에 대해 이렇게 설명하고 있다.

_**"앱 업데이트를 게시하지 않아도 하루 활성 사용자 수 제한 없이 무료로 앱의 동작과 모양을 변경할 수 있습니다."**_

_**"Firebase 원격 구성은 사용자가 앱 업데이트를 다운로드할 필요 없이 앱의 동작과 모양을 변경할 수 있는 클라우드 서비스입니다. 원격 구성을 사용할 때는 앱의 동작과 모양을 제어하는 인앱 기본값을 만듭니다. 그런 다음 나중에 Firebase 콘솔 또는 Remote Config REST API를 사용하여 모든 앱 사용자 또는 사용자층의 특정 세그먼트에 대한 인앱 기본값을 재정의할 수 있습니다. 업데이트를 적용할 시점을 앱에서 제어할 수 있으며, 성능에 거의 영향을 주지 않고 업데이트를 빈번하게 확인하여 적용할 수 있습니다"**_


간단하게 설명하면
Firebase 서버에 원격 구성 요소를 정의하고
앱에서는 해당 정보를 받아와서 앱에 반영하는 기능이라고 할 수 있다.

프로젝트 설정 방법은 공식 문서를 참조 [Remote Config 셋팅 방법](https://firebase.google.com/docs/remote-config/use-config-ios?hl=ko)

Remote Config 사용 방법은

**1. RemoteConfig의 configSettings를 설정한다.**

{% highlight swift %}
let kConfigSettings = RemoteConfigSettings()
kConfigSettings.minimumFetchInterval = 0
RemoteConfig.remoteConfig().configSettings
{% endhighlight %}

**2. RemoteConfig에서 정보를 받아오기 전 갱신해준다.**

{% highlight swift %}
RemoteConfig.remoteConfig().fetch(completionHandler: { status, error in
    // Stauts가 Success인 경우 다음 Flow 진행
})
{% endhighlight %}

**3. Fetch에 성공한 경우 받아온 값을 활성화한다.**

{% highlight swift %}
RemoteConfig.remoteConfig().activate(completionHandler: { error in
    // error가 발생하지 않은 경우 다음 Flow로 진행            
})
{% endhighlight %}

**4. Firebase RemoteConfig에 설정된 값을 받아와서 사용한다.**

{% highlight swift %}
let kRemoteVersion = RemoteConfig.remoteConfig().configValue(forKey: "Version").stringValue
}
{% endhighlight %}

즉, 상기된 문제의 해결 법으로 Firebase 서버에 앱스토어에 출시된 버전과  
심사 받으려는 버전을 비교 할 수 있는 정보와 리뷰어가 사용 할 테스트 계정의  
아이디, 패스워드, 테스트 계정 키 값을 등록해두고  

로그인 버튼을 클릭하는 시점에 Firebase에 등록된 정보를 갱신, 받아온 후  
앱 버전이 심사 중인 버전인 경우 리뷰어용 로그인 화면을 표시,  
앱스토어 출시된 버전인 경우 일반 사용자 로그인 화면을 표시

리뷰어용 로그인 화면에서 로그인 시도 할 경우 Firebase에 등록해둔  
아이디, 패스워드가 입력된것인지 확인 테스트용 계정 정보와 일치하는 경우  
테스트 계정 키를 Firebase 서버에서 받아와서 로그인 API를 호출하도록 수정했고  
첫 심사를 통과하고 이후 진행한 업데이트 심사까지 통과  

Remote Config를 적용한 방법의 단점은

앱스토어 심사 제출시 출시 설정을 수동으로 설정해야하고  
심사 통과 이후에 Firebase의 정보를 업데이트 하고  
앱스토어 수동 출시를 해야하는 문제점이 있고  
현재는 심사 제출시에만 심사 버전에대한 정보를 갱신하여  
출시된 앱에는 영향이 없고,  
자동 출시로 설정해도 문제가 없는 방법이 있을지 확인하고 있다.
