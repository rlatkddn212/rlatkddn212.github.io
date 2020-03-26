---
layout: post
comments: true
title:  "[자료구조] Kd-Tree"
date:   2020-03-25T00:25:52-05:00
author: 김상우
categories: 자료구조
tags:	자료구조
---







### Kd-Tree



 일전에 게임 속 캐릭터 위치에서 가장 가까운 npc의 위치를 어떻게 구할지에 대한 질문을 받은 적이 있습니다. 저는 간단하게 모든 npc 위치를 점과 점사이 거리를 이용하여 비교하는 방법을 이야기 했습니다. 하지만 npc가 엄청나게 많다면 어떻게 처리하면 되느냐라는 추가 질문을 받게 되었습니다. 

 단순히 비교하는 방법은 선형시간 O(N)인데 이보다 빠른 O(logN) 시간 알고리즘을 알려달라는 이야기였죠. log 시간으로 만드는 알고리즘을 고민해봤고, 정렬된 상태에서 이분 탐색, 트리 같은 것들이 생각났습니다. 하지만 해법까지 도달하진 못했습니다.

 그에 대한 해법 중하나는 공간을 분할한 트리입니다. 여기서 배울 Kd-Tree(k-dimensional tree)는 K 차원에 정점들을 공간 분할하는 자료구조입니다. 위에서 언급한 가장 가까운 이웃을 찾는 문제(Nearest neighbour search)를 풀수 있습니다. 

그럼 Kd-Tree의 구현 방법을 이야기해보고 관련된 백준 온라인 저지에 문제를 하나 풀어 보겠습니다.



### 구현

 자료구조에서 Tree는 어려운 축에 속하는 문제일 겁니다. Array, Linked List, Stack, Queue를 넘어 지쳐갈 때 쯤 다음으로 Tree라는 자료구조를 접하게 되는데요. 특히 Tree의 삽입, 삭제를 구현할 때 재귀(recursion)라는 개념을 사용해야하기에 여기서 많은 사람들이 고통받으며 배우게됩니다.



 Kd-Tree는 대학교 교과 과정에선 잘 배우지 않지만 자료구조에서 Tree를 제대로 배우셨다면 문제 없이 구현하실 수 있습니다. 자료구조에서 배운 트리에서 크게 벗어나지 않습니다. 이진 탐색 트리와 비슷한 구조로 공간을 분할 합니다.



![image](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/Kdtree_2d.svg/370px-Kdtree_2d.svg.png)

트리에 노드는 무엇으로 할까요? 위 그림을 보면 2차원 좌표에 6개의 점이 있고 여러 공간으로 분할된 것을 확인할 수 있습니다.  트리의 노드는 분할된 직선입니다.

트리는 아래와 같이 구성됩니다.

1. 임의의 정점(주로 x의 중심값)이 root 노드가 됩니다.
2. x좌표 값이 root노드보다 작은 점은 왼쪽 자식노드에 속합니다.
3. 마찬가지로 x좌표값이 root 노드 보다 큰 점을 오른쪽 자식에 속합니다.
4. 그럼 점에 x값 기준으로 공간이 분할 됩니다.
5. 자식 노드는 같은 방법으로 y 값 기준으로 공간을 분할합니다.
6. 또 그 자식노드는 x 값 기준으로 공간을 분할합니다. 이렇게 x값으로 분할 -> y값으로 분할 -> x 값으로 분할 ... 더이상 점이 없을 때 까지 분할합니다.



[![img](https://upload.wikimedia.org/wikipedia/commons/thumb/2/25/Tree_0001.svg/370px-Tree_0001.svg.png)](https://en.wikipedia.org/wiki/File:Tree_0001.svg)

예를 들면 위 그림과 같이 되죠. 이젠 이진 탐색 트리랑 똑같아 보이지 않나요? 특이하게 트리의 레벨에 따라 x축 y축이 번가라가면서 처리가 됩니다. 



#### kd-tree 생성

kd 트리의 생성 코드는 위키에서 파이썬으로 구현된 코드를 쉽게 찾아 볼 수 있습니다.

``` python
def kdtree(point_list, depth: int = 0):
   if not point_list:
       return None

   k = len(point_list[0]) # assumes all points have the same dimension
   # Select axis based on depth so that axis cycles through all valid values
   axis = depth % k

   # Sort point list by axis and choose median as pivot element
   point_list.sort(key=itemgetter(axis))
   median = len(point_list) // 2

   # Create node and construct subtrees
   return Node(
       location=point_list[median],
       left_child=kdtree(point_list[:median], depth + 1),
       right_child=kdtree(point_list[median + 1:], depth + 1)
   )

```



point_list는 kd-tree를 구성할 정점에 집합입니다. 마지막 리턴 값을 주목해 보면 left_child, right_child를 재귀적으로 호출하면서 구성하는 것을 확인해 볼 수 있습니다. 이는 트리를 구성할 때 보여지는 일반 적인 패턴중 하나죠. 이진 트리나, 세그먼트 트리, 트라이 등등 트리 형태의 자료구조는 이와 같이 재귀적으로 생성됩니다.

``` python
	axis = depth % k

   point_list.sort(key=itemgetter(axis))
   median = len(point_list) // 2
```

공간 분할이 트리의 깊이값에 따라 x값, y값을 기준으로 분할되므로 현재 재귀 호출에서의 axis값을 구해 구분해줍니다.  그리고 정점들을 축을 기준으로 정렬한 뒤 중심값을 기준으로 공간을 분할 한 뒤 자식 트리를 생성하고 있죠.

정점을 모두 공간을 분할 하는데 사용했다면 Tree 생성이 완료됩니다.



#### 가장 가까운점 탐색

 특정 점에서 가장 가까운 이웃을 찾는 문제는 공간이 분할되어 있다는 장점을 이용하면 빠르게 탐색이 가능합니다.  그 점이 어떤 분할된 공간안에 있는지 안다면 그 주변만 찾아보면 되겠죠. 



이해를 돕기 위해 예를 하나 들어 보겠습니다.

![image-20200326230456770](C:\Users\arctu\AppData\Roaming\Typora\typora-user-images\image-20200326230456770.png)

빨간색 점과 가장 가까운 점을 찾는다고 한다면   
우선 빨간색이 어떤 공간에 위치해 있는지 찾습니다. Kd-Tree를 재귀적으로 탐색하게 되겠죠.  
그리고 그 공간에 부모노드와 점과 점사이 거리를 구한 후 최소값으로 기록하게됩니다.  

![image-20200326230809310](C:\Users\arctu\AppData\Roaming\Typora\typora-user-images\image-20200326230809310.png)

그림에서 초록색 값이 최소 길이겠죠. 최소 길이를 구한 후 재귀를 빠져 나오면서 부모의 분할 선을 확인하게 됩니다. 여기서 파란색 선이 초록색 선 보다 길기 때문에 반대편 공간을 확인할 필요 없다는 걸 확인하실 수 있을 겁니다. 원 범위보다 벗어난 공간은 확인 할 필요 없습니다.



### 성능



성능은 트리를 구성이 어떻게 이루어지냐에 따라 달라집니다. 딱 정확히 절반씩 나눠진다면 O(logN)의 탐색 시간를 가지고, 균형이 틀어진 트리가 생성될 경우 O(N)의 탐색 시간이 걸리게 됩니다.



성능 측정은 백준온라인 저지의 문제를 풀면서 이야기를 해보죠.

https://www.acmicpc.net/problem/7890

사실 문제 내용은 위에서 모두 언급했습니다. NN을 찾는 문제입니다.

점의 개수가 최대 100000개 이므로 점 하나를 처리하는데 O(logN)으로 처리해야 통과 할 수 있음을 알 수 있습니다. kd 트리를 이용하면 가장 가까운 이웃을 찾는데 평균적인 경우O(logN)이 걸리므로 이를 만족합니다.



소스 코드는 https://github.com/rlatkddn212/algorithm-problem/blob/master/Algorithm/7890.cpp 에서 보실수 있습니다.



#### 참고 문헌

https://en.wikipedia.org/wiki/K-d_tree

