# computeIfAbsent, computeIfPresent

Map Collection Framework에서 사용되는 메서드 중 최근 SSE를 구현하다가 모르는 메서드가 있기에 정리하기 위해 글을 작성한다.

## computeIfAbsent
- 해당 키가 존재하는지 판단하여, 존재하는 경우에 해당 키에 대응하는 값을 반환하고 존재하지 않는다면 지정된 함수를 사용하여 값을 계산하고 새로운 키와 계산된 값을 맵에 추가한다.
- 시그니처 메시드: `V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)`
  - key: 맵에 대응하는 값을 가져올 키
  - mappingFunction: 키가 맵에 존재하지 않는 경우 호출되는 함수로, 새로운 값을 계산하여 반환해야 한다. 이 함수는 key를 입력으로 받아서 V 타입의 값을 반환.

## computeIfPresent
- 해당 키가 존재하는 경우에만 지정된 함수를 사용하여 해당 키에 대응하는 값을 계산하고 업데이트하는 메서드.
- 시그니처 메서드 : `V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)`
    - key: 맵에 대응하는 값을 업데이트할 키
    - 키가 맵에 존재하는 경우 호출되는 함수로, 새로운 값을 계산하여 반환해야 한다. key와 해당 키에 대응하는 현재 값 V를 입력으로 받아서 V 타입의 새로운 값을 반환
