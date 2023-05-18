[##_Image|kage@bFdlQ8/btsgfsBEMIz/lEM0JNlaLBIZDw2krEwyS0/img.png|CDM|1.3|{"originWidth":1051,"originHeight":750,"style":"alignCenter"}_##]

## **소개 배경**

원래 알고리즘 풀이에 대한 블로그 포스팅에는 큰 의미를 두지 않았지만, 해당문제의 경우 문제의 요구사항이 꽤나 복잡하다고 생각이 들어, 헤매는 사람들을 도와주고자 포스팅하게 되었다.

## **요구 사항 분석**

우선 해당 문제의 경우 우선적으로 콜라츠의 추측을 알아야한다.

> **💡 콜라츠의 추측**
>
> 콜라츠의 추측이란 생각보다 간단하다. 프로그래머스에도 설명으로 나와있지만 int형의 **`양의 정수`** 가 주어졌을 때 이를 조건부 연산을 통해 무조건 1로 만들 수 있다는 추측이다.

```
public void collatzConjecture(int k){
    while(k>1){
        if(k%2==0) k/=2;
            else k=(k*3)+1;
    }
}
```

이때 최초 K값의 X좌표는 0이고 이후 1씩 증가한다. 또한 K값 즉, Y좌표 또한 콜라츠 추측에 의해 값이 계속 변하게된다.

해당 점들을 좌표에 그려내는 것이 **해당 문제의 핵심**이다.

만약 K가 7일 경우를 예로 들어보자 처음 값은 **(x=0,y=7)** 일 것이다.

다음 연산에는 K가 22가되어 **(x=1,y=22)** 이다.

이후 1씩 증가하며 그래프를 그려본다면 아래와 같다.

[##_Image|kage@bbmNyW/btsglNRYY92/K3hiPgqyehgzg21DETEh3k/img.png|CDM|1.3|{"originWidth":724,"originHeight":941,"style":"alignCenter"}_##]

여기까지 진행했다면 거의 정답에 근사했다고 볼 수 있다.

자, 해당문제는 콜라츠의 추측을 진행하는 것이 아닌, 이를 이용하여 생성된 그래프의 정적분을 구하는 것이다.

여기서 사진을 보고 눈치 챈사람이 존재하겠지만, 해당 그래프의 정적분을 구하려면 각 그래프가 꺽이는 지점(X가 1씩커지는 지점)에서 선을 내려본다면, 그래프는 X좌표 1당 **사다리 꼴** 형태를 띄게된다.

그렇다면 하나의 좌표 예를 들어 x=0 부터 x=1 까지의 사다리 꼴 넓이를 구한다면 X 좌표 하나에 대한 그래프의 넓이(총 그래프의 넓이를 N이라 가정하였을 때 1/N)를 구할 수 있다.


> 💡 사다리 꼴 넓이
>
> 사다리 꼴의 넓이를 구하는 공식은 알다시피 ((윗변+아랫변)*높이)/2 이다.
> 하지만, 높이에 해당하는 X 좌표는 1의 단위로 나누어지기에 사실상 공식은 (윗변+아랫변)/2 이다.

그렇다면 콜라츠의 추측으로 생겨나는 생겨나는 좌표 값들이 끝나는 지점의 X 좌표를 XN이라고 칭했을 때, 0부터 XN까지의 합을 구한다면,
range [0,0]에 해당하는 값을 구한것이다.

결국 range는 [1,-2]가 주어진다면 1부터 XN-2까지의 사다리 꼴 넓이의 합을 구하면된다.

좀더 명확한 예시를 위해 프로그래머스 예시를 기준으로 살펴보자

![image](https://github.com/DY-WhatSong/BE-What_Song/assets/39437170/d473aefe-fe23-4f22-bab3-f42685b6f82a)

해당 그래프의 경우 0(초기값)부터 5(최종적으로 1이 되는)까지 총 6개의 지점이 발생한다.

이 때 range가 [0,-3]이라면 [0,(6 + -3)] 인 것이기에 최종적으로 [0,3]이 되고각지점의 값인 28.5가 정답이 될 것이다.

이 점을 유의하며 문제를 풀이한다면 쉽게, 풀어갈 수 있다.

## **코드 및 설명**

```java
    static List<Integer> point=new ArrayList<>();
```
각 그래프가 꺽이는 지점, 즉 Point를 저장하기 위한 Int 형의 List를 선언한다.
지점의 경우 k가 10000까지의 수가 대입되므로 Int형으로 충분하다고 생각하였다.

다음으로는 콜라츠의 추측을 실행시킨다.

```java
		point.add(k);
		while (k>1){
			if (k%2==0) k=k/2;
			else k=(k*3)+1;

			point.add(k);
			cnt++;
		}
```
최초값이 그래프에 그려져야하기에 리스트에 추가하고 추측이 이루어지며 변형되는 값들 또한 추가한다.

여기서 중요한 부분은 1이되는 X좌표를 알기위해 cnt라는 변수를 반복문동안 증가시켜준다.


이후 주어진 ranges 변수동안 반복문을 돌며, 문제에서 제시한 요구사항인 범위에 대한 처리를 진행하여준다.

```java
	for (int i=0;i<ranges.length;i++){
			int x = ranges[i][0];
			int y = cnt+ranges[i][1];
			if(x==y) answer[i]=0.0;
			else if(x>y) answer[i]=-1.0;
			else {
				answer[i]=getDefiniteIntegral(x,y);
	}
```
요구되는 좌표가 예를 들어 [2,2]로 같은 수라면, 범위를 구할 수 없기에 0.0 값을 정답 배열에 추가하여준다.
또는 [3,2]와 같이 좌측좌표가 우측좌표보다 클 수는 없으므로 해당경우 -1.0을 정답배열에 추가한다.

마지막으로는 정상적인 좌표가 주어졌다면 연산을 시키기 위해 내부 메소드로 분리한 getDefiniteIntegral로 x값과 y값을 넘겨준다.

```java
public static double getDefiniteIntegral(int x,int y){
		double sum=0.0;
		for (int i=x;i<y;i++){
			double point1 = point.get(i);
			double point2 = point.get(i + 1);
			sum+=(point1+point2)/2;
		}
		return sum;
	}
```
미리 앞서 구한 point 리스트에서 현재 지점과 +1 지점의 넓이를 사다리꼴 공식을 이용하여 구한 후 누적합을 return하여 주면 구현은 끝이다.

**전체 코드**
```java
import java.util.*;
class Solution {
    static List<Integer> point=new ArrayList<>();
	public double[] solution(int k, int[][] ranges) {
		double[] answer = new double[ranges.length];
		int cnt=0;
		point.add(k);
		while (k>1){
			if (k%2==0) k=k/2;
			else k=(k*3)+1;

			point.add(k);
			cnt++;
		}
		
		for (int i=0;i<ranges.length;i++){
			int x = ranges[i][0];
			int y = cnt+ranges[i][1];
			if(x==y) answer[i]=0.0;
			else if(x>y) answer[i]=-1.0;
			else {
				answer[i]=getDefiniteIntegral(x,y);
			}
		}
		return answer;
	}

	public static double getDefiniteIntegral(int x,int y){
		double sum=0.0;
		for (int i=x;i<y;i++){
			double point1 = point.get(i);
			double point2 = point.get(i + 1);
			sum+=(point1+point2)/2;
		}
		return sum;
	}
}
```

## **결론**
요구사항이 까다로울 뿐 구혀자체는 상당히 간단한 문제였다. 역시 코딩테스트는 요구사항을 잘 분석한다면 어떻게 구현하여야할지가 잘보이는 것같다.