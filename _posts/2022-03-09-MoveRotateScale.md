---
layout: post
title:  "[Basic] 스크립트로 오브젝트 이동, 회전, 크기 조절"
subtitle:   ""
categories: Unity
comments: true
---

<br>

우선 스크립트로 오브젝트를 이동하는 방법을 살펴보자. 

1)transform.position

~~~csharp
 transform.position = new Vector3(0f,1f,0f);
~~~

위와같은 방법은 해당 스크립트를 컴포넌트로 가지고 있는 오브젝트의 position을 (0,1,0)으로 이동 시키는 것 이다. 

2)transform.Translate(Vector3 * Time.deltaTime)

```cs
 transform.Translate(new Vector3(0,1,0) * Time.deltaTime);
```

여기서 매개변수로 전해지는 Vector3값은 위치를 나타내는게 아니라 방향,속도(거리)를 나타내기 때문에 Update문에서 호출하면 Vector3의 방향,속력으로 게속해서 움직인다. 그리고 Time.deltaTime을 곱해주는 이유는 프레임 당 움직이는 거리를 똑같게 해주기 위함이다. transform에 의한 이동은 항상 Time.deltaTime을 곱해주어야 한다.



그 다음으로는 오브젝트의 회전이다.

1)transform.rotation

```csharp
  transform.rotation=Quaternion.Euler(0,0,0); 
```

위의 방법에서 Quaternion.Euler(0,0,0); 이게 뜻하는 것은 Euler 안에있는 Vector3형식을 Quaternion으로 바꿔준다. 즉,  Quaternion.Euler(90,0,0);을 transform.rotation에 넣으면 해당 오브젝트는 x축을 기준으로 90도 회전하게 된다.

2)transform.Rotate(Vector3)

```csharp
 transform.Rotate(Vector3.up*Time.deltaTime,Space.World);
```

위의 방법은 월드 좌표계 기준으로 내가 원하는 축을 기준으로 오브젝트를 회전 시켜주는 함수이다. 함수의 매개변수 마지막에 Space.World를 명시해주지 않으면 로컬 좌표계 기준으로 회전한다.

3)transform.forward

```csharp
transform.forward = Vector3.up;
```

위의 방법은 오브젝트를 내가 원하는 방향을 바라보게 만든다. 즉 위의 코드는 오브젝트를 위를 바라보게 만든다.

4)transform.LookAt(Vector3)

```csharp
    public Transform target;
    void Update()
    {
        transform.LookAt(target.position);
    }
```

transform.LookAt함수는 보통 위와 같이 특정 오브젝트의 위치를 계속해서 바라보게 회전하는 방식으로 많이 사용한다.

transform.forward와의 차이점은 transform.forward는 바라 볼 방향을 정하지만 LookAt함수는 바라 볼 지점을 정한다.

<br>

마지막으로 오브젝트의 크기를 바꾸는 방법을 살펴보자

1)transform.localScale

```csharp
 transform.localScale = new Vector3(0, 0, 0);
```

여기서 로컬 스케일을 쓰는 이유는 월드 기준으로 오브젝트의 스케일을 바꾸면 오브젝트가 회전하게 되었을 때 내가 원하는 모양으로 스케일이 바뀌지 않기 때문이다.



