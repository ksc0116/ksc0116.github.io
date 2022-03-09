---
layout: post
title:  "[Skill] 플레이어가 마우스 위치 바라보게 하기"
subtitle:   ""
categories: Unity
comments: true
---

<br>

플레이어가 마우스 위치 바라보게 하는 방법을 살펴보자.

우선 코드부터 보자!

```csharp
    void Update()
    {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        RaycastHit hit;
        if(Physics.Raycast(ray,out hit))
        {
            Vector3 mousePos = new Vector3(hit.point.x, transform.position.y, hit.point.z);
            transform.forward= mousePos - transform.position;
        }
    }
```



위 코드에서 가장 중요한 점은 hit.point를 바로 사용하는게 아니라 새로운 Vector3로 만들어서 사용하는 것이다.

자세히보면 mousePos의 y좌표에 현재 오브젝트의 y좌표를 넣은걸 확인이 가능하다. 현재 오브젝트의 y좌표를 넣는 이유는 현재 오브젝트가 hit.point를 바로 바라보게 된다면 현재 오브젝트보다 더 높거나 낮은 곳에 hit된 곳을 바라볼 때 오브젝트가 기우는 현상을 방지하기 위함이다. 그리고 현재 오브젝트의 forward를 mousePos에서 현재오브젝트의 위치를 빼서 현재 오브젝트에서 mousePos까지의 방향을 구해 바꿔준다. 그럼 현재 오브젝트가 마우스의 위치를 바라보게 된다! 

<br>

**이 글을 마치면서 다시한번 말하지만 여기서 가장 중요한 점은 hit.point를 그대로 사용하는것이 아닌**

**현재 오브젝트의 y좌표를 넣어서 높이를 맞춰주는 것이다.**

