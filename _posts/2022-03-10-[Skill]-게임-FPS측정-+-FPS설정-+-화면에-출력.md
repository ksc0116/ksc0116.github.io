---
layout: post
title:  "[Skill] 게임 FPS측정 + FPS설정 + 화면에 출력"
subtitle:   ""
categories: Unity
comments: true
---

# FPS

FPS란 Frame Per Second 즉, 1초 동안 화면이 몇번이나 다시 그려지는가에 대한 값이다.

예시로 FPS가 30이면 1초에 화면을 30번 그린다는 뜻이다.

이제 FPS를 화면에 출력하는 코드를 봐보자.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FrameCount : MonoBehaviour
{
    private float deltaTime = 0f; //화면에 다시 그려지는 시간을 계산하는데 사용함

    [SerializeField, Range(1, 100)]
    private int size = 25; //글자의 크기를 정함
    [SerializeField]
    private Color color = Color.green; //글자의 색을 정함

    public bool isShow; //현재 FPS를 화면에 보여줄지 말지 정함

    void Update()
    {
        //한 프레임이 지나는데 걸리는 시간을 계산
        //0.1을 곱하는 이유는 너무 빠르게 프레임이 바뀌는 걸 방지하기 위함
        deltaTime += (Time.unscaledDeltaTime - deltaTime) * 0.1f;

        //F1을 누르면 화면에 보여줄지 말지 결정함
        if (Input.GetKey(KeyCode.F1))
        {
            isShow = !isShow;
        }
    }

    private void OnGUI() //GUI 이벤트에 따라서 프레임마다 여러차례호출 됨, 간단한 UI를 표시하기 위한 함수
    {
        if (isShow)
        {
            GUIStyle style = new GUIStyle();
                                            //x좌표,y좌표,너비,높이
            Rect rect = new Rect(30, 30, Screen.width, Screen.height);
            style.alignment = TextAnchor.UpperLeft; //정렬기준
            style.fontSize = size;
            style.normal.textColor = color;

            float ms = deltaTime * 1000f;
            float fps = 1.0f / deltaTime;
            string text = string.Format("{0:0.} FPS ({1:0.0} ms)", fps, ms);

            GUI.Label(rect, text, style);
        }
    }
}

```

실행결과는 아래와 같다.

![image](https://user-images.githubusercontent.com/101051124/157675793-0bdaa257-9785-42bb-9903-4a084b417a6b.png)

# FPS설정 방법

FPS를 설정해야 하는 이유는 FPS가 갑자기 올라갔다 내려가면 플레이어가 어색함을 느끼기 때문이다.

그래서 최대 30,60으로 FPS를 정하고 만약 사용자의 컴퓨터가 그 이상의 FPS를 견딜 수 있다면 제한을 해제하는 옵션을 추가하는 것이 좋다. 예시 코드를 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FrameCount : MonoBehaviour
{
    private float deltaTime = 0f; //화면에 다시 그려지는 시간을 계산하는데 사용함

    [SerializeField, Range(1, 100)]
    private int size = 25; //글자의 크기를 정함
    [SerializeField]
    private Color color = Color.green; //글자의 색을 정함

    public bool isShow; //현재 FPS를 화면에 보여줄지 말지 정함

    void Update()
    {
        //한 프레임이 지나는데 걸리는 시간을 계산
        //0.1을 곱하는 이유는 너무 빠르게 프레임이 바뀌는 걸 방지하기 위함
        deltaTime += (Time.unscaledDeltaTime - deltaTime) * 0.1f;

        //F1을 누르면 화면에 보여줄지 말지 결정함
        if (Input.GetKey(KeyCode.F1))
        {
            isShow = !isShow;
        }

        //1~4까지 입력을 받아서 프레임을 결정 해줌
        if (Input.GetKey(KeyCode.Alpha1))
        {
            Application.targetFrameRate = 30;
        }
        if (Input.GetKey(KeyCode.Alpha2))
        {
            Application.targetFrameRate =60;
        }
        if (Input.GetKey(KeyCode.Alpha3))
        {
            Application.targetFrameRate =144;
        }
        if (Input.GetKey(KeyCode.Alpha4))
        {
            Application.targetFrameRate =-1;
        }
    }

    private void OnGUI() //GUI 이벤트에 따라서 프레임마다 여러차례호출 됨, 간단한 UI를 표시하기 위한 함수
    {
        if (isShow)
        {
            GUIStyle style = new GUIStyle();
                                            //x좌표,y좌표,너비,높이
            Rect rect = new Rect(30, 30, Screen.width, Screen.height);
            style.alignment = TextAnchor.UpperLeft; //정렬기준
            style.fontSize = size;
            style.normal.textColor = color;

            float ms = deltaTime * 1000f;
            float fps = 1.0f / deltaTime;
            string text = string.Format("{0:0.} FPS ({1:0.0} ms)", fps, ms);

            GUI.Label(rect, text, style);
        }
    }
}

```

위 코드처럼 Application.targetFrameRate를 통해 프레임을 정할 수 있다. 유니티에서 초당 프레임 수를 정하지 않으려면 targetFrameRate를 -1로 설정하면 된다. 실행결과로 1을 눌렀을 때의 화면만 봐보자.

![image](https://user-images.githubusercontent.com/101051124/157678069-73ab9075-707e-41d4-b86b-268357af8310.png)

프레임이 30으로 고정된 걸 볼 수 있다.