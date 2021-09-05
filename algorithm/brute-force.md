완전탐색(Exhaustive Search)
===========

## 완전탐색 알고리즘이란?
완전탐색은 가능한 **모든 경우의 수를 다 체크해서 정답을 찾는 방법** 이다. 즉, 무식하게 가능한 거 다 해보겠다는 방법을 의미한다.  
이 방법은 무식하게 한다는 의미로 "Brute Force"라고도 부르며, 직관적이여서 이해하기 쉽고 문제의 정확한 결과값을 얻어낼 수 있는 가장 확실하며 기초적인 방법이다.
예를 들면, 4자리의 암호로 구성된 자물쇠를 풀려고 시도한다고 생각해보면 반드시 해결할 수 있는 가장 확실한 방법은 0000 ~ 9999까지 모두 시도해보는 것이다. (최대 10,000번의 시도로 해결 가능)  


하지만, Computer Science에서 문제 해결 알고리즘을 사용할 때는 **기본적으로 2가지 규칙을 적용**한다.  
1. 사용된 알고리즘이 적절한가? (문제 해결이 가능한지)
2. 효율적으로 동작하는가?
위 두가지의 규칙에 대해서 생각할 때, 1번은 만족될 수 있는 가장 확실한 방법이나 대부분의 경우 2번의 경우 때문에 이 방법이 사용되는데 제한이 따른다.  



### 완전탐색 기법을 활용하는 방법
우선 완전탐색 기법으로 문제를 풀기 위해서는 다음과 같이 고려해서 수행한다.  
1. 해결하고자 하는 문제의 가능한 경우의 수를 대략적으로 계산한다.
2. 가능한 모든 방법을 다 고려한다.
3. 실제 답을 구할 수 있는지 적용한다.


여기서 위의 모든 방법에는 다음과 같은 방법들이 있다.  
1. Brute Force 기법: 반복/조건문을 활용해 모두 테스트하는 방법
2. 순열(Permutation): n개의 원소 중 r개의 원소를 중복 허용 없이 나열하는 방법
3. 재귀 호출
4. 비트마스크: 2진수 표현 기법을 활용하는 방법
5. BFS, DFS를 활용하는 방법


### 각 방식에 대한 설명
1. Brute Force 기법
이 방법은 반복/조건문을 통해 가능한 모든 방법을 단순히 찾는 경우를 의미한다.
예를 들어, 위의 자물쇠 암호를 찾는 경우와 같이 모든 경우를 다 참조하는 경우가 그러하다.  


2. 순열(Permutation)
우선 순열이 무엇인지 이해하자. 순열은 **임의의 수열이 있을 때 그것을 다른 순서로 연산하는 방법** 을 의미한다.  
즉, 순서가 중요하다! 만약 수열에서 숫자 [1,2,3]이 있다면 이것을 [1,2,3]으로 보는 순서와 [3,2,1]로 보는 순서가 차이가 있음이 중요한 경우를 의미한다.  

같은 데이터가 입력된 수열이지만, 그 순서에 따라 의미가 있고 이 순서를 통해 연결되는 이전/다음 수열을 찾아낼 수 있는 경우를 계산할 수 있다.  
그래서, 만약 N개의 서로 다른 데이터가 있고 이를 순열로 나타낸다면 전체 순열의 가지 수는 N!개가 된다.  
최초에 N개 중 1개가 올 수 있고 그 이후에는 각각 N-1, N-2, ..., 1개가 올 수 있어 이를 모두 곱하면 N!이 되기 때문이다.  
[1,2,3]을 사전 순으로 나열하는 순열이 있다고 가정해보자. 그렇다면 아래와 같이 나열될 수 있을 것이다.  
[이미지]
{1 2 3} → **최초 순열 - 오름차순**   
{1 3 2}  
{2 1 3}  
{2 3 1}  
{3 1 2}  
{3 2 1} → **최종 순열 - 내림차순**  

위 내용과 같이 순열을 나열할 수 있는데, 최초/최종 순열을 보면 숫자가 오름/내림차순인 것을 알 수 있다. (중복된 숫자가 있을 경우 비내림/비오름차순으로 된다)
여기서 사전 순 순열의 규칙을 알아낼 수 있는데 N개의 데이터가 있고 1~i번째 데이터를 설정했을 때, i번째 데이터 기준 최종 순열은 i+1부터 N까지가 모두 내림차순이라는 것이다.  


* 순열을 구현하는 방법
현재 순열을 구성하는 배열을 A라고 하고 i,j는 그 배열의 index값을 의미한다고 하자. 예를 들어 A={7,2,3,6,5,4,1}이고 i,j는 각각의 index 값이다.  
아래에서는 현재의 다음 순열을 구하는 로직을 기반으로 설명한다.  
1. A[i-1] <= A[i]를 만족하는 i중 가장 큰 값을 찾는다. (혹은 뒤에서부터 찾는 경우 A[i-1] >= A[i]중 가장 작은 i를 찾는다.)  
   → 현재 i값을 기준으로 이후는 모두 내림차순으로 되는 경우를 찾는다는 것이다. 현재 기준 최종 순열을 찾음
     A배열을 보면 A[i-1] < A[i]가 되는 가장 큰 i는 6인 3번째(0부터 시작)이다. 즉, i=3이 된다.
2. j >= i 중, A[j] > A[i-1]을 만족하는 가장 큰 j의 값을 찾는다.
   → 현재가 최종 순열 상태이므로, i-1번째 숫자를 변경하여 최초 순열을 찾아야 한다.
     A배열을 기준으로 i-1번째 숫자는 3으로 3보다 큰 경우는 6,5,4이나 그 중 j값이 가장 큰 경우는 4이다.  
3. A[i-1]과 A[j]를 Swap한다.
   → i-1인 2번째 숫자 3과 j인 5번째 숫자 4를 변경한다. A배열은 다음과 같이 변경된다.
     A = {7,2,4,6,5,3,1}
4. i이후의 순열을 모두 뒤집는다.
   → 최초 순열 상태로 만들어야 하므로 i번째부터는 오름차순으로 만들어야 한다. A배열은 다음과 같이 변경된다.
     A = {7,2,4,1,3,5,6}


위 로직의 시간복잡도는 어떨까? 전체 N개의 숫자에 대해 각각 순열을 구하는 문제가 된다.  
**제일 좌축에 숫자 N개가 올 수 있고 각 자리마다(N-1)!개의 순열** 이 있기 때문에 시간복잡도는 O(N x (N-1)!)이라서 O(N!)이 된다.  

N!은 굉장히 높은 시간 복잡도를 갖는다. N=10이면 약 360만번의 연산이 수행되며 N=11이 되면 4억번의 연산이 필요하다.  
따라서 이 방법은 항상 사용할 수는 없고, 문제에서 주어진 N의 크기를 고려해야 한다.  

```
import java.io.BufferedWriter;
import java.io.OutputStreamWriter;

class Main {
    static int[] perm;
    static int n = 4;
    public static void main(String[] args) {
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

        perm = new int[]{1, 2, 3, 4};
        
        while (get_next_perm()) {
          for(int num : perm) {
            bw.write(String.valueOf(num) + " ");
          }
          bw.write("\n");
        }
        bw.flush();
    }

    private static boolean get_next_perm() {
        int i = n-1;

        //뒤에서부터 체크하여 현재 위치가 뒤에서부터 오름차순이 아닌 경우를 찾음
        //뒤에서부터 오름차순이라는 것은 사전 순으로 최종 수열이라는 의미임
        while (i > 0 && perm[i-1] >= perm[i]) i--;

        //이미 자체적으로 최종 순열이라면 false를 반환
        if (i <= 0) return false;

        //j는 현재 i-1위치에서 시작
        //i-1 이후의 모든 숫자 중 큰 것을 고르는데 그 중, j의 값이 가장 큰 경우로 선택
        int j = i-1;
        while (j < n-1 && perm[j+1] > perm[j-1]) j++;

        //j와 i-1번째의 숫자를 swap
        swap(i-1, j);

        //i번째부터 이후의 모든 숫자를 뒤집음
        //i~n-1 사이의 숫자를 상호 뒤집어야 하므로 j값을 n-1로 초기화
        j = n-1;
        while (i < j) {
            swap(i, j);
            i += 1;
            j -= 1;
        }
        return true;
    }

    private static void swap(int i, int j) {
        int temp = perm[i];
        perm[i] = perm[j];
        perm[j] = temp;
    }
}
```
순열 관련 자료는 [뱀귤님 블로그의 순열](https://bcp0109.tistory.com/14) 정리가 참 잘되있다 :)  


3. 재귀(Recursive)
재귀는 말 그대로 자기 자신을 호출한다는 의미이다. 왜 자기 자신을 호출할 필요가 있을까?  
만약 숫자 N개의 숫자 중 M개를 고르는 경우라고 할 때 N과 M이 매우 큰 숫자라면 어떨까? 반복문을 수백, 수천개를 써 내려갈 수는 없다.  
이를 **재귀함수를 활용** 한다면 자기 자신을 호출함으로써 다음 숫자를 선택할 수 있도록 이동시켜 전체 코드를 매우 짧게 줄일 수 있다.  
아래와 같이, 1~100의 숫자 중 5개를 선택하는 경우를 코드로 작성해보자.  

```
class Main {
    static int limit = 100; //1~100
    static int n = 5; //5개만 선택
    public static void main(String[] args) {
        int[] chosen = new int[n]; //선택된 숫자가 저장되는 배열

        //시작은 0부터 시작하며 0개를 현재 선택했으니 아래와 같이 파라미터 전달
        solution(chosen, 0, 0);
    }

    private static void solution(int[] chosen, int current, int cnt) {
        //chosen은 선택된 숫자가 저장된 배열
        //current는 현재 숫자를 선택하는 index
        //cnt는 몇 개의 숫자가 선택되었는지 확인

        //n개의 숫자를 다 선택했다면 출력 후 더 이상 재귀를 돌지 않아야 한다
        //escape 조건 정의
        if (cnt == n) {
            for (int i : chosen) {
                System.out.print(i + " ");
            }
            System.out.println();
            return;
        }

        for (int i=current+1; i<=limit; i++) {
            //현재 선택된 숫자를 저장
            chosen[cnt] = i;

            //다음 숫자를 선택하기 위해 재귀 호출
            solution(chosen, current, cnt+1);
        }
    }
}
```

여기서 중요한 점은 **재귀를 탈출하기 위한 탈출 조건이 필요**, **현재 함수의 상태를 저장하는 파라미터가 필요**, **return문을 신경쓸 것**  

4. 비트마스크
비트마스크 관련 내용은 생략하겠습니다.  


5. BFS, DFS 사용하기
이는 **그래프 자료 구조에서 모든 정점을 탐색하기 위한 방법**이다.  
BFS는 너비 우선 탐색으로 현재 정점과 인접한 정점을 우선으로 탐색하고 DFS는 깊이 우선 탐색으로 현재 인접한 정점을 탐색 후 그 다음 인접한 정점을 탐색하여 끝까지 탐색하는 방식이다.  
6. 










