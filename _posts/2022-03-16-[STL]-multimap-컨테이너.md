---
layout: post
title:  "[STL] multimap 컨테이너"
subtitle:   ""
categories: STL
comments: true
---

<br>

multimap 컨테이너는 중복 key를 허용하는것 말고는 map과 다른점이 없다.

예시로 multimap에 중복 key를 저장하고 찾기 관련 함수를 사용해보겠다.

# count() / find()

우선 원소의 갯수를 알려주는 count()와 인자로 넘긴 값에 해당하는 key를 가진 원소가 있다면 원소중에서 첫 번째 원소의 반복자를 반환하고 없다면 end()를 반환하는 find()를 사용해 보겠다.

```cpp
#include <iostream>
#include <map>
#include<string>
using namespace std;

void main() {
	multimap<int, int> mm;

	mm.insert(pair<int, int>(5, 100));
	mm.insert(pair<int, int>(3, 100));
	mm.insert(pair<int, int>(8, 30));
	mm.insert(pair<int, int>(3, 40));
	mm.insert(pair<int, int>(1, 70));
	mm.insert(pair<int, int>(7, 100));
	mm.insert(pair<int, int>(8, 50));

	multimap<int, int>::iterator iter;

	for (iter = mm.begin(); iter != mm.end(); iter++) {
		cout << "(" << iter->first << ", " << iter->second << ")" << " ";
	}
	cout << endl;

	cout << "key 3의 원소의 갯수는 : " << mm.count(3) << endl;

	iter = mm.find(3);
	if (iter != mm.end())
		cout << "첫 번째 key 3에 매핑된 value: " << iter->second << endl;
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/158505418-91538d73-4735-478c-bd3d-e56b4082734a.png)

어려운 내용은 없다.

# lower_bound() / upper_bound() / equal_range()

인자로 넘긴 값에 해당하는 key를 가진 원소가 있다면 그 원소의 시작 반복자를 반환하는 lower_bound(), 인자로 넘긴 값에 해당하는 key를 가진 원소가 있다면 그 원소의 끝 반복자를 반환하는 upper_bound(), lower_bound()의 반환값과 upper_bound()의 반환값을 반복자쌍으로 반환하는 equal_range()를 사용해보겠다. 여기서 다시한번 말하지만 lower_bound()의 값과 upper_bound()의 값이 같다면 찾고자하는 해당 원소는 없는 것이다. 예시를 보자.

```cpp
#include <iostream>
#include <map>
#include<string>
using namespace std;

void main() {
	multimap<int, int> mm;

	mm.insert(pair<int, int>(5, 100));
	mm.insert(pair<int, int>(3, 100));
	mm.insert(pair<int, int>(8, 30));
	mm.insert(pair<int, int>(3, 40));
	mm.insert(pair<int, int>(1, 70));
	mm.insert(pair<int, int>(7, 100));
	mm.insert(pair<int, int>(8, 50));

	multimap<int, int>::iterator iter;

	for (iter = mm.begin(); iter != mm.end(); iter++) {
		cout << "(" << iter->first << ", " << iter->second << ")" << " ";
	}
	cout << endl;

	multimap<int, int>::iterator lower_iter;
	multimap<int, int>::iterator upper_iter;
	lower_iter = mm.lower_bound(3);
	upper_iter = mm.upper_bound(3);
	cout << "구간 [ lower_iter, upper_iter )의 순차열 : ";
	for (iter = lower_iter; iter != upper_iter; iter++) {
		cout << "(" << iter->first << ", " << iter->second << ")" << " ";
	}
	cout << endl;

	pair< multimap<int, int>::iterator, multimap<int, int>::iterator> iter_pair;
	iter_pair = mm.equal_range(3);
	cout << "구간 [ iter_pair.first , iter_pair.second )의 순차열 : ";
	for (iter = iter_pair.first; iter != iter_pair.second; iter++) {
		cout << "(" << iter->first << ", " << iter->second << ")" << " ";
	}
}
```

실행 결과

![image](https://user-images.githubusercontent.com/101051124/158506701-74ca2873-89ad-4b81-953d-9a02907cace4.png)

어려운 내용은 아니다. 이번 글에서 알고 넘어가야하는건 multimap은 중복 key를 허용한다는 것이다.