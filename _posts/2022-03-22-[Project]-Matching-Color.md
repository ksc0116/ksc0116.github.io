---
layout: post
title:  "[Project] Matching Color"
subtitle:   ""
categories: Unity
comments: true
---

<br> 계속해서 전진해오는 큐브의 색상에 나의 큐브 색상을 맞추는 게임이다.

# 프로젝트 기본 설정

(1) Main Camera의 위치 : 0, 3, -6 회전 : 10, 0, 0 / Camera컴포넌트 Clear Flags:Solid Color BackGround:0,0,0,0

(2) 오른쪽 하단의 버튼을 눌러 Lighting 뷰에서 Auto Generate를 활성화해준다.

(3) 바닥으로 사용 할 Cube를 만들고 이름을 Ground로 바꾼 다음 위치:0, -1.6, 25 크기:4, 0.1, 60

(4) 단순히 보여지는 용도로만 사용되기 때문에 박스 콜라이더 컴포넌트는 삭제한다.

그럼 다음과 같은 화면이 된다.

![image](https://user-images.githubusercontent.com/101051124/159385057-120d01a8-958c-4789-9103-9821741bb773.png)

# 큐브셋 오브젝트 자동 생성

(1) 큐브셋의 이동을 제어하는 Movement스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 0.0f;
    [SerializeField]
    private Vector3 moveDirection = Vector3.zero;

    // 이동방향이 결정되면 알아서 움직이도록 함
    private void Update()
    {
        transform.position += moveDirection * moveSpeed * Time.deltaTime;
    }

    //외부에서 매개변수로 이동방향을 설정
    public void MoveTo(Vector3 direction)
    {
        moveDirection = direction;
    }
}
```

(2) 특정위치에 도달했을 때 오브젝트를 삭제하는 PositionAutoDestroyer스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PositionAutoDestroyer : MonoBehaviour
{
    [SerializeField]
    private Vector3 destroyPosition;

    private void LateUpdate()
    {
        if((destroyPosition - transform.position).sqrMagnitude < 0.1f)
        {
            Destroy(gameObject);
        }
    }
}
/* 화면 밖으로 나갈 수 있는 오브젝트에 부탁해서 사용
 * 오브젝트가 일정 범위 밖으로 나가면 삭제
 */
```

**Vector3.sqrMagnitude** : 계산된 거리의 제곱 값 반환(루트 연산을 하지 않아 연산속도가 빠름). 정확한 거리가 아닌 얼마나 가까운지와 같은 근사치가 필요할 때 사용

**Vector3.Distance(Vector3,Vector3), Vector3.magnitude** : 정확한 거리 값을 계산

(3) 큐브셋 프리펩에 Movement스크립트, PositionAutoDestroyer스크립트를 추가하고,

Movement -> moveSpeed:5, moveDirection:0, 0, -1

PositionAutoDestroyer -> DestroyPosition:0,0,-2

(4) 큐브셋 오브젝트를 자동으로 생성하는 CubeSpawner스크립트를 만들어 준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeSpawner : MonoBehaviour
{
    // 현재 스테이지에서 사용 가능한 큐브의 색상 수
    [SerializeField]
    private Color[] cubeColors;
    public Color[] CubeColors => cubeColors;

    // 큐브셋 생성을 위한 변수
    [SerializeField]
    private GameObject cubesetPrefab;       // 큐브셋 프리펩
    [SerializeField]
    private Transform spawnPoint;               // 큐브셋 생성 위치
    [SerializeField]    
    private float spawnTime = 1.0f;             // 큐브셋 생성 주기

    private void Awake()
    {
        StartCoroutine("SpawnCubeSet");
    }

    private IEnumerator SpawnCubeSet()
    {
        while (true)
        {
            GameObject clone =Instantiate(cubesetPrefab,spawnPoint.position,Quaternion.identity);
            // 큐브셋 오브젝트의 자식으로 있는 9개 큐브의 MeshRenderer 컴포넌트 정보
            MeshRenderer[] renderers=clone.GetComponentsInChildren<MeshRenderer>();
            // 9개 큐브의 색상을 임의로 성정 ( 현재 스테이지에서 사용 가능한 큐브 색상으로 = CubeColors)
            for(int i = 0; i < renderers.Length; ++i)
            {
                int index = Random.Range(0, CubeColors.Length);
                renderers[i].material.color = CubeColors[index];
            }
            yield return new WaitForSeconds(spawnTime);
        }
    }
}
```

(5) 빈 오브젝트를 생성해 이름을 CubeSpawner라고 바꾼뒤 CubeSpawner스크립트를 추가한다.

(6) CubeSpawner오브젝트의 자식으로 빈 오브젝트를 하나 더 생성해 이름을 SpawnPoint로 바꿔주고 위치:0,0,45로 설정해준다.

(7) CubeSpawner오브젝트의 컴포넌트로 있는 CubeSpawner스크립트의 변수를 다음과 같이 설정한다.

![image](https://user-images.githubusercontent.com/101051124/159387573-a5029c9a-caec-430f-9ba6-61caf7d82286.png)

그리고 게임을 실행하면 다음과 같이 큐브셋에 색상이 랜덤으로 적용되어 1초마다 생성되는걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159387994-032df4f4-86f2-47f3-b437-b9ca99b24820.png)

(8) 스폰타임이 너무 빠르기 때문에 5초로 바꿔준다.

# 터치 가능한 큐브셋

(1) TouchCubeset 프리펩을 하이어라키창에 넣어준다.

(2) 터치 큐브의 색상 변경, 큐브간 충돌을 검사하는 CubeController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeController : MonoBehaviour
{
    private CubeSpawner cubeSpawner;
    private MeshRenderer meshRenderer;
    private int colorIndex;

    public void SetUp(CubeSpawner cubeSpawner)
    {
        this.cubeSpawner=cubeSpawner;

        meshRenderer = GetComponent<MeshRenderer>();
        meshRenderer.material.color = this.cubeSpawner.CubeColors[0];
        colorIndex = 0;
    }

    public void ChangeColor()
    {
        // 다음 큐브 색상으로 인덱스를 변경
        if (colorIndex < cubeSpawner.CubeColors.Length - 1)
        {
            colorIndex++;
        }
        else
        {
            colorIndex=0;
        }

        // 실제 큐브 색상 변셩
        meshRenderer.material.color=cubeSpawner.CubeColors[colorIndex];
    }
}
```

(3) 마우스클릭으로 특정 오브젝트를 검출하는 Objectdetector스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

// 유니티 이벤트 시스템
// 클래스 객체명.Invoke() : 메소드 호출
// 클래스 객체명.AddListener(호출할 메소드);
// 가장 대표적인 유니티 이벤트 시스템은 Button UI의 OnClick()이다.
[System.Serializable]
public class RaycastEvent : UnityEvent<Transform> { }
public class ObjectDetector : MonoBehaviour
{
    [HideInInspector]
    public RaycastEvent  raycastEvent=new RaycastEvent();

    [SerializeField]
    private LayerMask layerMask;
    private Camera mainCamera;
    private Ray ray;
    private RaycastHit hit;

    private void Awake()
    {
        // "MainCamera" 태그를 가지고 있는 오브젝트 탐색 후 Camera 컴포넌트 정보 전달
        // GameObject.FindGameObjectWithTag("MainCamera").GetComponent<Camera>()와 동일
        mainCamera =Camera.main;
    }

    private void Update()
    {
        // 마우스 왼쪽 버튼을 눌렀을 때
        if (Input.GetMouseButtonDown(0))
        {
            // 카메라 위치에서 화면의 마우스 위치를 관통하는 광선 생성
            // ray.origin : 광선의 시작위치(=카메라 위치)
            // ray.direction : 광선의 진행방향
            ray = mainCamera.ScreenPointToRay(Input.mousePosition);

            // 2D 모니터를 통해 3D 월드의 오브젝트를 마우스로 선택하는 방법
            // 광선에 부딪히는 오브젝트를 검출해서 hit에 저장
            if(Physics.Raycast(ray, out hit, Mathf.Infinity, layerMask))
            {
                // 부딪힌 오브젝트의 Transform 정보를 매개변수로 이벤트 호출
                raycastEvent.Invoke(hit.transform);
                //raycastEvent.Invoke(); : 이벤트에 등록된 메소드들을 호출
                //이벤트 클래스를 정의 할 때 UnityEvent<Transform>이라고 작성했기 때문에 Transform 매개 변수를
                //넘겨줄 수 있다.
            }
        }
    }
}
```

(4) 검출된 오브젝트의 색상 변경 메소드를 호출하는 CubeSelector스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeSelector : MonoBehaviour
{
    private ObjectDetector objectDetector;
    private void Awake()
    {
        objectDetector = GetComponent<ObjectDetector>();

        // 레이어가 "Touchable"인 오브젝트만 선택 가능하도록 ObjectDectector의 layerMask설정
        // ObjectDetector 클래스의 raycastEvent.Invoke(hit.transform);가 호출 될 때 SelectCube 호출
        objectDetector.raycastEvent.AddListener(SelectCube);
    }

    public void SelectCube(Transform hit)
    {
        // 선택된 오브젝트의 CubeController.ChageColor()를 호출해서 큐브 색상 변경
        hit.GetComponent<CubeController>().ChangeColor();
    }
}
```

(5) 이동 큐브셋과 터치 큐브셋의 색상 매칭을 검사하는 cubeChecker를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeChecker : MonoBehaviour
{
    [SerializeField]
    private CubeSpawner cubeSpawner;

    private CubeController[] touchCubes;        //터치 가능한 큐브셋의 큐브들

    private void Awake()
    {
        touchCubes=GetComponentsInChildren<CubeController>();

        for(int i = 0; i < touchCubes.Length; i++)
        {
            touchCubes[i].SetUp(cubeSpawner);
        }
    }
}
```

(6) 터치 큐브셋의 자식으로 있는 큐브들을 다 선택하여 CubeController스크립트를 넣어준다.

(7) 터치 큐브셋에 CubeSelector,ObjectDetector,CubeChecker를 넣어준다.

(8) Touchable레이어를 만들고 터치 큐브셋의 자식으로 있는 큐브들의 레이어를 Touchable로 바꾼다.

(9) Touchable레이어를 가진 오브젝트끼리 부딪히지 않도록 레이어간의 물리 충돌 설정을 해준다.

![image](https://user-images.githubusercontent.com/101051124/159397222-5f724446-984e-4f7d-bc1b-4922c91cf618.png)

# 큐브셋 색상 매칭 검사

(1) 각 큐브들의 맞은 갯수, 틀린 갯수를 바탕으로 성공,실패를 판단하기 위해 CubeCheck스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeChecker : MonoBehaviour
{
    [SerializeField]
    private CubeSpawner cubeSpawner;

    private CubeController[] touchCubes;        //터치 가능한 큐브셋의 큐브들

    private int correctCount = 0;
    private int incorrectCount = 0;

    public int CorrectCount
    {
        set => correctCount = Mathf.Max(0, value);
        get=> correctCount;
    }
    public int IncorrectCount
    {
        set=>incorrectCount = Mathf.Max(0, value);
        get => incorrectCount;
    }
    private void Awake()
    {
        touchCubes=GetComponentsInChildren<CubeController>();

        for(int i = 0; i < touchCubes.Length; i++)
        {
            touchCubes[i].SetUp(cubeSpawner,this);
        }
    }

    private void Update()
    {
        // 맞은 갯수 + 틀린 개수가 터치 가능한 큐브의 개수와 같을 때
        if (correctCount + incorrectCount == touchCubes.Length)
        {
            // 하나도 틀리지 않았을 때 : 성공
            if (IncorrectCount == 0)
            {
                Debug.Log("Success");
            }
            // 하나라도 틀렸을 때 : 실패
            else
            {
                Debug.Log("Failed");
            }
            CorrectCount = 0;
            IncorrectCount = 0;
        }
    }
}
```

(2) 터치 큐브에 부딪히는 이동 큐브와의 색상 매칭을 검사하기 위해 CubeController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeController : MonoBehaviour
{
    private CubeSpawner cubeSpawner;
    private CubeChecker cubeChecker;
    private MeshRenderer meshRenderer;
    private int colorIndex;

    public void SetUp(CubeSpawner cubeSpawner,CubeChecker cubeChecker)
    {
        this.cubeSpawner=cubeSpawner;
        this.cubeChecker=cubeChecker;

        meshRenderer = GetComponent<MeshRenderer>();
        meshRenderer.material.color = this.cubeSpawner.CubeColors[0];
        colorIndex = 0;
    }

    public void ChangeColor()
    {
        // 다음 큐브 색상으로 인덱스를 변경
        if (colorIndex < cubeSpawner.CubeColors.Length - 1)
        {
            colorIndex++;
        }
        else
        {
            colorIndex=0;
        }

        // 실제 큐브 색상 변셩
        meshRenderer.material.color=cubeSpawner.CubeColors[colorIndex];
    }

    private void OnTriggerEnter(Collider other)
    {
        MeshRenderer renderer=other.GetComponent<MeshRenderer>();
        // 터치 가능한 큐브와 부딪힌 큐브의 색상이 같을 때
        if (meshRenderer.material.color == renderer.material.color)
        {
            cubeChecker.CorrectCount++;
        }
        else
        {
            cubeChecker.IncorrectCount++; ;
        }
    }
}
```

이제 게임을 실행해보면 색상이 전부 맞으면 성공 메세지가 나오고, 하나라도 틀리면 실패 메세지가 나온다.

# 현재 획득 점수 출력

(1) 현재 획득 점수 출력을 위해 TextmeshPro를 생성하고 이름을 TextScore로 바꾼다.

(2) 앵커로 포지션 잡아주고 설정들을 바꿔준다.

(3) 점수를 등록시키고 업데이트 된 점수를 화면에 출력하는 GameController스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class GameController : MonoBehaviour
{
    [SerializeField]
    private TextMeshProUGUI textScore;
    private int score = 0;

    public void IncreaseScore()
    {
        score++;
        textScore.text = "Score "+score.ToString();
    }
}
```

(4) GameController실행을 위해 빈 오브젝트를 만들고 이름을 GameController로 바꾸고 GameController스크립트를 넣어준다.

(5) 색상 매칭에 성공했을 때 점수를 증가시키기 위해 CubeChecker스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeChecker : MonoBehaviour
{
    [SerializeField]
    private CubeSpawner cubeSpawner;
    [SerializeField]
    private GameController gameController;

    private CubeController[] touchCubes;        //터치 가능한 큐브셋의 큐브들

    private int correctCount = 0;
    private int incorrectCount = 0;

    public int CorrectCount
    {
        set => correctCount = Mathf.Max(0, value);
        get=> correctCount;
    }
    public int IncorrectCount
    {
        set=>incorrectCount = Mathf.Max(0, value);
        get => incorrectCount;
    }
    private void Awake()
    {
        touchCubes=GetComponentsInChildren<CubeController>();

        for(int i = 0; i < touchCubes.Length; i++)
        {
            touchCubes[i].SetUp(cubeSpawner,this);
        }
    }

    private void Update()
    {
        // 맞은 갯수 + 틀린 개수가 터치 가능한 큐브의 개수와 같을 때
        if (correctCount + incorrectCount == touchCubes.Length)
        {
            // 하나도 틀리지 않았을 때 : 성공
            if (IncorrectCount == 0)
            {
               // Debug.Log("Success");
               gameController.IncreaseScore();
            }
            // 하나라도 틀렸을 때 : 실패
            else
            {
                Debug.Log("Failed");
            }
            CorrectCount = 0;
            IncorrectCount = 0;
        }
    }
}
```

이제 게임을 실행하고 색상 매칭에 성공했을 때 스코어가 갱신되는 모습을 볼 수 있다.

# 결과 화면 출력

결과 화면은 게임화면 위에 출력 되며 게임화면을 조금 어둡게 보이도록하고 GameOver,Score,High Score 텍스트를 출력하게 만든다.

(1) 게임 화면을 어둡게 만들기 위해 Panel을 만들고 이름을 PanelResult로 바꾸고 색상을 검은색으로 바꾸고 알파값을 200으로 한다.

(2) Game Over 텍스트 출력을 위해 TxtMeshPro를 패널리절트의 자식으로 만들고 이름을 TextGameOver로 바꾸고 설정을 해준다.

(3) 결과 점수 텍스트 출력을 위해 TxtMeshPro를 패널리절트의 자식으로 만들고 이름을 TextResultScore로 바꾸고 설정을 해준다.

(4) 최고 점수 텍스트 출력을 위해 TxtMeshPro를 패널리절트의 자식으로 만들고 이름을 TextHighScore로 바꾸고 설정을 해준다.

(5) PanelResult를 비활성화 해준다.

(6) 게임 오버시 PanelResult를 활성화하고 현재 점수와 최고 점수가 갱신 되도록 GameController스크립트를 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class GameController : MonoBehaviour
{
    [SerializeField]
    private TextMeshProUGUI textScore;
    private int score = 0;


    [SerializeField]
    private GameObject panelResult;
    [SerializeField]
    private TextMeshProUGUI textResultScore;
    [SerializeField]
    private TextMeshProUGUI textHighScore;

    public bool isGameOver { private set; get; }
    private void Awake()
    {
        isGameOver = false;
    }
    public void IncreaseScore()
    {
        score++;
        textScore.text = "Score "+score.ToString();
    }
    public void GameOver()
    {
        // 현재 상태를 게임오버로 설정 (CubeChecker클래스에서 점수 계산을 하지 않도록)
        isGameOver=true;
        // 스테이지의 점수 출력 Text UI 비활성화
        textScore.enabled = false;
        // 결과 화면 Panel UI 활성화
        panelResult.SetActive(true);

        // 기존의 등록된 최고 점수 불러오기
        int highScore = PlayerPrefs.GetInt("HighScore");

        // 기존 점수가 최고 점수보다 낮을 때
        if (score < highScore)
        {
            textHighScore.text ="High Score "+highScore.ToString();
        }
        else
        {
            // 현재 점수를 최고 점수로 갱신하기
            PlayerPrefs.SetInt("HighScore", score);

            textHighScore.text = "High Score " + highScore.ToString();
        }

        textResultScore.text="Score "+score.ToString();
    }
}
```

(7) CubeChecker스크립트도 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CubeChecker : MonoBehaviour
{
    [SerializeField]
    private CubeSpawner cubeSpawner;
    [SerializeField]
    private GameController gameController;

    private CubeController[] touchCubes;        //터치 가능한 큐브셋의 큐브들

    private int correctCount = 0;
    private int incorrectCount = 0;

    public int CorrectCount
    {
        set => correctCount = Mathf.Max(0, value);
        get=> correctCount;
    }
    public int IncorrectCount
    {
        set=>incorrectCount = Mathf.Max(0, value);
        get => incorrectCount;
    }
    private void Awake()
    {
        touchCubes=GetComponentsInChildren<CubeController>();

        for(int i = 0; i < touchCubes.Length; i++)
        {
            touchCubes[i].SetUp(cubeSpawner,this);
        }
    }

    private void Update()
    {
        if (gameController.isGameOver == true) return;      // 게임 오버시 점수 계산 하지 않도록

        // 맞은 갯수 + 틀린 개수가 터치 가능한 큐브의 개수와 같을 때
        if (correctCount + incorrectCount == touchCubes.Length)
        {
            // 하나도 틀리지 않았을 때 : 성공
            if (IncorrectCount == 0)
            {
               // Debug.Log("Success");
               gameController.IncreaseScore();
            }
            // 하나라도 틀렸을 때 : 실패
            else
            {
               // Debug.Log("Failed");
               gameController.GameOver();
            }
            CorrectCount = 0;
            IncorrectCount = 0;
        }
    }
}
```

이제 게임을 시작하고 색상 매칭에 실패했을 때 게임오버 화면이 출력 된다.

# 재시작 버튼

(1) 재시작 버튼을 출력하기 위해 PanelResult오브젝트의 자식으로 Button-TextMeshPro UI를 하나 만들고 이름을 ButtonRestart로 바꿔준 다음 설정을 알맞게 해준다.

(2) ButtonRestart의 자식으로 있는 텍스트를 Restart로 바꿔준다.

(3) 씬 로드를 제어하는 SceneLoder스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneLoader : MonoBehaviour
{
    public void LoadScene(string sceneName=" ") 
    {
        if(sceneName==" ")
        {
            SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        }
        else
        {
            SceneManager.LoadScene(sceneName);
        }
    }
}
```

(4) GameController오브젝트에 SceneLoader를 추가해준다.

(5) 버튼 UI의 OnClick()에 GameController를 할당하고 함수를 SceneLoader로 설정해준다.

이제 게임을 시작하고 게임오버가 됐을 때 재시작 버튼을 누르면 현재 씬이 다시 로드 되는 것을 볼 수 있다.