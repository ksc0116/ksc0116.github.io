---
layout: post
title:  "[STL] multiset컨테이너"
subtitle:   ""
categories: STL
comments: true
---

<br>

multiset 컨테이너는 중복 원소가 저장 가능하다는 사실 말고는 set 컨테이너와 아예 똑같다. 근데 중복 원소를 허용하기 때문에 insert()는 삽입 위치와 삽입 성공여부의 값을 저장한 pair객체를 반환하는 것이 아닌 삽입된 위치만을 반복자로 반환한다. 예시를 보자

```cpp
#include <iostream>
#include <set>
using namespace std;

void main() {
	multiset<int> ms;
	multiset<int>::iterator iter;
	ms.insert(50);
	ms.insert(30);
	ms.insert(80);
	ms.insert(80);//80중복 됨
	ms.insert(30);//30중복 됨
	ms.insert(70);
	iter = ms.insert(10);
	//ms의 모습 10 30 30 50 70 80 80 
	
	cout<<"iter이 가리키는 원소 : " << *iter << endl;

	for (iter = ms.begin(); iter != ms.end(); iter++) {
		cout << *iter << " ";
	}
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/158308356-82998fab-49ea-4950-9de7-d60f7077ced5.png)

위 코드와 같이 multiset에서 insert()는 반환하는 값이 반복자이다.

multiset도 찾기 관련 함수(**count(), find(), lower_bound(),upper_bound(),equal_range()**)들을 제공한다. 차근차근 살펴보자.

# mutilset의 찾기 관련 함수들

각 함수가 뜻하는 바는 set 컨테이너 설명할 때 했기에 건너뛰고 예시 코드를 봐보자.

```cpp
#include <iostream>
#include <set>
using namespace std;

void main() {
	multiset<int> ms;
	ms.insert(50);
	ms.insert(30);
	ms.insert(80);
	ms.insert(80);
	ms.insert(30);
	ms.insert(70);
	ms.insert(10);
	//ms의 모습 10 30 30 50 70 80 80 

	multiset<int>::iterator iter;
	for (iter = ms.begin(); iter != ms.end(); iter++) {
		cout << *iter << " ";
	}
	cout << endl;

	//count()
	cout << "30의 갯수 : " << ms.count(30)<<endl; //결과 2

	//find()
	iter = ms.find(30); //30의 첫번째 반복자를 반환
	cout<<"iter : " << *iter << endl;

	//lower_bound() //upper_bound()
	multiset<int>::iterator iter_lower;
	multiset<int>::iterator iter_upper;

	iter_lower = ms.lower_bound(30);//30 순차열의 시작 반복자 반환
	iter_upper = ms.upper_bound(30);//30 순차열의 끝 반복자 반환
	cout << "iter_lower : " << *iter_lower << endl;
	cout << "iter_upper : " << *iter_upper << endl;

	cout << "구간 [iter_lower, iter_upper)의 순차열 : ";
	for (iter = iter_lower; iter != iter_upper; iter++) {
		cout << *iter << " ";
	}
}
```

실행 결과

![image-20220315135744353](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220315135744353.png)

여기서 중요한건 find()는 해당 원소의 시작 반복자를 반환한다는 것, 그리고 lower_bound()는 해당 원소의 순차열의 시작 반복자를 upper_bound()는 해당 원소의 순차열의 끝 반복자를 반환한다는 것이다.

# equal_range()

마지막으로 equal_range()에 대해서 설명하고 포스팅을 마치겠다. 사실 설명할 것도 없는게 set하고 똑같다. equal_range()가 반환하는건 반복자 쌍으로 이루어진 pair객체라는 것이다. 예시를 보자.

```cpp
#include <iostream>
#include <set>
using namespace std;

void main() {
	multiset<int> ms;
	ms.insert(50);
	ms.insert(30);
	ms.insert(80);
	ms.insert(80);
	ms.insert(30);
	ms.insert(70);
	ms.insert(10);
	//ms의 모습 10 30 30 50 70 80 80 

	multiset<int>::iterator iter;
	for (iter = ms.begin(); iter != ms.end(); iter++) {
		cout << *iter << " ";
	}
	cout << endl;

	//반복자 쌍을 받기 위한 pair객체 생성
	pair<multiset<int>::iterator, multiset<int>::iterator> pr;
	pr = ms.equal_range(30);

	//구간 [pr.first,pr.second)의 순차열
	for (iter = pr.first; iter != pr.second; iter++) {
		cout << *iter << " ";
	}
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/158310133-f5d70c91-9a82-4908-a78e-c92c856dd529.png)

결과로 보이는 것 처럼 pr.first에는 lower_bound()의 반복자가 pr.second에는 upper_bound()의 반복자가 들어간다. 그림으로 설명하고 마치겠다.

![image](https://user-images.githubusercontent.com/101051124/158310392-a95914d2-7fd5-45a4-8f4e-f62ef7afcdd5.png)

위 코드를 그림으로 나타내면 위와 같은 상황인 것이다.