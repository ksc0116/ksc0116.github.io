---
layout: post
title:  "[STL] vector 컨테이너"
subtitle:   ""
categories: STL
comments: true
---

<br>

vector 컨테이너에 대해 정리해보자. vector 컨테이너를 사용하려면 vector를 include 해야한다.

# push_back

vector 컨테이너에 원소를 추가 해주는 것이다. 

ex)

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);
}
```

그럼 v 컨테이너의 모습은 1 2 3 4가 된다.

# size()

모든 컨테이너는 size() 멤버함수를 가진다. 이는 그 컨테이너의 원소의 **갯수**를 반환한다. 예를 들어 위의 코드에서 v.size()는 4이다.

# vector<int>::size_type

vector의 size_type은 원소의 개수나 [] 연산자등의 인덱스로 사용되는 멤버이다. 예를들어 for문으로 컨테이너의 원소를 출력하고 싶다면 

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);

	for (vector<int>::size_type i = 0; i < v.size(); i++) {
		cout << v[i] << " ";
	}
}
```

위와 같이 사용하면 된다.

# capacity()

vector 컨테이너에만 존재하는 멤버함수로 컨테이너에 실제 할당된 메모리 크기를 의미한다. vector 컨테이너는 하나의 메모리 공간에 원소를 넣어서 관리하기 때문에 원소가 계속 추가되다가 용량이 부족하면 새로운 메모리 공간을 할당하고 원소를 복사해서 다시 붙여놓고 새로운 원소를 추가하는 식으로 동작하는데 이러한 상황이 계속 발생하면 비효율적이기 때문에 미리 메모리 공간을 확보해두는 방법으로 동작하게 된다. 예시를 들어보면 

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(10);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(20);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(30);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(40);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(50);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(60);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(70);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(80);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	v.push_back(90);
	cout << "size : " << v.size() << " capacity : " << v.capacity() << endl;
	

	for (vector<int>::size_type i = 0; i < v.size(); i++) {
		cout << v[i] << " ";
	}
	cout << endl;
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/157780740-5887a0e2-81a5-4aed-a821-2c2835889d05.png)

즉, 원소를 추가하는데 capacity가 size보다 크지 않다면 메모리 재할당은 하는데 1을 증가시키는게 아니라 여유분까지 만든다. 위의 결과를 보면 이해하기 쉬울 것이다.

# max_size()

컨테이너가 최대한으로 가질 수 있는 원소의 갯수를 나타낸다.

# reserve()

메모리를 재할당하는 과정은 비효율적이기 때문에 vector는 미리 메모리를 할당할 수 있는 메모리 예약 함수 reserve()를 제공한다. 예시를 보자

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;

	v.reserve(5);
}
```

다음과 같은 코드가 있으면 v는 size가 0이고 capacity가 0이다. 하지만 미리 예약해둔 메모리 크기보다 원소가 계속 추가 되다가 capacity보다 size가 커지게 되면 메모리를 재할당한다.

# vector의 생성자

vector의 생성자로 size와 초기값을 정할 수 있다.

예시 코드를 보자.

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v1(5); //size가 5이며 초기값이 0
	//v1의 모습 : 0 0 0 0 0

	vector<int> v2(5, 10); //size가 5이며 초기값이 10
	//v2의 모습 : 10 10 10 10 10

	vector<int> v3(v1.begin(), v1.end()); //순차열로 초기화가 가능하다
	//v3의 모습 : 0 0 0 0 0
}
```

또한 resize()함수로 컨테이너의 size를 변경할 수 있다.

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v1(5,2); //size가 5이며 초기값이 2
	//v1의 모습 : 2 2 2 2 2
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	v1.resize(10); //size가 10으로 변하며 새로 추가 되는 부분은 0으로 초기화
	//v1의 모습 : 2 2 2 2 2 0 0 0 0 0
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	v1.resize(15,100); //size가 15으로 변하며 새로 추가 되는 부분은 1000으로 초기화
	//v1의 모습 : 2 2 2 2 2 0 0 0 0 0 100 100 100 100 100
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	v1.resize(5); //다시 size를 5로 되돌림
	//v1의 모습 : 2 2 2 2 2
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/157782559-52156266-765a-405a-8325-3c60e4172763.png)

위의 결과를 보면 한 번 늘어난 capacity는 size가 줄어도 줄지 않는 모습을 볼 수 있다.

# clear() / empty()

clear()함수는 컨테이너의 모든 원소를 지운다. 즉 size가 0으로 되고 empty()함수는 size가 0이면 true를 반환한다.

예시를 보자.

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v1(5,2); //size가 5이며 초기값이 2
	//v1의 모습 : 2 2 2 2 2
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;
	
	v1.clear();
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	if (v1.empty()) {
		cout << "v1은 비어있다.";
	}
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/157782889-9caaa2a3-09b8-45c1-8e9c-1770ab187c07.png)

결과를 보면 size도 0이 되고 비어있다고 출력이 되지만 capacity가 남아있는걸 확인 가능하다. 즉, 아직 메모리 공간을 차지하고 있다는 것이다. 그로인해 clear()는 원소만 지운다는걸 확인했다. 그럼 capacity를 0으로 만드는 방법이 없을까? 아니다 존재한다. 방법을 알아보자

# swap()

모든 컨테이너는 컨테이너 객체끼리 교환할 수 있는 swap()함수를 제공한다. 그럼 size와 capaxity가 0인 컨테이너와 위의 컨테이너를 교환하면 위의 컨테이너는 size와 capacity가 0인 우리가 원하는 모양으로 만드는게 가능하다. 예시를 보자.

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v1(5,2); //size가 5이며 초기값이 2
	//v1의 모습 : 2 2 2 2 2
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	vector<int>().swap(v1);
	cout << "size : " << v1.size() << " capacity : " << v1.capacity() << endl;

	if (v1.empty()) {
		cout << "v1은 비어있다.";
	}
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/157783527-c76656b1-f75d-4b58-9050-45070900a033.png)

위처럼 size와 capacity가 0인 임시객체를 만들어서 기존에 있던 size와 capacity를 0으로 만들고 싶은 컨테이너 객체를 swap해주면 된다. 임시객체로 하는 이유는 swap하고 바로 메모리에서 지울 것이기 때문이다.

# front() / back()

시퀀스 컨테이너는 삽입 위치(순서)개념이 있으니 모든 시쿠너스 컨테이너 vector,deque,list는 첫번째 원소와 마지막 원소를 참조하는 front()와 back()을 제공한다.  front()와 back()으로 순차열의 처음 값과 마지막 값을 출력할 수 있고,  front()와 back()으로 처음 과 끝의 원소를 다른 원소로 수정하는게 가능하다.

# at()

vector는 배열 기반 컨테이너기 때문에 []연산자를 통해 임의의 위치의 원소를 참조하는데 가능하다. 그리고 at()멤버함수로 []연산자와 똑같이 사용할 수 있다. 예를들어 v[1]이랑 v.at(1)은 같은 의미이다. 그럼 왜 존재하는 걸까? 분명한 차이점이 있기 때문이다. []연산자는 배열의 범위를 점검하지 않아서 빠르고 at()은 범위점검을 해서 예외를 발생하기 때문에  조금 느리다 하지만 안전하다는 장점이 있다.

# assign()

뜻 그대로 할당하다라는 뜻이다. 컨테이너 생성 후에 일괄적으로 값을 컨테이너에 할당하는게 가능하다. 모든 시퀀스 컨테이너는 assign() 멤버 함수를 제공한다. 예시를 보자

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v1;
	v1.assign(3,10); //3개의 원소에 10을 할당
	for (vector<int>::size_type i=0; i < v1.size(); i++) {
		cout << v1[i] << " ";
	}
	cout<<endl;

	vector<int> v2;
    //반복자를 이용하여 값 할당 가능
	v2.assign(v1.begin(), v1.end());//v2의 원소에 v1순차열을 할당
	
	for (vector<int>::size_type i = 0; i < v2.size(); i++) {
		cout << v2[i] << " ";
	}
}
```

이러면 v1의 모습은 10 10 10이고 v2의 모습도 10 10 10이 된다.

# const_iterator

반복자를 통해서 원소의 값을 변경하지 않을 것이라면  상수 반복자를 사용하는 것이 좋다.const_iterator는 const int*와 비슷하다. 예시를 보자

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);
	v.push_back(5);
	
	vector<int>::const_iterator citer = v.begin();
	vector<int>::iterator iter = v.begin();

	//가리키는 곳 이동은 가능
	citer += 2;
	iter += 2;
	
    //값 읽기는 가능
    cout<<*citer<<endl;
    cout<<*iter<<endl;
    
	//원소의 값 변경 불가능
	//*citer = 100;

	*iter = 100;
}
```

다음과 같이 const_iterator는 가리키고 있는 위치를 변경하는 것은 가능하지만 가리키고 있는 곳의 값 변경은 불가능 하다. 여기서 잠깐 헷갈릴만한 개념이 나온다. const vector<int>::iterator와 

const vector<int>::const_iterator다.  const vector<int>::iterator는 (int* const)와 비슷하고 

const vector<int>::const_iterator는 const int* const와 비슷하다. 즉 모든걸 정리하면 아래와 같다.

vector<int>::iterator (int) : 값 변경 가능 , 다음 원소 이동 가능

vector<int>::const_iterator (const int*) : 값 변경 불가능, 다음 원소 이동 가능

const vector<int>::iterator (int* const) : 값 변경 가능, 다음 원소로 이동 불가능

const vector<int>::const_iterator (const int* const) : 값 변경 불가능 , 다음 원소로 이동 불가능

# insert() / erase()

insert() 멤버 함수는 반복자가 가리키는 위치에 원소를 추가할 수 있고

erase() 멤버 함수는 반복자가 가리키는 곳의 원소를 제거한다.

먼저 insert()의 예시를 보자

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);
	v.push_back(5);
	//v의 모습 1 2 3 4 5

	vector<int>::iterator iter;
	iter = v.begin() + 1;

	vector<int>::iterator iter2;
	//iter2는 원소가 삽입 된 위치를 가리킴
	iter2=v.insert(iter, 3);
	//v의 모습 1 3 2 3 4 5

	v.insert(iter2, 2, 100); //iter2가 가리키는 곳에 100을 2개 넣어라
	//v의 모습 1 100 100 3 2 3 4 5

	vector<int> v2;
	v2.push_back(10);
	v2.push_back(20);
	v2.push_back(30);
	
	v.insert(iter2, v2.begin(), v2.end()); //iter2가 가리키는 위치에 v2의 순차열을 넣어라
	//v의 모습 1 10 20 30 100 100 3 2 3 45
}
```

간단한게 이해가 될 것이다.

이제 erase()의 에시를 보자

```cpp
#include <iostream>
#include<vector>
using namespace std;

void main() {
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);
	v.push_back(5);
	//v의 모습 1 2 3 4 5

	vector<int>::iterator iter;
	iter = v.begin() + 1;

	vector<int>::iterator iter2;
	//iter2는 iter을 지우고 iter 다음의 원소를 가리킴
	iter2 = v.erase(iter);
	//v의 모습 1 3 4 5
	//iter2가 가리키는 원소 3

	iter2 = v.erase(v.begin() + 1, v.end());
	//v의 모습 1
	//iter2가 가리키는 곳 v.end()
}
```

여기서 헷가리기 쉬운건 erase()는 제거된 원소의 다음 원소를 가리키는 반복자를 반환한다는 것이다.

