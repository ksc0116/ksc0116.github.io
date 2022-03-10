---
layout: post
title:  "[Basic] Time에 관하여"
subtitle:   ""
categories: Unity
comments: true
---

<br>

Time에 대해서 공부를 하였는데 내가 모르는 개념이 너무나 많았다. 사실 내가 알고 있는거라고는 deltaTime뿐이긴 했다. 이제부터 내가 공부한 Time에 관한 모든걸 쓰겠다.

# time scale

time sclae을 직역하면 시간의 크기라는 뜻이다. 이 값을 조절하면 시간이 빠르게 혹은 느리게 흘러가게 할 수 있다. 예시 코드를 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class timesclae : MonoBehaviour
{
    float timer;    
    void Start()
    {
        timer = 0f;
    }

    void Update()
    {
        timer+=Time.deltaTime;
        //오브젝트의 위치를 Sin함수 값에 따라 바뀌게 함
        transform.position=  new Vector3(Mathf.Sin(timer) *10f ,0f,0f);

        if (Input.GetKey(KeyCode.UpArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale+0.01f,0f,2f);
        }
        if (Input.GetKey(KeyCode.DownArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale - 0.01f, 0f, 2f);
        }
        if (Input.GetKey(KeyCode.Alpha1))
        {
            Time.timeScale = 1f;
        }
    }
}

```

위의 코드는 방향키 위쪽을 누르면 timeScale이 커지고 아래쪽을 누르면 timeScale이 작아진다. 그리고 숫자 1을 누르면 timeScale을 1로 돌려서 원래 속도로 돌아오게 한다. 실행 결과를 확인해보면 0이 되었을 때 오브젝트는 멈추고 timeScale값이 커지면 커질수록 속도가 빨라지느게 확인이 된다. 이처럼 timeScale값을 조절해서 슬로우 모션등을 연출 할 수 있다.

# unscaled delta time

unscaled delta time은 크기가 변하지 않는 deltaTime을 의미한다. 즉, timeScale의 영향을 받지 않는다는 말이다.

예시 코드를 먼저 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class unscaled : MonoBehaviour
{
    float timer;
    void Start()
    {
        timer = 0f;
    }

    void Update()
    {
        timer += Time.unscaledDeltaTime;
        //오브젝트의 위치를 Sin함수 값에 따라 바뀌게 함
        transform.position = new Vector3(Mathf.Sin(timer) * 10f, 2f, 0f);
    }
}

```

이 스크립트를 컴포넌트로 가지고 있는 게임 오브젝트의 위치는 timeScale의 영향을 받지 않는다. 확인을 하기 위해서 sphere 게임 오브젝트를 추가하고 위 스크립트를 컴포넌트로 추가한 다음 실행시키면

![image](https://user-images.githubusercontent.com/101051124/157660185-c313aba9-0bde-44e8-8cb4-5ffb1b4e7f95.png)

아무리 timeScale값이 변해도 sphere 게임 오브젝트가 빨라지거나 느려지지 않는다.

이처럼 timeScale, deltaTime, unscaledDeltaTime을 적절히 혼합하여 사용하면 적들은 느려지고 플레이어는 빨라지는 연출을 구현할 수 있다.

# time

Time.time은 게임이 시작하고 나서 흐른 시간을 보여주는 프로퍼티이다. 예시 코드를 보자

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class time : MonoBehaviour
{
    [SerializeField]
    Text timeText;
    void Update()
    {
        timeText.text=string.Format("Time : {0}",Time.time);
    }
}

```

위 코드는 게임이 실행되고 나서 흐른 시간을 계속해서 화면에 Text로 뿌려준다. 실행하면 아래와 같다.

![image](https://user-images.githubusercontent.com/101051124/157661529-f6d8b5de-3ccd-4006-b706-94f028a8b067.png)

# realtimeSinceStartup

time과 같이 게임이 시작되고 나서부터 흐른 시간을 보여준다. 그렇다면 time이랑은 무슨 차이가 있을까? 예시 코드와 실행 결과를 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class time : MonoBehaviour
{
    [SerializeField]
    Text timeText;
    void Update()
    {
        timeText.text = string.Format("Time : {0}\nrealtimeSinceStartup : {1}", Time.time, Time.realtimeSinceStartup);

        if (Input.GetKey(KeyCode.UpArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale + 0.01f, 0f, 2f);
        }
        if (Input.GetKey(KeyCode.DownArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale - 0.01f, 0f, 2f);
        }
        if (Input.GetKey(KeyCode.Alpha1))
        {
            Time.timeScale = 1f;
        }
    }
}

```

위 코드는 time과 realtimeSinceStartup을 Text로 화면에 뿌려주고 방향키를 입력받아  timeScale의 값을 조절한다. 실행결과를 보면

![image](https://user-images.githubusercontent.com/101051124/157662585-531a205e-b4c2-47f8-bd52-a1a3ab13b61c.png)

time은 timeScale의 영향을 받지만 realtimeSinceStartup은 영향을 받지 않는 모습을 볼 수 있다. 그리고 realtimeSinceStartup은 게임을 중간에 멈춘 시간까지도 포함한다. 즉 게임을 멈추었을 때 시간이 7초였고 멈춰있던 시간이 5초라면 게임을 다시 실행하였을 때 realtimeSinceStartup의 값은 12가 된다. 이처럼 realtimeSinceStartup은 플레이어가 게임을 한 시간에 따라 업적을 줄 수도 있고, 너무 오래 게임을 하였다면 경고문을 주는데 사용할 수 있다.

# timeSinceLevelLoad

timeSinceLevelLoad는 마지막으로 씬이 로드된 이후로 흐른 시간을 의미한다. 예시를 먼저 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class time : MonoBehaviour
{
    [SerializeField]
    Text timeText;
    void Update()
    {
        timeText.text = string.Format("Time : {0}\nrealtimeSinceStartup : {1}\ntimeSinceLevelLoad : {2}", Time.time, Time.realtimeSinceStartup,Time.timeSinceLevelLoad);

        if (Input.GetKey(KeyCode.UpArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale + 0.01f, 0f, 2f);
        }
        if (Input.GetKey(KeyCode.DownArrow))
        {
            Time.timeScale = Mathf.Clamp(Time.timeScale - 0.01f, 0f, 2f);
        }
        if (Input.GetKey(KeyCode.Alpha1))
        {
            Time.timeScale = 1f;
        }
        if (Input.GetKey(KeyCode.Space))
        {
            SceneManager.LoadScene(0);
        }
    }
}

```

위 코드는 스페이스바를 누르면 현재 씬을 다시 불러오도록 해놨다. 그리고 시간이 흐른 뒤 스페이스바를 누르면 timeSinceLevelLoad의 값이 0이 되는걸 확인이 가능하다. 이 프로퍼티를 이용하면 플레이어가 던전 씬에 들어가서 클리어하는데 걸리는 시간을 측정하는데 이용이 가능하다.

