---
layout: post
title:  "애니메이션 속도 조절?????"
subtitle:   ""
categories: Unity
comments: true
---

<br>

에셋 스토어에서 다운 받은 애니메이션이 너무 느리다거나 너무 빠르면 우리는 속도를 조절한다.<br>

조절할 때 3가지 방법이 있는데 단점이 있는 방법들이 있다.<br>

1)Animaotr Controller에 가서 내가 원하는 State의 스티드를 바꿔준다.<br>

아래 사진은 AttackSpeed라는 Stated의 speed를 1에서 2로 바꾸는 작업이다.<br>

![simpleChangeSpeed](https://user-images.githubusercontent.com/101051124/157177630-4c8a5aea-3217-453a-8d26-6c8bae61b056.png)

<br>

하지만 위와 같은 방법은 게임 플레이중에 계속해서 속도를 바꾸지는 못한다.<br>

그래서 다른 방법이 존재한다.

2)스크립트상에서 속도 조절하기

~~~~csharp

	private void Update()
    {
        if (Input.GetButtonDown("Jump"))
        {
            anim.speed += 0.2f;
        }
    }
~~~
~~~~

<br>

위의 방법은 스페이스바를 누를때 마다 계속해서 애니메이션의 속도를 올리는 코드다.<br>

위와 같은 방법은 특정조건에 대해서 속도를 올리는 것은 가능하지만<br>

위의 방법도 단점이 존재하는데 그것은 Animator의 모든 State의 속도가 빨라진다는 것이다.<br>

그래서 마지막 방법!!! 내가 원하는 State만 속도를 올리는 방법이 존재한다!!<br>

3)애니메이터의 파라미터와 Multiplier로 원하는 스테이트의 속도 바꾸기<br>

![parameterSet](https://user-images.githubusercontent.com/101051124/157179928-2a70a7a0-8243-4462-b2db-8242548fcfb1.PNG)



<br>

위와 같이 float형으로 파라미터 하나를 만들고 기본값을 1.0으로 설정한다.<br>

다음으로는 아래 사진과 같이 원하는 스테이트의 Multiplier를 활성화 해주자<br>

![Multi](https://user-images.githubusercontent.com/101051124/157180405-a559250e-79f5-4d31-a359-4f927edd6f1a.PNG)

<br>

이제 스크립트로 넘어가서 파라미터 값만 바꿔주면 된다. 방법은 아래와 같다.<br>

~~~~csharp
    private void Update()
    {
        if (Input.GetButtonDown("Jump"))
        {
            anim.SetFloat("AttackSpeed", anim.GetFloat("AttackSpeed") + 0.2f);
        }
    }
~~~
~~~~

<br>

이러면 이제 스페이스바를 누를때 마다 내가 원하는 스테이트의 속도만 바꿔줄 수 있다.

<br>

1번 같은 경우는 애니메이션의 속도를 게임 플레이 중에 바꿀 필요가 없을 때 사용하면 좋다.

<br>

2번 같은 경우는 나의 움직임은 빠르게하고 게임에 존재하는 다른 오브젝트의 속도는 그대로 둬서

<br>

나의 움직임이 빨라진 것 처럼 보이게 할 때 사용한다.

<br>

3번은 내가원하는 스테이트의 속도만을 바꾸고 싶을 때 사용한다!

