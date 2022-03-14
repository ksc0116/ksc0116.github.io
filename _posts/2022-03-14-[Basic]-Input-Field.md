---
layout: post
title:  "[UI] Input Field"
subtitle:   ""
categories: Unity
comments: true
---

<br>

플레이어에게 직접 값을 입력할 수 있도록 만들어주는 컴포넌트 Input Field에 대해 알아보자.

우선 하이어라키창에 마우스 우클릭 -> UI -> Input Field를 눌러서 생성해준다.

![image](https://user-images.githubusercontent.com/101051124/158166892-ec9433b0-5cb6-4b55-abb9-ba907bca065f.png)

다음과 같이 나오고 화면에는

![image](https://user-images.githubusercontent.com/101051124/158166964-ee4f882e-adbc-46bc-8fc7-5ae70d07f26c.png)

이렇게 뭔가 입력이 가능하게 나온다 실제로 게임을 실행시켜서 확인해보겠다.

![image](https://user-images.githubusercontent.com/101051124/158167238-4dc50eb8-8c1b-42d9-9a03-8073b34aa9b5.png)

다음과 같이 내가 입력한 그대로 화면에 출력이 되고 하이러라키창에 InputField Input Caret라는 오브젝트가 생긴걸 확인이 가능하다. **InputField Input Caret**은 화면에서 커서가 깜빡거리게 해주는 놈이다. **PlaceHolder**는 맨처음 화면에서 플레이어가 값을 입력하지 않았을 때 보이는 텍스트이다.처음에는 Enter text...가 나온다.

![image](https://user-images.githubusercontent.com/101051124/158167891-9b5aa386-b7e0-45a9-a7d9-fc4dcc043e6f.png)

위 화면처럼 PlaceHolder에서 Text프로퍼티를 바꿔주면 그대로 화면에 반영이 되는 것을 볼 수 있다.

그리고 InputFiled안에있는 Text는 플레이어가 입력한 텍스트이다. 다음 화면을 보자.

![image](https://user-images.githubusercontent.com/101051124/158168234-4a5ce77b-e284-4345-ab34-07918950f9b1.png)

다음과 같이 입력한 그대로 인스펙터창에 적용되는 걸 확인이 가능하다.

이제 Input Field의 프로퍼티들을 확인해보자.

![image](https://user-images.githubusercontent.com/101051124/158168963-40b56db0-83bc-47ad-b9f7-5159ffca8be8.png)

우선 **Text Component**는 Text와 **Placeholder**는 PlaceHolder와 연동이 된 걸 확인이 가능하다.

# Charactor Limit

Charactor Limit는 말 그대로 문자의 갯수를 제한한다. 여기에 5를 넣으면 플레이어는 5글자만 입력이 가능하게 된다.

# Content Type

Content Type은 입력 스타일을 정할 수 있다. 숫자만 입력 가능하게 하던지, 입력된 값들을 전부 별로 보이게 한다던지 말이다.

# Caret 프로퍼티들

![image](https://user-images.githubusercontent.com/101051124/158169721-61045052-0af5-4761-82b0-31f9a3b84244.png)

### Caret Blink Rate

커서가 깜빡거리는 주기

### Caret Width

커서의 너비

### Custom Caret Color

커서의 색을 커스텀 할건지

![image](https://user-images.githubusercontent.com/101051124/158170113-8c3765e9-e8c8-4938-99a6-06c107a23ac2.png)

다음과 같이 빨간색으로 하면 커서의 색이 빨간색이 된다.

### Selection Color

드래그 했을 때 색상 지금 연두색으로 되어있는데 드래그 해보겠다.

![image](https://user-images.githubusercontent.com/101051124/158170315-5c4cf9ae-ee25-4b70-807f-a3245e1d2bfd.png)

다음과 같이 연두색이 나오는걸 확인할 수 있다.

### On Value Changed (String)

![image](https://user-images.githubusercontent.com/101051124/158170468-0a0cebf0-0e08-426c-a7b7-9211ac7df426.png)

 On Value Changed (String)은 값을 입력했을 때 호출 할 함수를 추가할 수 있다. 어디에 쓰이냐면 예를들어 우리는 1에서5부터의 숫자만 입력 받게 하고싶은다. 그런데플레이어가 6~9의 숫자를 입력하면 그 입력을 지우는 함수를 만들어서 추가해주는 식으로 사용한다.

### On End Edit(String)

![image](https://user-images.githubusercontent.com/101051124/158170788-e0610679-8a16-4a36-8be5-24662c4845ec.png)

On End Edit(String)은 플레이어가 입력을 끝내고 엔터를 치거나 다른 곳을 클릭해서 입력창을 나갈 때 호출 할 함수를 추가해준다.

이제 간단한 예제로 입출금 시스템을 만들어 보겠다.

우선 다음과 같은 화면 구성을 하기 위해서 기본 텍스트 UI, 버튼 UI 2개를 생성해준다.

 ![image-20220314213927470](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220314213927470.png)

그리고 안의 프로퍼티들을 위 형식에 맞게 바꿔준다. 이제 스크립트로 넘어가보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Test : MonoBehaviour
{
    [SerializeField] private Text txt_money; //현재 나의 돈을 화면에 보여주는 텍스트
    [SerializeField] private InputField input_txt_money; //플레이어가 입력하는 텍스트

    private int cur_money; //현재 나의 돈

    public void Input()//입금 버튼이 눌렸을 때 호출 될 함수
    {
        cur_money += int.Parse(input_txt_money.text);//string이라 int형으로 형변환
        txt_money.text  = cur_money.ToString();//int형이라 string으로 형변환
    }

    public void Output()//출금 버튼이 눌렸을 때 호출 될 함수
    {
        cur_money -= int.Parse(input_txt_money.text);

        txt_money.text = cur_money.ToString();
    }
}
```

어려운 코드는 아니니 설명은 넘어 간다. 이제 이스크립트를 캔버스 오브젝트에 추가하고 필드들을 초기화 해준 다음 입금 버튼,출금 버튼에 각각 클릭되었을 때 호출 될 함수들을 등록해주면 끝이다. Input Field는 정말 많은 곳에서 쓰인다. 예를들면 게임에 로그인 할 때 아이디와 비밀번호 입력창, 또는 플레이어가 아이템을 버릴 때 몇 개 버릴지 입력하는 입력창 등등 아주 다양하게 많이 쓰이니 알아두는게 좋다. 