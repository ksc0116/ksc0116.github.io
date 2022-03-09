---
layout: post
title:  "[Basic] 코루틴 1"
subtitle:   ""
categories: Unity
comments: true
---

<br>

이번에는 코루틴에 대한 포스팅을 해보려고 한다. 코루틴은 굉장히 많이 쓰이고 중요하다 생각한다.

코루틴을 사용하는 이유로는 Update와 다르게 내가 원하는 시점에 잠시동안 함수 실행을 멈추었다 다시 함수가 실행되게 만들 수 있기 때문이다.

예를 들어 캐릭터가 공격을 하고 일정 시간동안 공격을 못하게 하는 기능을 만들 수 있다.

코루틴 함수를 만드는 법은 간단하다.

**코루틴 함수 만들기**

```csharp
    IEnumerator AttackDelay()
    {
        yield return null;
    }
```

위와 같이 반환형을 IEnumerator로 설정해주면 된다. 근데 일반 함수와 다른점이 또 있다.

바로 일반 함수는 반환할 때 return만 쓰는데 코루틴은 yield return을 쓴다는 것이다. 여기서 yield return의 의미는 함수 실행 중 yield return을 만나면 yield return뒤에 명시 된 만큼 잠시 함수가 멈추었다 다시 실행이 된다.

여기서 yield return 뒤에 붙일 수 있는 것들을 확인해보자.

```csharp
    IEnumerator AttackDelay1()
    {
        yield return null; //한 프레임만 기다림
        yield return new WaitForSeconds(2f);//2초동안 기다림





        Time.timeScale = 0.5f; //시간의 길이를 조절함(게임속 시간의 흐름이 절반으로 느려짐)
        yield return new WaitForSeconds(1f); //2초를 기다리게 됨
        yield return new WaitForSecondsRealtime(1f);//타임 스케일에 영향을 받지 않고 1초만 기다림

        yield return new WaitForFixedUpdate();//픽스드업데이트가 끝날때 까지 기다림
        yield return new WaitForEndOfFrame();//업데이트는 물론 화면을 렌더링하는 과정 등 한 프레임이 지나가는 과정이 끝날 때 까지 기다림
    }
```

여기서 주의 할 점은 WaitForSeconds와  WaitForSecondsRealtime의 차이점을 아는 것이다.

위의 코드에서 보이는 것과 같이 WaitForSeconds는 타임스케일의 영향을 받고 WaitForSecondsRealtime는 받지 않는다.

**코루틴을 실행시키기**

코루틴을 실행 시키려면 일반함수와 달리  StartCoroutine(AttackDelay()); 처럼 StartCoroutine함수 안에 내가 실행시키고자 하는 코루틴을 써주면 된다.

# 코루틴 사용 시 주의 사항

1) 코루틴은 스타트함수가 호출 된 이후에 사용 되어야 하며 게임 오브젝트가 활성화 되어 있어야함
2) 코루틴 내에 무한루프를 만든다면 무한루프 안에 yield return은 반드시 써야한다.



