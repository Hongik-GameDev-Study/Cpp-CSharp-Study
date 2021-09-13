# Map & Unordered Map in C++11

<br>

* queue는 선입선출 기반의 선형 자료구조이며 내부는 deque로 되어있습니다.
priority_queue는 이름은 queue이나 우선 순위가 높은 데이터가 먼저 나가는 heap 기반의 tree 자료구조입니다.

* queue는 front()와 back()을 통해서만 접근할 수 있으며,
priority_queue는 top()을 통해 맨 앞에만 접근할 수 있습니다.


<br>

> ## Queue

* queue는 선입 선출 기반 deque이므로 가장 먼저 들어온 요소가 front()에 존재하며, 가장 늦게 들어온 요소가 back()에 존재합니다.

* 접근 자체를 front(), back()으로만 가능한 deque이기 때문에 어떠한 상황에도 insert, delete 모두 O(1)의 시간이 소요되며,
인덱스 접근, 포인터 연산이 불가능하기 때문에 search는 불가능합니다.


<br>

> ## Priority Queue

* priority_queue는 default로 최대 heap으로 되어 있는데,
이는 가장 큰 요소가 top()으로 옴을 의미합니다.

* heap은 내부적으로 완전 이진 tree이기 때문에 insert, delete 모두 O(logn)의 시간이 소요되며,
priority를 생성할 때 넣어주는 compare 함수를 통해 정렬 방식을 바꾸어줄 수 있습니다.
