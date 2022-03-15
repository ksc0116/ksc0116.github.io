---
layout: post
title:  "[Basic] Physics클래스"
subtitle:   ""
categories: Unity
comments: true
---

<br>

Physics클래스에서 가상의 광선으로 충돌 체크하는 Raycast와 주변의 콜라이더를 체크하는 Overlap을 활용해보자.

우선 다음과 같이 Enemy오브젝트와 Player오브젝트를 만들어 두고 Enemy가 Player를 바라보게끔 y축 기준으로 180도 회전시키고 실제로 Enemy가 Player오브젝트를 바라보고 있는지 확인해보자.여기서 사용 할 방법은 Enemy가 가상의 광선을 쏴서 Player가 맞으면 바라보고 있다고 하겠다. 

![image](https://user-images.githubusercontent.com/101051124/158342924-114f96bc-73f3-45bc-bc17-99d9991c932a.png)

이제 스크립트를 작성해보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    void Update()
    {
        RaycastHit hit; //레이를 쐈을 때 맞은놈의 정보를 저장해두는 용도
        if(Physics.Raycast(transform.position,transform.forward,out hit, 10f))
        {
            Debug.Log(hit.transform.name);//맞은 놈의 이름을 출력
        }
    }
}
```

위 스크립트를 보면 설명할 게 있다. 바로 **Physics.Raycast()**이다. 가상의 광선을 쏘게하는 함수인데 들어가는 인자가 많다. 하지만 간단하게 설명해보겠다. Physics.Raycast()에서 ()안에 들어 갈 인자는 순서대로

(쏘기 시작 할 위치, 방향, out 광선에 누군가 맞았을 때 정보를 저장할 Raycasthit 객체, 거리)이다. 위와같이 작성한 코드를 해석하면 내 위치에서 내 앞쪽 방향으로 누군가 광선에 맞으면 hit에 저장하고 거리는 10만큼 쏘겠다는 뜻이다. 위 스크립트를 프로젝트에서 Enemy에 부착하고 실행시키면 다음과 같은 로그가 출력된다.

![image](https://user-images.githubusercontent.com/101051124/158344774-c383d998-2095-499b-8ebc-d1c6d7ad2d88.png)

Enemy앞에  Player가 있고 그래서 Enemy가 쏜 광선에 맞았기 때문이다.

이번엔 좀 더 응용해서 Enemy와 Player사이에 방해물이 있다고 가정하겠다.

![image-20220315181831969](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220315181831969.png)

그럼 다음과 같이 광선에 맞은 놈의 정보가 Cube가 되는걸 확인 가능하다. 이제 방해물이 있어도 그걸 무시하고 Player가 맞게끔 만들어보자. 여러가지 방법이 있는데 우선 간단하게

![image](https://user-images.githubusercontent.com/101051124/158346171-83632f05-df1c-4c8a-b90b-193971a9dc85.png)

Cube의 Layer를 Ignore Raycast로 설정해주면 광선에 맞지 않게 된다.

![image](https://user-images.githubusercontent.com/101051124/158346339-22052122-c39f-4a86-8e25-85bd6f244070.png)

위와 같이 말이다. 다른 방법은 LayerMask를 사용하는 것이다, 스크립트를 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] private LayerMask layerMask; //레이를 쐈을 때 특정 Layer만 맞도록 설정가능하다.
    void Update()
    {
        RaycastHit hit; //레이를 쐈을 때 맞은놈의 정보를 저장해두는 용도
        if(Physics.Raycast(transform.position,transform.forward,out hit, 10f, layerMask))
        {
            Debug.Log(hit.transform.name);
        }
    }
}
```

코드를 분석하면 Physics.Raycast()에서 마지막에 lyerMask가 들어간게 확인이 된다. 저렇게 Physics.Raycast()의 마지막 인자에 layerMask가 들어가면 해석이 조금 달라진다. 위 코드를 해석하면

 **내 위치에서 앞쪽 방향으로 10만큼 쏘는데 특정 레이어(layerMask)만 광선에 맞게하고 맞으면 hit에 저장한다.**이다. 이제 다시 Enemy오브젝트를 눌러서 컴포넌트를 확인하면 다음과 같이 보인다.

![image](https://user-images.githubusercontent.com/101051124/158347498-2d9a16d1-9559-4eed-9407-26c7b49245f8.png)

기본상태는 Nothig이다. 즉, 아무도 맞지 않게 하는 것이다. 우선 특정 레이어만 맞게 하기 위해서 사전 작업이 필요하다.

![image](https://user-images.githubusercontent.com/101051124/158347869-9cd1e3bb-fd2b-4114-be97-81856d620140.png)

위와 같이 Layer를 추가해주고 각각 오브젝트에 맞게 할당해준다. 그 다음 Test 컴포넌트의 필드에서 Layer Mask를 Player로 바꿔주면 Player만 광선에 맞게 된다. 다음 화면을 잘 봐보자.

![image](https://user-images.githubusercontent.com/101051124/158348253-6c9b5dd9-cdb7-4f95-8316-a953bdcaa572.png)

이제 여태까지 했던것들을 또 응용해서 Enemy가 Player를 바라보고 있으면 총알을 발사하게끔 만들어보자 물론 방해물이 업을 때 만 총알을 쏜다. 그러기 위해서는 우선 총알 오브젝트를 만들어야한다.

![image](https://user-images.githubusercontent.com/101051124/158348866-eeecf64c-6719-43a8-9059-c0a59131a283.png)

다음과 같이 Sphere오브젝트를 생성하고 크기 조절하고 Material바꾸고 이름까지 바꿔 준 다음에 Bullet에 대한 스크립트를 만들어보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bullet : MonoBehaviour
{
    [SerializeField] private float speed; //총알이 날아가는 속도
    [SerializeField] private float damage; //총알의 데미지
    void Update()
    {
        transform.position+=transform.forward*speed*Time.deltaTime; //1초에 speed만큼 정면으로 날아감
    }

    private void OnTriggerEnter(Collider other)
    {
        Debug.Log(other.transform.name + "에게 데미지 " + damage + "을 입혔습니다.");
        Destroy(gameObject);
    }
}
```

위와 같이 스크립트를 만들고 Bullet 오브젝트에 추가하고 속도와 데미지를 설정한 다음 프리펩으로 만들어 주자. 이제 Enemy가 Player를 바라보고 있을 때 총알을 만들어서 발사하게끔 만들어 보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;
    private float createTime=1f; //총알 생성 주기
    private float curCreateTime=0;
    void Update()
    {
        curCreateTime += Time.deltaTime;
        if(curCreateTime >= createTime)
        {
            curCreateTime = 0;
            RaycastHit hit; //레이를 쐈을 때 맞은놈의 정보를 저장해두는 용도
            if (Physics.Raycast(transform.position, transform.forward, out hit, 10f))//누군가 맞으면 true
            {
                if (hit.transform.tag == "Player") //누군가 맞았는데 그 맞은 놈의 tag가 Player면 true
                {
                    //bulletPrefab에 등록된 게임 오브젝트를 내 위치에서 ,내가 상대방을 바라보는 방향으로 회전시켜서 만듬
                    Instantiate(bulletPrefab, transform.position, Quaternion.LookRotation(hit.transform.position - transform.position));
                }
            }
        }
    }
}
```

위와 같이 스크립트를 만들고 프로젝트에서 Player의 tag를 Player로 바꿔주고 Test 컴포넌트의 필드에 미리 만들어둔 총알 프리펩을 할당하고 실행해보면

![image](https://user-images.githubusercontent.com/101051124/158352423-30208cb2-59e1-4372-85c4-972990354a74.png)

위와 같이 방해물이 있을 때는 총알을 발사하지 않다가 

![image](https://user-images.githubusercontent.com/101051124/158352570-30aaacbe-3d90-4b6d-8471-c350ed84f048.png)

방해물이 사라지고 설정해둔 1초뒤에 Player를 향해 총알을 발사하는 모습을 볼 수 있다. 그리고 Bullet 스크립트에서 OnTriggerEnter()이벤트를 설정해놨는데 그게 실행이 되려면 Player오브젝트에는 Rigidbody컴포넌트가 있어야 한다. 그리고 총알이 Player한테 맞으면 다음과 같은 로그가 출력 된다.

![image](https://user-images.githubusercontent.com/101051124/158352906-8a2cca92-a81d-4f88-b506-18d130dc63e8.png)

하지만 문제가 있다. 분명 1초에 한번씩 총알을 발사하게 하려했는데 총알 1개가 만들어지고 Player한테 맞은 다음 다음 총알이 생성된다. 그 이유는 현재 위 사진과 같은 상황에서 Enemy가 총알을 생성하고 발사하는데 Enemy가 쏘고 있는 광선에 총알이 맞고 있기 때문이다. 위 문제를 해결하기 위해 LayerMask를 Bullet 빼고 전부 다 맞도록  설정하면 총알이 계속해서 1초마다 나가게 될 것이다. 스크립트를 다시 아래와 같이 수정하다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;
    [SerializeField] private LayerMask layerMask;
    private float createTime=1f; //총알 생성 주기
    private float curCreateTime=0;
    void Update()
    {
        curCreateTime += Time.deltaTime;
        if(curCreateTime >= createTime)
        {
            curCreateTime = 0;
            RaycastHit hit; //레이를 쐈을 때 맞은놈의 정보를 저장해두는 용도
            if (Physics.Raycast(transform.position, transform.forward, out hit, 10f, layerMask))//설정해둔 layer가 맞으면
            {
                if (hit.transform.tag == "Player") //누군가 맞았는데 그 맞은 놈의 tag가 Player면 true
                {
                    //bulletPrefab에 등록된 게임 오브젝트를 내 위치에서 ,내가 상대방을 바라보는 방향으로 회전시켜서 만듬
                    Instantiate(bulletPrefab, transform.position, Quaternion.LookRotation(hit.transform.position - transform.position));
                }
            }
        }
    }
}
```

그리고 프로젝트로 돌아가 총알 프리펩에 Bullet 레이어를 추가해주고

![image-20220315190623531](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220315190623531.png)

위와 같이 Bullet레이어만 맞지 않게 설정해주자. 그리고 실행하면

![image-20220315190709330](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220315190709330.png)

위와 같이 1초마다 총알이 생성되는걸 확인할 수 있다. 이제 새로운 시스템을 만들어보자, 우선 화면을 다음과 같이 세팅 해보자.

![image-20220315191126451](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220315191126451.png)

우리는 이제 Player가 Enemy주변을 공전하게 만들고 Player가 Enemy주변의 특정 영역에 들어가면 Enemy가 Player를 바라보게 하고 총알을 발사하게 만들 것이다. Test 스크립트를 수정해보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] GameObject bulletPrefab; //생성할 총알 프리펩
    private float createTime = 1f;
    private float curCreateTime = 0;
    void Update()
    {
        //OverlapSphere 구 모양으로 영역을 생성하고 그 영역안에 있는 모든 콜라이더의 정보를 col에 저장
        Collider[] col = Physics.OverlapSphere(transform.position,5f); //내 위치에서 구 모양 영역을 만듬 (반지름 5)

        if(col.Length > 0)//콜라이더 정보가 1개이상이라도 있으면
        {
            for (int i = 0; i < col.Length; i++) //콜라이더의 갯수만큼 반복
            {
                Transform target = col[i].transform;

                if (target.tag == "Player") //Player tag를 가지고 있다면
                {
                    //내가 타겟을 바라보는 방향을 계속해서 구해줌
                    Quaternion rotation = Quaternion.LookRotation(target.position - transform.position);
                    transform.rotation = rotation; //나의 방향을 타겟쪽으로 바꿈
                    
                    curCreateTime += Time.deltaTime;

                    if (curCreateTime > createTime)
                    {
                        Instantiate(bulletPrefab, transform.position, rotation); //타겟의 방향으로 총알을 만듬
                        curCreateTime = 0;
                    }
                }
            }
        }
    }
}
```

이렇게 바꾸고 Player가 Enemy주변을 공전해야하기 때문에 Player 스크립트를 만들어서 Player 오브젝트에 추가해주자. 스크립트는 아래와 같다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player : MonoBehaviour
{
    [SerializeField] private float speed;
    [SerializeField] private Transform target;

    void Update()
    {
        //공전하게 만든다 타겟의 위치를 중심으로 y축 기준으로 speed만큼의 속도로
        transform.RotateAround(target.position, Vector3.up,speed*Time.deltaTime);
    }
}
```

그리고 실행해주면 Player가 Enemy주변을 공전하게 되고 특정 영역에 들어가면 총알이 발사가 된다. 확인해보자.

![image](https://user-images.githubusercontent.com/101051124/158358860-abcad111-701b-4a96-96b8-c7030475917a.png)

주변을 돌지만 총알이 발사되지 않는다. 이제 영역 안으로 들어가 보겠다.

![image](https://user-images.githubusercontent.com/101051124/158359054-5256bb70-dddc-4d10-8cce-d5c416b7e6a5.png)

총알이 발사되는 모습을 확인할 수 있다. 그리고 새로운 함수를 소개하겠다. 바로 **Physics.IgnoreCollision()**이다.  Physics.IgnoreCollision()는 인자로 콜라이더 2개를 받는데 인자로 받은 2개의 콜라이더 끼리의 충돌은 무시하게 만들어준다. 현재 시스템은 총알이 플레이어한테 맞으면 데미지를 주고 파괴되는 형식이다. 이제 플레이어와 총알의 충돌을 무시하게끔 만들어보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] GameObject bulletPrefab; //생성할 총알 프리펩
    private float createTime = 1f;
    private float curCreateTime = 0;
    void Update()
    {
        //OverlapSphere 구 모양으로 영역을 생성하고 그 영역안에 있는 모든 콜라이더의 정보를 col에 저장
        Collider[] col = Physics.OverlapSphere(transform.position,5f); //내 위치에서 구 모양 영역을 만듬 (반지름 5)

        if(col.Length > 0)//콜라이더 정보가 1개이상이라도 있으면
        {
            for (int i = 0; i < col.Length; i++) //콜라이더의 갯수만큼 반복
            {
                Transform target = col[i].transform;

                if (target.tag == "Player") //Player tag를 가지고 있다면
                {
                    //내가 타겟을 바라보는 방향을 계속해서 구해줌
                    Quaternion rotation = Quaternion.LookRotation(target.position - transform.position);
                    transform.rotation = rotation; //나의 방향을 타겟쪽으로 바꿈
                    
                    curCreateTime += Time.deltaTime;

                    if (curCreateTime > createTime)
                    {
                        //총알을 생성하자마자 bullet객체로 만듬
                        GameObject bullet= Instantiate(bulletPrefab, transform.position, rotation); //타겟의 방향으로 총알을 만듬
                        //총알의 콜라이더와 플레이어의 콜라이더의 충돌을 무시하게끔 만든다.
                        Physics.IgnoreCollision(bullet.GetComponent<Collider>(),target.GetComponent<Collider>());
                        curCreateTime = 0;
                    }
                }
            }
        }
    }
}
```

이러면 총알과 플레이어의 충돌은 무시가 된다.