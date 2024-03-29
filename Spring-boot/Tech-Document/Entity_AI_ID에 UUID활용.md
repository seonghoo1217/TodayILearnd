# Primary Key에 UUID사용

나는 모든 프로젝트에서 Entity의 AI(Auto Increment)되는 Primary Key 즉, ID값을 Long 자료형을 사용하여 최대 64 bits의 수를 보장하기에
회원 Entity에서 이를 사용한다면 회원의 증가에 대비할 수 있고 이를 수로 환산하면 최대  18,446,744,073,709,600,000의 사람의 데이터를 DB에 저장할 수 있다.

하지만 최근들어 이를 String 자료형(DB에서는 varchar)으로 두어 UUID를 사용하는 방법을 사용해보면 어떨까? 라는 생각을 가지게 되었다.
그러기 위해선 우선 UUID를 알아보자.

## UUID

- UUID는 Universal Unique IDentifier의 약자로 범용적인 고유식별자라는 뜻이다.
- 개체들을 식별하고 구별하기 위한 용도로 사용되며 고유성을 충족시키는 방법으로 사용된다.
- 총 랜덤한 16바이트 총 128bit의 값을 산출한다.
- 총 340,282,366,920,938,463,463,374,607,431,768,211,456개의 사용가능한 UUID를 운용할 수 있다.

그렇다면 우리는 여기서 어떤 키워드에 주목을 해야하는가? 그것은 바로 **랜덤한** 값을 산출 한다는 것이다.
랜덤하다는 것은 개발자의 역량으로 알고리즘의 수정이 가능하다는 것이아닌 말그대로 난수적인 값이 나온다.
이 때 우리는 한가지 고민이 자연스럽게 따라온다. "UUID는 중복될 가능성이 없나?"

uuid v4는 생성 원리상 중복이 가능하다. 다만 그 중복될 가능성이 매우 희박하다. 총 340,282,366,920,938,463,463,374,607,431,768,211,456 개의 uuid가 생성 가능한데 이 중 중복되는 uuid가 생성될 가능성이 얼핏 봐도 희박해보인다.
그래도 불안요소가 존재하는 서비스를 그대로 사용하는 것은 서버개발자로서 적어도 Runtime에서의 에러는 지양해야하는것이 지당하다고 생각하기에 확률적으로는 0.00000000006지만 가능성이 없는 것은아니다.

하지만 사실상 이는 천재지변과 같은 수준이기에 다시 이 글의 본질로 돌아와보자

## Long을 UUID로 대체?

지금와서 생각해보지만 조금은 멍청한 생각이 아닌가 싶다. 애초에 해당생각을 하게된 경위 자체가 어 DB가 다차게 되면 어떡하지 같은 안일한 생각이었다.
하지만 Long 또한 경의 단위까지 가는 방대한 범위의 자료형이며, 이후 만약 다쓰게 된더라도 DB분산 또는 리팩터링을 거치면 되기에 전제가 잘못된 주니어 개발자의 망상이었다.