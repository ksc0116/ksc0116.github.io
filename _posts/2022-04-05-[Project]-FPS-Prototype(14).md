---
layout: post
title:  "[Project] FPS Prototype(14)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

부서지는 드럼통, 과녁을 제작 해보자.

# 부서지는 드럼통

* 오브젝트가 부서지는 효과
  * 코드를 이용해 오브젝트의 색상정보를 가지고 있는 조각 오브젝트들을 생성
  * 부서지기 전과 부서진 이후의 오브젝트를 미리 생성해두고, 부서져야할 때 오브젝트를 교체하는 방식

(1) 부서지는 드럼통의 부서지기 전 오브젝트 제작을 위해 explosive_barrel을 하이어라키창으로 넣고 이름을 DestructibleBarrel로 바꾼다.위치(0 / 1 / 0), 

크기(3 / 3 /3), 충돌처리를 위해 Capsule콜라이더, Rigidbody 컴포넌트를 추가한다.

(2) Capsule콜라이더 : Center(0 / 0 / 0), Radius : 0.16, Height : 0.55

(3) 부서지는 드럼통 오브젝트를 프리팹으로 만들고 태그를 InteractionObject로 바꾼다.

(4) 부서지는 드럼통 오브젝트의 부서진 후 오브젝트 제작을 위해 explosive_barrel_destructible을 하이러라키창으로 넣고 이름을 DestructibleBarrelPieces로 바꾼다. 위치 (0 / 1 / 0), 크기(3 / 3 / 3)

(5) 드럼통의 조각들이 충돌이 가능하도록 DestructibleBarrelPieces의 자식오브젝트 전부 MeshCollider, Rigidbody 컴포넌트를 추가한다.

(6) MeshCollider(Convex : true), Rigidbody(Mass : 100)으로 설정하고 프리팹으로 만든다.

(7) 맵에 임의로 부서지는 드럼통을 2개 배치한다.

(8) 부서지는 드럼통을 제어하는 DestructibleBarrel스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DestructibleBarrel : InteractionObject
{
    [Header("Destructible Barrel")]
    [SerializeField]
    private GameObject destructibleBarrelPieces;

    private bool isDestroyed = false;

    public override void TakeDamage(int damage)
    {
        currentHP -= damage;
        
        if(currentHP<=0 && isDestroyed == false)
        {
            isDestroyed = true;

            Instantiate(destructibleBarrelPieces,transform.position,transform.rotation);

            Destroy(gameObject);
        }
    }
}
```

(9) DestructibleBarrel프리팹에 DestructibleBarrel스크립트를 추가한다. 그리고 컴포넌트들을 설정한다.

위 방법은 부서지기 전과 부서진 이후의 오브젝트를 미리 생성해두고, 부서져야할 때 오브젝트를 교체하는 방식이다.

이제 게임을 실행해서 부서지는 드럼통을 공격해보면 드럼통이 부서지는 모습을 볼 수 있다.

***

# 과녁

(1)  과녁으로 사용할 빈 오브젝트(Target)을 만든다. BoxCollider, AudioSource컴포넌트를 추가한다.

(2) Target=> 위치(0 / 2 / 5), BoxCollider( Center(0 / 0.69 / 0), Size(0.7 / 1.36 / 0.1) ), AudioSource(Play On Awake(false))

(3) 화면에 보여지는 과녁 오브젝트 제작을 위해 taerget FBX를 Target오브젝트의 자식으러 넣어준다. 크기(2 / 2 / 2)

(4) 과녁 오브젝트를 프리팹으로 만든다. Target프리팹의 태그를 InteractionObject로 바꾼다.

(5) 과녁 오브젝트를 제어하는 Target스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Target : InteractionObject
{
    [SerializeField]
    private AudioClip clipTargetUp;
    [SerializeField]
    private AudioClip clipTargetDowm;
    [SerializeField]
    private float targetUpDelayTime = 3;

    private AudioSource audioSource;
    private bool isPossibleHit = true;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }

    public override void TakeDamage(int damage)
    {
        currentHP-=damage;

        if(currentHP <= 0 && isPossibleHit == true)
        {
            isPossibleHit = false;

            StartCoroutine("OnTargetDown");
        }
    }

    private IEnumerator OnTargetDown()
    {
        audioSource.clip = clipTargetDowm;
        audioSource.Play();

        yield return StartCoroutine(OnAnimation(0, 90));

        StartCoroutine("OnTargetUp");
    }

    private IEnumerator OntargetUp()
    {
        yield return new WaitForSeconds(targetUpDelayTime);

        audioSource.clip = clipTargetUp;
        audioSource.Play();

        yield return StartCoroutine(OnAnimation(90, 0));

        isPossibleHit=true;
    }

    private IEnumerator OnAnimation(float start, float end)
    {
        float percent = 0;
        float current = 0;
        float time = 1;

        while (percent < 1)
        {
            current += Time.deltaTime;
            percent=current/time;

            transform.rotation = Quaternion.Slerp(Quaternion.Euler(start, 0, 0), Quaternion.Euler(end, 0, 0), percent);

            yield return null;
        }
    }
}
```

(6) Traget프리팹에 Target스크립트를 넣어주고 컴포넌트들을 할당해준 다음 게임을 실행하면 과녁을 공격했을 때 과녁이 Down 되었다 Up 되는 모습을 볼 수 있다.