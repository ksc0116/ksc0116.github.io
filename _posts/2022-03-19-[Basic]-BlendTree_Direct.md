---
layout: post
title:  "[Basic] BlendTree_Direct"
subtitle:   ""
categories: Unity
comments: true
---

<br>

BlendTree_Direct을 실습해보자.

(1) 우리가 블랜드 트리를 적용할 부분은 얼굴이기 때문에 Base Layer에 Wait애니메이션을 기본 스테이트로 만들어준다.

(2) Avatar Mask를 생성하고 이름을 FaceAvatar라고 바꾼다.

(3) FaceAvatar에서 Transform탭을 열고 None이라고 되어 있는 곳에 현재 사용중인 유니티짱의 아바타를 넣어준다음 Toggle All을 눌러서 체크를 다 해제 해준다.

(4) 애니메이션을 적용 할 부분만 체크를 활성화 해준다.

![image](https://user-images.githubusercontent.com/101051124/159111114-d7dd9c36-df0d-4345-b9f9-96be793aa9cc.png)



(5) FaceLayer라는 새로운 레이어를 생성해주고 Weight를 1로 바꾸고 Mask에 FaceAvatar를 넣어준다.

(6) FaceLayer에 블랜드 트리를 하나 생성하고 더블클릭하여 내부로 들어간다.

(7) 파라미터를 4개 생성하고, 이름을 각각 angry, eye, sap, smile로 설정

(8) 블랜드 타입을 Direct로 설정

(9) Add Motionㅇ,로 4개의 Motion 생성

(10) Asset - unitychan! - Art - Animations - FaceAnimations 폴더의 애니메이션을 Motion에 등록

(11) Motion에 등록된 애니메이션과 매치되도록 파라미터 설정

![image](https://user-images.githubusercontent.com/101051124/159111348-6cb158c8-44fc-4f3d-bdb6-1bb2eb23bcc2.png)

이제 이 블랜드 트리를 제어 할 스크립트를 만들자.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerConstrollerDirect : MonoBehaviour
{
    private Animator anim;
    private void Awake()
    {
        anim = GetComponent<Animator>();
    }
    private void Update()
    {
        KeyEvent(0, KeyCode.Q, "angry"); //Q키를 누르면 angry 파라미터 값 증가
        KeyEvent(1, KeyCode.A, "angry"); //A키를 누르면 angry 파라미터 값 감소

        KeyEvent(0, KeyCode.W, "eye"); //W키를 누르면 eye 파라미터 값 증가
        KeyEvent(1, KeyCode.S, "eye"); //S키를 누르면 eye 파라미터 값 감소

        KeyEvent(0, KeyCode.E, "sap"); //E키를 누르면 sap 파라미터 값 증가
        KeyEvent(1, KeyCode.D, "sap"); //D키를 누르면 sap 파라미터 값 감소

        KeyEvent(0, KeyCode.R, "smile"); //R키를 누르면 smile 파라미터 값 증가
        KeyEvent(1, KeyCode.F, "smile"); //F키를 누르면 smile 파라미터 값 감소
    }
    private void KeyEvent(int type, KeyCode key, string parameter)
    {
        // key를 누르면 파라미터 값 증가/감소 시작
        if (Input.GetKeyDown(key))
        {
            string coroutine = type == 0 ? "parameterUp" : "parameterDown";
            StartCoroutine(coroutine,parameter);
        }
        //key를 떼면 파라미터 값 증가/감소 중지
        else if (Input.GetKeyUp(key))
        {
            string coroutine = type == 0 ? "parameterUp" : "parameterDown";
            StopCoroutine(coroutine);
        }
    }
    private IEnumerator parameterUp(string parameter)
    {
        //현재 파라미터 값을 받아온다
        float percent =anim.GetFloat(parameter);

        //파라미터 값을 증가시키는 코루틴이기 때문에 1이 될때까지 실행
        while (percent < 1)
        {
            percent += Time.deltaTime;
            anim.SetFloat(parameter, percent);

            yield return null;
        }
    }

    private IEnumerator parameterDown(string parameter)
    {
        //현재 파라미터 값을 받아온다
        float percent = anim.GetFloat(parameter);

        //파라미터 값을 감소시키는 코루틴이기 때문에 0이 될때까지 실행
        while (percent > 0)
        {
            percent-=Time.deltaTime;
            anim.SetFloat (parameter, percent);

            yield return null;
        }
    }
}
```

에디터로 돌아가 게임을 샐행하면 내가 설정해둔 키에 따라 플레이어의 표정이 변하는 걸 볼 수 있다.