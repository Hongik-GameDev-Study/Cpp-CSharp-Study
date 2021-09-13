# Map & Unordered Map in C++11

<br>

기본적으로 map과 unordered_map은 key, value의 한 쌍으로, pair<Tkey, Tvalue> 형태의 요소를 갖는다.

<br>

> ## Map

---

map은 여타 STL 컨테이너와 비슷하게 RB Tree로 구현되어 있습니다.

* RB Tree는 AVL Tree처럼 완벽한 이진 트리를 형성하진 않지만,
Self balancing 기능으로 어느 정도 O(log n)의 탐색 시간을 보장하는 자료구조입니다.
그리고 자료는 탐색에 용이하도록 Key 값을 기준으로 정렬됩니다. 

* Key 값의 분포가 고르지 못한 경우, balancing에 대한 cost때문에 insert, delete의 성능이 떨어집니다.
하지만 그 속도가 O(log n)인 것은 보장됩니다.
