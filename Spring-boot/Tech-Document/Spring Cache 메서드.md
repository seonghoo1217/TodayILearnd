## 시작하기에 앞서
Cache에 대한 개념을 견고하게 한번 잡고가보자.

* Cache : 자주 사용하는 데이터를 저장하여 재활용하는 기술

위와같이 캐시는 자주 사용되는 데이터를 고속 데이터 스토리지(접근이 빠른)에 저장한다.
그래서, 최초 접근 이후 데이터에 재접근시 더 빠른 처리가 가능해진다.

동일한 리소스 대해 Select 작업이 빈번할때 사용한다면 DB의 부하를 전체적으로 줄일 수 있다.

## Cache Abstraction
트랜잭션과 마찬가지 메서드에 캐싱되는 과정이 실행과정에 투명하게 적용되며, 캐시 API 코드에 추가하지 않더라도 캐시기능을 제공한다.

즉, 캐시의 종류나 서비스가 달라지더라도 애플리케이션 코드에 영향을 끼치지 않는다.


## Cache Manager
캐시의 추상화기술을 사용하기 위해선 캐시매니저가 Bean에 등록되어 있어야한다.


**ConcurrentMapCacheManager**
- ConcurrentHashMap을 이용해 캐시 기능을 구현하는 간단한 캐시 매니저
- 캐시 정보를 Map 타입으로 메모리에 저장해두기 때문에 빠르고 별다른 설정이 필요없다는 장점이 있지만, 실제 서비스에서 사용하기에는 기능이 빈약하다.

**SimpleCacheManager**
사용할 캐시를 직접 등록하여 사용하기 위한 캐시 매니저

**EhCacheCacheManager**
- 자바에서 유명한 캐시 프레임워크 중 하나인 EhCache를 지원하는 캐시 매니저

**CaffeineCacheManager**
- Java 8로 Guava 캐시를 재작성한 Caffeine 캐시를 사용하는 캐시 매니저

**CompositeCacheManager**
- 한 개 이상의 캐시 매니저를 사용하도록 지원해주는 혼합 캐시 매니저입니다.

## @EnableCaching
Cache관련 config을 정의할 때 사용한다.

## @Cacheable
적용된 메소드의 리턴값을 기준으로 캐시에 값을 저장
