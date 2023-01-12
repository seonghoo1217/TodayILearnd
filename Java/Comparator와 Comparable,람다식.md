# Comparator와 Comparable, Lambda

셋 모두 배열 또는 List의 형태를 정렬할 때 사용되는 인터페이스와 기법이다. 
최근 알고리즘 풀이를 학습하며 더 나은 기법에 관한 고찰을 작성해보려한다. 

## Comparator와 Comparable

우선 자세한 설명에 앞서
Comparator와 Comparable의 자세한 설명은 아래링크를 참고하길 바란다.

[Comparator와 Comparable](https://github.com/seonghoo1217/Algorithm/blob/master/src/algorithm/java/comparable/Comparable%20%EC%A0%95%EB%A6%AC.md)

Comparator와 Comparable은 모두 인터페이스로 객체들을 정렬 또는 이진검색트리를 구성하는데 필요한 메서드를 정의한다.

- Comparable : 인터페이스를 구현한 객체 스스로에게 부여하는 한 가지 기본 정렬 규칙을 설정하는 목적으로 사용
- Comparator : 이 인터페이스를 구현한 클래스는 정렬 규칙 그 자체를 의미하며, 기본 정렬 규칙과 다르게 원하는대로 정렬 순서를 지정하고 싶을 때 사용

## 람다식
람다식은 그 의미 자체가 워낙 거대하기에 알고리즘을 사용하는데 있어서 장점을 위주로 설명을 풀어나가겠다.
람다식은 기본적으로 익명함수를 활용하여 코드의 간결성을 높이는 목적으로 활용한다.

이를 예시코드를 통해 코드적으로 분석해보자.

예시코드는 백준 문제 `11650번 좌표 정렬하기`에서 가져왔다.

해당문제를 보면 랜덤하게 받은 수들을 크기에 따라 정렬하는 문제이다.
여기서 보통이라면 Comparator 인터페이스의 `compare` 메소드를 구현하여 정렬기준을 세워 그에 맞추는게 정석적인 풀이방법이다.

```java
        Arrays.sort(xy, new Comparator<Integer[]>() {

			@Override
			public int compare(Integer[] o1, Integer[] o2) {
				if (o1[0].equals(o2[0])){
					return o1[1]-o2[1];
				}else {
					return o1[0]-o2[0];
				}
			}
		});
```

이 코드는 물론 통과하는 코드이며, 알고리즘 풀이 자체에 문제 또한 존재하지 않는다.
하지만, Comparator라는 새로운 객체에 대한 메모리할당과 Wrapper 클래스를 선언하는데 드는 비용 등이 요구되므로 개발자입장에서는
'살짝 무언가 아쉬운데? 좀더 효율성을 추구할 수 있지 않을까?'라는 생각을 할 수 있다.
참고로, `Integer`라는 Wrapper 클래스를 사용하는 값끼리 `==`비교를 할 경우에는 equals 메소드를 사용해주어야한다.

그러면 이를 람다로 바꾸어보면 어떻게 될까?

```java
Arrays.sort(xy,(x1,x2)->{
			if (x1[0]==x2[0]){
				return x1[1]-x2[1];
			}else {
				return x1[0]-x2[0];
			}
		});
```

놀라울 정도로 간단해지고, 코드의 명시성이 높아지게된다.
우선적으로 Comparator의 선언 자체가 없어지고, Wrapper 클래스를 사용하지 않으므로서 메모리에서 이득도 보게된다.
게다가 Comparator를 선언하여 compare 메소드를 사용하기에 객체의 메소드에 접근하는 비용과 메소드 이용시 지불 비용도 아낄 수 있다.

하지만 람다의 사용이 무조건적으로 이득인 것은 아니나, 알고리즘 문제 풀이와 같이 작은 영역의 문제를 해결하는데 있어서는
이점을 크게 가져온다.