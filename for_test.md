# 시험을 위한 고급 알고리즘 정리

## 목차
 1. Lazy propagation
 2. Bit masking
 3. 카라츠바 알고리즘
 4. 타일 채우기
 5. 네트워크 플로우

## Lazy propagation 
- 의미<br>게으른 전파로 **O(logN)** 만에 Segment Tree에서 구간 업데이트를 달성하는 알고리즘
- update 
```cpp
//i~j 구간에 diff만큼 더해줄 때 SegmentTree를 업데이트 하는 함수
void update_range(int node, int start, int end, int i, int j, ll diff)
{
  //lazy가 남아있을 때
  if(tree[node].lazy != 0)
  {
    tree[node].value += (end-start+1)*tree[node].lazy; //자신에게 lazy 반영 
    if(start != end){ //자식 노드가 있을 때
    tree[node*2].lazy += tree[node].lazy; //왼쪽 자식 노드에게 lazy 기록
    tree[node*2+1].lazy += tree[node].lazy; //오른쪽 자식 노드에게 lazy 기록
    }
    tree[node].lazy =0; //자신의 lazy 0으로 초기화: lazy 중복 사용 막기
  }
  
  //구간에 걸쳐있지 않은 노드면 걍 빠꾸
  if(j < start || i > end) return;
  
  //구간 안에 속해 있는 노드
  if(i <= start && end <= j)
  {
    tree[node].value += (end-start+1)*diff; //update 반영
    if(start != end)//자식 노드가 있을 때
    {
      tree[node*2].lazy += diff;//왼쪽 자식 노드에게 lazy 기록
      tree[node*2+1].lazy += diff;//오른쪽 자식 노드에게 lazy 기록
    }
    return; //빠끄
  }
  
  //구간에 걸쳐있는 노드일 경우
  update_range(node*2, start, (start+end)/2, i, j, diff); //왼쪽 자식으로 가기
  update_range(node*2+1, (start+end)/2+1, end, i, j, diff); //오른쪽 자식으로 가기
  tree[node].value = tree[node*2].value+tree[node*2+1].value; //두 자식의 합으로 자신의 값 업데이트(lazy가 아래로만 전파되도 되는 이유)
}
```
- sum
```cpp
//i번째 값부터 j번째 값 까지의 합을 구하는 함수
ll sum(int node, int start, int end, int i, int j)
{
  //lazy가 남아있을 때
  if(tree[node].lazy != 0){
    tree[node].value += (end-start+1)*tree[node].lazy; //lazy 반영 
    if(start != end){ //자식노드가 있을 때 
      tree[node*2].lazy += tree[node].lazy;//왼쪽 자식 노드에게 lazy 기록
      tree[node*2+1].lazy += tree[node].lazy;//오른쪽 자식 노드에게 lazy 기록
    }
    tree[node].lazy =0;//자신의 lazy 0으로 초기화: lazy 중복 사용 막기
  }
  
  //구간에 걸쳐있지 않은 노드면 걍 빠꾸
  if(i> end || j < start) return 0;
  
  //구간 안에 속해 있는 노드
  if(i <= start && end <= j) return tree[node].value;
  //구간에 걸쳐있는 노드일 경우
  return sum(node*2, start, (start+end)/2, i, j)+sum(node*2+1, (start+end)/2+1, end, i, j);//양쪽 자식으로 가기
}
```
- 핵심
   1. lazy가 자식 노드에게 전파될 때 = 부모 노드에 방문했을 때
   2. 문제 없는 이유: 자손 노드를 가려면 무조건 조상 노드를 거쳐야 함
   3. 자식에서 부모로의 업데이트는? update 함수 최하단에 있음!
- [출처](https://bowbowbow.tistory.com/4)

## Bit masking
- 의미
   + 비트 단위로 값을 보존하는 것
   + 메모리 절약 가능
   + 예시: visit 배열 대신 사용할 수 있음
- 비트 연산
   1. **&**(and) : 둘 다 1인 경우 1 / else 0<br> ex) 1011 & 1101 = 1001
   2. **|**(or) : 둘 중 하나 혹은 모두 1인 경우 1 / else 0<br> ex) 1011 | 1101 = 1111
   3. **^**(xor) : 둘이 다르면 1 / else 0<br> ex) 1011 ^ 1101 = 0110
   4. **~**(not) : 1은 0으로, 0은 1로<br> ex) ~1011 = 0100 
- 연산의 활용
   1. a에서 k번 비트 확인하기
      - a & (1<<k)
      - 1101 & (1<<3) = 0100
      - 1101 & (1<<2) = 0000
   2. a에서 k번 비트 1로 만들기
      - a | (1<<k)
      - 1001 | (1<<3) = 1101
      - 1001 | (1<<2) = 1011
   3. a에서 k번 비트 0으로 만들기
      - a & ~(1<<k)
      - 1111 & ~(1<<3) = 1011
      - 1111 & ~(1<<2) = 1101
      
## 카라츠바 알고리즘
- 의미
   + 큰 두 수의 곱을 자릿수가 절반인 수들의 곱 세 번과 덧셈, 시프트 연산으로 계산
   + 일반적인 곱셈을 하면 시간 복잡도 O(log n^2)
   + 카라츠바 알고리즘으로는 시간 복잡도 O(n^log 3)
- 알고리즘은
   [위키를 봅시다](https://ko.wikipedia.org/wiki/카라추바_알고리즘)

## 타일 채우기
- [problem](http://koistudy.net/?mid=prob_page&NO=2281&SEARCH=0)
- 이현지 학생의 좋은 코드
```cpp
#include<stdio.h>
int tile(int x) //대칭 고려 안하고 그냥 채우는 경우의 수 
{
    if(x<0) return 0;
    if(x==0||x==1) return 1;
    return tile(x-1)+2*tile(x-2);
}

int f(int x) //대칭 제거한 경우의 수
{
    if(x<=1) return 0;
    if(x%2==1) return tile(x)-tile((x-1)/2); //개수 홀수면: 가운데에 1*2 끼우기
    else return tile(x)-tile(x/2)-2*tile((x-2)/2); //개수 짝수면: 반띵 + 가운데에 2*1 둘 또는 2*2 끼우기
}

int main()
{
    int n;
    scanf("%d",&n);
    printf("%d",f(n));
}
```

## 네트워크 플로우
- 해야하는건지 잘 모르겠어요 근데 내용도 잘 모르겠넹
- 의미
   + 그래프에서 각 노드들 간의 용량 (Capacity)이 정의되어 있을 시, 시작점(source)에서 끝점(target)까지 흐를 수 있는 최대 유량을 구하는 것
- 용어
   + **소스(S, source)** : 시작점
   + **싱크(T, sink)** : 끝점
   + **정점(Vertex)** : 유량이 모이는 위치
   + **간선(Edge)** : 유량이 흐르는 파이프 역할
   + **용량(Capacity)** : 유량이 흐를 수 있는 크기
   + **유량(Flow)** : 간선에 흐르는 현재 유량의 크기
   + **잔류 유량(Residual Flow)** : Capacity - Flow로서 현재 간선에 흐를 수 있는 유량이 얼마나 인 지를 나타냄
   + **c(u, v)** : c는 Capacity를 의미, u에서 v로 흐를 수 있는 간선의 용량을 의미한다.
   + **f(u, v)** : f는 Flow를 의미하고, u에서 v로 흐른 실제 유량을 뜻한다.
- 핵심 성질
   + 특정 경로를 따라 유량을 보낼 때는 그 경로에 포함된 간선중 **가장 용량이 작은 간선** 에 의해 결정된다.
   + **용량 제한 속성** : f(u,v) <= c(u,v)
   + **유량의 대칭성** : f(u,v) = -f(v,u)
   + 나오는 유량과 들어오는 유량의 합은 항상 같아야 한다. (source, sink 예외)
- 경로 찾고 왔다갔다 하는건 [이거 읽읍시다](https://engkimbs.tistory.com/353)
- 아마 내용 설명해주고 그래프 주고 과정 따라 선 그려보라고 할 듯...?
