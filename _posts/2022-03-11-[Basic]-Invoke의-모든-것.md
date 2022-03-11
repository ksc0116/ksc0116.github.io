---
layout: post
title:  "[Basic] Invoke의 모든 것"
subtitle:   ""
categories: Unity
comments: true
---

<br>

이번 글에서는 Invoke에 대하여 다룰 것이다. 기본적인 Invoke함수는 알고 있기에 넘어가겠다.

# InvokeRepeating()

InvokeRepeating()함수는 매개변수 인자로 3가지를 받는다. 다음과 같이 말이다.

InvokeRepeating("실행하고자 하는 함수명",3f,1f);

여기서 2번째 3번째 매개변수의 의미를 보면 3초뒤에 함수가 실행되고 그 뒤로는 1초마다 반복해서 실행하라는 뜻이다.

# CancelInvoke()

CancelInvoke()함수는 현재 작동중인 Invoke함수를 멈추게 한다. CancelInvoke();처럼 매개변수에 아무것도 넘겨 주지 않으면 현재 오브젝트가 실행중인 Invoke함수를 모두 멈추고 CancelInvoke("멈추고 싶은 함수명"); 다음과 같이 내가 멈추고 싶은 함수의 이름을 매개변수로 넘기면 내가 멈추고 싶은 함수만 멈추는게 가능하다.

# IsInvoking()

IsInvoking()함수는 현재 작동하고 있는 Invoke함수가 있는지 판단한다. 반환형은 bool형이다. 이 함수도 마찬가지로 IsInvoking(); 매개변수에 인자를 넘겨주지 않으면 현재 하나라도 작동중인 Invoke함수가 있으면 true를 반환한다. IsInvoking("확인하고 싶은 함수명"); 이렇게 매개변수를 넘기면 내가 확인하고 싶은 함수가 작동중인지 확인할 수 있다.

# 단점

Invoke함수에는 단점이 있다. 우선 Invoke함수를 코루틴 함수로 똑같이 작동하게 만들 수 있다. 유니티 공식 문서에 따르면 Invoke대신 코루틴을 사용하는걸 권장하고 있다. 성능부분에서 확인해보면 Invoke는 함수명을 문자열로 찾아내고 있는데 이러면 c#의 리플렉션이라는 기능을 사용하게 되는데 이렇게 리플렉션을 통해 함수를 찾아서 동작하게 하는건 직접 함수를 호출하는거보다 수천배 정도 느리다고 한다.