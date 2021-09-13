# Map & Unordered Map in C++11

<br>

기본적으로 map과 unordered_map은 key, value의 한 쌍으로, pair<Tkey, Tvalue> 형태의 요소를 갖는다.

<br>

> ## Map

* map은 여타 STL 컨테이너와 비슷하게 RB Tree로 구현되어 있습니다.

* RB Tree는 AVL Tree처럼 완벽한 이진 트리를 형성하진 않지만,
Self balancing 기능으로 어느 정도 O(log n)의 탐색 시간을 보장하는 자료구조입니다.
그리고 자료는 탐색에 용이하도록 Key 값을 기준으로 정렬됩니다. 

* Key 값의 분포가 고르지 못한 경우, balancing에 대한 cost때문에 insert, delete의 성능이 떨어집니다.
하지만 그 속도가 O(log n)인 것은 보장됩니다.


<br>

> ## Unordered Map

* unordered_map은 hash table로 구현되어 있습니다.

* hash table은 말 그대로 hash 값을 hash function으로 주소로 변환하여 접근합니다.
map처럼 정렬할 필요가 없기 때문에 insert, delete, search가 모두 O(1)로 일정합니다.

* 하지만 hash function 만큼의 cost가 항상 필요하기 때문에 고정적인 cost가 존재합니다.
그래서 데이터의 양에 따라 아래와 같이 map과 성능이 비교될 수 있습니다.

﻿

![Untitled](https://postfiles.pstatic.net/MjAyMTA5MDRfMjA4/MDAxNjMwNjg3MjEyOTU4.n9RL2F7CCaydmdYMR2SR_DlyJqUBN2SsvVjJb2jyAQgg.WJt-JwzdwMML8LYlBoDapscncHYbFsdYW6chi_5sM_sg.PNG.psh50zmfhtm/image.png?type=w966)

﻿

> Key 값이 int일 때, 위와 같이 특정 데이터의 양 이상으로 많아지면 unordered_map이 압도적으로 성능이 좋아집니다.

<br>

> 하지만 Key 값이 string일 땐 주의할 점이 있습니다.

map의 Key 비교 함수는 앞에서부터 차례대로 비교하여 크고 작음을 가려내기 때문에, Key 문자열 길이의 변화는 그 영향이 적을 확률이 높습니다.
(하지만 타임 스탬프 기록 등 앞 부분 유사도가 높은 경우엔 주의해야 합니다.)

하지만 unordered_map은 문자열을 모두 Hashing하기 때문에, 그 길이에 영향을 비교적 많이 받습니다.

문자열의 길이가 각각 4, 8, 12, 16인 경우의 비교 그래프입니다.

![Untitled](https://postfiles.pstatic.net/MjAyMTA5MDRfMjUy/MDAxNjMwNjg5NTc3MzQ1.PBfUqI-56Z9fzC-5ns29wfbgnOD2J47ubzsdHfWHDpsg.6TZStB0YbpKIKV_Y8xchvNTBYp-FYkLloqVZfyJrJZAg.PNG.psh50zmfhtm/image.png?type=w966)

> ## 결론

1. 데이터가 많은 경우에는 unordered_map 이 map 보다 성능 면에서 유리합니다. 
2. 문자열을 Key로 사용하는 경우 문자열이 길어질 수록 unordered_map 이 map에 비해 더 성능이 떨어질 수 있습니다. 
3. 유사도가 높은 문자열 집합을 키로 사용하는 경우에 map 의 성능이 떨어질 수 있습니다. 
 
결론적으로 key를 이용하여 정렬을 해야 하는 경우를 제외하고, 대량의 데이터를 저장할 때는 unordered_map 을 사용하는 걸 추천합니다.
