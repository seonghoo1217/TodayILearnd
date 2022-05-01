# 자바 API 에서 FIFO 처리

## FIFO란?
* First In First Out이라는 뜻으로 자료구조형에서 가장 먼저 넣은 값이 순서대로 출력되는 구조를 뜻한다. 예시로는 큐가 있다.
* 반대로 FILO(First In Last Out) 구조가 존재하며 스택이 이 예시에 해당된다.

## FIFO 메서드 종류

* Queue 데이터 추가 메서드: add(e),offer(e)

* Queue 데이터 삭제 메서드 : remove(),poll()

* Queue 데이터 검사 메서드:element(),peek()

## 비슷하지만 다른 메서드
add의 경우 Queue가 꽉 찬 경우 예외 처리를 발생시킨다. 하지만 offer의 경우 반환값이 boolean으로 값을 더 넣을 수 없다면 false를 리턴한다.

