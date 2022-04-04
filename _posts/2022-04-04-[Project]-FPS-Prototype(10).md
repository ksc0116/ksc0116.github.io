---
layout: post
title:  "[Project] FPS Prototype(10)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

오브젝트 풀을 아용해 적을 주기적으로 생성해보자.

# 적 캐릭터 오브젝트

(1) 적 캐릭터들에게 필요한 오브젝트들을 묶어서 보관하는 빈 오브젝트(Enemy)를 만든다.

(2) 적 캐릭터 몸통에 적용 할 Material(EnemyBody)을 만든다. Albedo Color(255, 255, 0, 255)

(3) 적 캐릭터 무기에 적용 할 Material(EnemyGun)을 만든다. Albedo Color(0, 0, 0, 255)

(4) 적 캐릭터 몸체 제작을 위해 Capsule(EnemyBody) 오브젝트를 Enemy오브젝트의 자식으로 만든다. 위치(0 / 1 / 0), EnemyBody 머테리얼 추가

(5) 적 캐릭터 무기 제작을 위해 Cube(EnemyGun) 오브젝트를 Enemy오브젝트의 자식으로 만든다. 위치(0 / 1 / 0.7), EnemyGun 머테리얼 추가, 크기(0.2 / 0.2 / 0.5)

(6) 적 캐릭터 오브젝트를 프리펩으로 만든다.

***

# 메모리 풀을 이용한 적 캐릭터 관리

(1) 적 캐릭터 등장 위치를 알리는 오브젝트에 적용 할 Material(EnemySpawnPoint)을 만든다. RenderingMode(Fade), AlbedoColor(255 / 0 / 0 / 150)

(2) 적 캐릭터 등장 위치 오브젝트 제작을 위해 Cube(EnemySpawnPoint) 오브젝트를 하나 만든다. 크기(1 / 2 / 1), EnemySpawnPoint 머테리얼 추가

(3) EnemySpawnPoint를 프리펩으로 만든다.

(4) 적 캐릭터 등장 위치를 제어하는 EnemySpawnpoint스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemySpawnPoint : MonoBehaviour
{
    [SerializeField]
    private float fadeSpeed = 4;
    private MeshRenderer meshRenderer;

    private void Awake()
    {
        meshRenderer = GetComponent<MeshRenderer>();
    }

    private void OnEnable()
    {
        StartCoroutine("OnFadeEffect");  
    }

    private void OnDisable()
    {
        StopCoroutine("OnFadeEffect");
    }

    private IEnumerator OnFadeEffect()
    {
        while (true)
        {
            Color color = meshRenderer.material.color;
            color.a = Mathf.Lerp(1, 0, Mathf.PingPong(Time.time * fadeSpeed, 1));
            meshRenderer.material.color = color;

            yield return null;
        }
    }
}
```

**float f = Mathf.PingPong(float t, float lenth);**

t값에 따라 0부터 length 사이의 값이 반환된다.

t값이 계속 증가할 때 length까지는 t값을 반환하고, t가 length보다 커졌을 때 순차적으로 0까지 -, length까지 +를 반복한다.

예를들어 length가 2이면 f는 0 1 2 1 0 1 2 1 0 1 2같이 된다.

(5) EnemtSpawnPoint오브젝트에 EnemySpawnPoint스크립트를 넣어준다.

(6) 메모리 플을 이용해 기둥과 적 캐릭터를 생성하고 활성/비활성화 하는 EnemyMemoryPool스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyMemoryPool : MonoBehaviour
{
    [SerializeField]
    private GameObject enemySpawnPointPrefab;           // 적이 등장하기 전 적의 등장 위치를 알려주는 프리팹
    [SerializeField]
    private GameObject enemyPrefab;     // 생성되는 적 프리팹
    [SerializeField]
    private float enemySpawnTime = 1;       // 적 생성 주기
    [SerializeField]
    private float enemySpawnLatency = 1;    // 타일 생성 후 적이 등장하기까지 대기 시간

    private MemoryPool spawnPointMemoryPool;        // 적 등장 위치를 알려주는 오브젝트 생성, 활성/비활성화 관리
    private MemoryPool enemyMemoryPool;      // 적 생성, 활성/비활성화 관리

    private int numberOfEnemiesSpawnedAtOne = 1;        // 동시에 생성되는 적의 숫자
    private Vector2Int mapSize = new Vector2Int(100, 100);

    private void Awake()
    {
        spawnPointMemoryPool = new MemoryPool(enemySpawnPointPrefab);
        enemyMemoryPool = new MemoryPool(enemyPrefab);

        StartCoroutine("SpawnTile");
    }

    private IEnumerator SpawnTile()
    {
        int currentNumber = 0;
        int maximumNumber = 50;

        while (true)
        {
            // 동시에 numberOfEnemiesSpawnAtOne 숫자만큼 생성되도록 반복문 사용
            for(int i = 0; i < numberOfEnemiesSpawnedAtOne; i++)
            {
                GameObject item = spawnPointMemoryPool.ActivatePoolItem();

                item.transform.position = new Vector3(Random.Range(-mapSize.x * 0.49f, mapSize.x * 0.49f), 1,
                                                                       Random.Range(-mapSize.y * 0.49f, mapSize.y * 0.49f));

                StartCoroutine("SpawnEnemy", item);
            }
            currentNumber++;
            if(currentNumber >= maximumNumber)
            {
                currentNumber = 0;
                numberOfEnemiesSpawnedAtOne++;
            }

            yield return new WaitForSeconds(enemySpawnTime);
        }
    }

    private IEnumerator SpawnEnemy(GameObject point)
    {
        yield return new WaitForSeconds(enemySpawnLatency);

        // 적 오브젝트를 생성하고, 적의 위치를 point의 위치로 설정
        GameObject item = enemyMemoryPool.ActivatePoolItem();
        item.transform.position=point.transform.position;

        // 타일 오브젝트 비활성화
        spawnPointMemoryPool.DeactivatePoolItem(point);
    }
}
```

(7) EnemyMemoryPool을 적용 할 빈 오브젝트(EnemyMemoryPool)을 만든다.

(8) 각 프리팹을 할당해준다.

이제 게임을 실행하면 아래와 같이 적이 생성되는 모습을 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/161457782-4c656cb2-f6fa-41b9-ba04-ea87974c3611.png)