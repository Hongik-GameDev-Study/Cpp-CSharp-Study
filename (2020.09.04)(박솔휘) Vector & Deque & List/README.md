# Vector & Deque & List in C++11

<br>

vector는 array 기반, list는 Linked List 기반으로 value를 저장하는 자료구조입니다.


<br>

> ## Vector

* vector는 array이므로, 연속적인 메모리 공간에 할당합니다.
그렇기에 iterator 뿐 아니라 index로도 접근이 가능합니다.

* 연속적인 메모리 공간에 저장되기 때문에, list에 비해서 임의 접근 속도가 O(1)으로 빠릅니다.

* 또한, 동적으로 확장/축소가 가능한 Dynamic Array로 구현되어 있으며,
컨테이너 끝에서 insert / delete 하는 속도가 O(1)로 아주 빠릅니다.

* 하지만 그 외의 곳에 insert / delete 하는 경우, 모든 요소를 한 칸 뒤로 밀고 요소를 삽입하기 때문에,
O(n)으로 느린 시간 복잡도를 가집니다.



<br>

> ## List

* list는 Linked List이므로, 요소가 다음 요소를 가리키는 포인터를 가집니다. (요소 데이터 크기도 커짐)

* 또한 연속적인 메모리 공간에 할당된 것이 아니므로, 임의 접근은 O(n) 속도를 가지며, 인덱스 접근이 불가합니다.

* 하지만 요소를 insert / delete 하는 것은 포인터 간 관계를 끊고, 그 사이에 연결만 하면 되므로 O(1)의 
짧은 시간 복잡도를 가집니다.

---


<br>

> ## Vector와 List의 단편적인 사용처

* 이러한 특징 때문에 

1. search가 잦은 경우 vector 
2. insert / delete가 잦은 경우 list

* 를 사용하는데, vector 사용에는 추가적인 고려 요인이 있습니다.

---


<br>

> ## 고려해야 할 Vector의 내부

* vector는 array 기반이기 때문에, Dynamic하게 사용하기 위해선 내부 구현에 한계가 있습니다.

* 바로 capacity와 size에 대한 문제입니다. 

* vector는 size가 커져서 더이상 값을 저장할 수 없게 되기 전에, 저장 가능한 크기인 capacity를 대폭 증가시켜 이를 방지합니다.
(capacity를 증가시키는 과정은 오버헤드가 크기 때문에)

* 하지만 이 행위는 size에 비해 capacity를 방대하게 키워 낭비하는 공간을 유도할 수 있습니다.

ㅡ> 예방하기 위해선 vector::reserve(int capacity - 기존 capacity보다 큰 값)를 통해 벡터의 capacity를 지정해줘야 합니다. 
* 조사 과정에서 resize(int size)와 헷갈렸는데, 이는 this->capacity = size, this->size = size 라고 함

* list는 지정된 capacity가 없기 때문에, 요소가 삭제되면 메모리를 해제합니다.
하지만 vector는 요소가 삭제되어도 capacity를 유지하기 때문에 메모리를 해제하지 않습니다. ㅡ> 오버헤드 발생 가능성

---

<br>

> ## Vector와 List의 중간 즈음, Deque

* deque는 인덱스를 통한 접근이 가능하며, 
vector와 달리 back() 뿐만 아니라 front()와 back()에 insert, delete 하는 것에 대한 시간 복잡도가 O(1)입니다.

* 하지만 결국 중간에 요소를 추가하게 되면 O(n)의 시간 복잡도를 가지기 때문에 
vector와 list의 중간 정도라고 보면 편할 것 같습니다.

* 메모리에 연속으로 할당되는 자료구조도 아니기 때문에, 포인터 연산 접근이 불가능합니다. 

![Untitled](http://1.bp.blogspot.com/-jn-REeJyDZA/UnWQm_zO0SI/AAAAAAAAAeY/Y521Nmy2vMc/s320/deque+sketch.png)
