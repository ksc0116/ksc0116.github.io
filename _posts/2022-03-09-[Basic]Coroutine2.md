---
layout: post
title:  "[Basic] 코루틴 2"
subtitle:   ""
categories: Unity
comments: true
---

<br>

이번에는 코루틴 중단하기와 코루틴 함수의 매개변수, yield break를 알아보자.

# 코루틴 중단시키기

```csharp
    void Start()
    {
        StartCoroutine(Test());
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            StopCoroutine(Test());
        }
    }
    IEnumerator Test()
    {
        yield return null;
    }
```

위의 코드를 보면 Start함수에서 코루틴을 실행 시키고 Update함수에서 스페이스바를 눌렀을 때 코루틴을 중단하게 되어있다. 하지만 실제로 스페이스바를 눌러도 코루틴이 중단되지 않는다.

왜 중단되지 않는지 살펴보자.

![image](https://user-images.githubusercontent.com/101051124/157430854-dcebcf76-4f57-4a69-aa23-72f80068df19.png)



![image](https://user-images.githubusercontent.com/101051124/157430987-510052ed-216d-47a6-b25e-e7e571b6f949.png)

StartCoroutine함수와  StopCoroutine함수의 매개변수 형식을 보면 IEnumerator형식이다.

그렇기 때문에 봔환형이 IEnumerator인 코루틴 함수를 매개변수로 받을 수 있는 것이다.

하지만 여기서 문제가 생긴다 위의 코드상에서  StartCoroutine함수에 전달된 IEnumerator와

StopCoroutine함수에 전달된 IEnumerator은 같은 값일까? 결론적으로는 다르다. 그렇기 때문에 Start함수에서 실행시킨 Test 코루틴 함수가 StopCoroutine에 의해 중단되지 않는 것이다. 그럼 중단시키는 방법이 없을까? 아니다 있다! 코드부터 봐보자!

```csharp
    IEnumerator enumerator;
    void Start()
    {
        enumerator = Test();
        StartCoroutine(enumerator);
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            StopCoroutine(enumerator);
        }
    }
    IEnumerator Test()
    {
        yield return null;
    }
```

위와 같이 IEnumerator형식으로 변수 하나 만들고 Test 코루틴 함수가 반환하는  IEnumerator 형식의 값을 enumerator에 저장해서 enumerator로 코루틴을 실행시키고 멈추게 하는 것이다.

그리고 사실 코루틴을 실행시키고 중단하게하는 방법이 하나가 더 있다. 

바로 **StartCoroutine("코루틴 함수명");**과 **StopCoroutine("코루틴 함수명");**이다.

코드로 봐보자.

```csharp
    void Start()
    {
        StartCoroutine("Test");
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            StopCoroutine("Test");
        }
    }
    IEnumerator Test()
    {
        yield return null;
    }
```

위와 같은 방법은 맨 처음 방법과는 다르게 StartCoroutine으로 실행시킨 코루틴을 StopCoroutine으로 중단시키는게 가능하다. 물론 당연한거지만 **StartCoroutine(Test());**로 실행한 코루틴을 **StopCoroutine("Test");**으로 중단시키는건 불가능 하다. 그리고 절대적으로 코루틴을 중단 시켜주는 함수가 존재한다. 

그것은 바로 **StopAllCoroutines();**이다. 이 함수는 현재 스크립트가 실행하고 있는 모든 코루틴을 중단 시킨다.

다음으로는 코루틴의 매개변수를 알아보자.

# 코루틴의 매개변수

코루틴 함수에도 매개변수를 설정할 수 있다. 우선 코드를 확인해보자.

```csharp
    void Start()
    {
        StartCoroutine(Test(10));
        StartCoroutine("Test",10);
    }

    IEnumerator Test(int num)
    {
        yield return null;
    }
```

위와 같이 코루틴을 호출하고자 할 때 매개변수 형식에 맞게 값을 넣으면 된다. 하지만   StartCoroutine("Test",10);와 같이 코루틴의 이름으로 호출을 하면서 매개변수를 넘길 때 문제가 조금 있다.

첫번째로, 

![image](https://user-images.githubusercontent.com/101051124/157434590-b0d765b0-96d3-4d7f-be36-3cbe2ad1b04c.png)

두번째 인자로 object형식을 받게 되어 있다. 이렇게 되면 박싱과 언박싱이 일어나기에 오버헤드가 조금이라도 발생하게 되고 남발하게되면 최적화에 문제가 생긴다.

두번째로, 코루틴의 이름으로 호출을 하면 매개변수를 2개이상 넣지 못한다는 단점이 있다.

마지막으로 yield break만 확인하고 끝내자.

# yield break

코루틴 안에서 yield break를 만나면 그 밑으로는 코루틴이 실행되지 않는다. 코드로 예시를 들어 보겠다.

```csharp
    IEnumerator Test(int num)
    {
        Debug.Log(1);
        yield break;
        Debug.Log(2);
    }
```

이렇게 존재하는 코루틴이 있다면 코루틴을 실행 시키면 1은 찍히지만 2는 찍히지 않는다.