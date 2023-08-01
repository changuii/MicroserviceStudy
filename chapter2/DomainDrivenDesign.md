# 2.4 딱 도메인 주도 설계 만큼


마이크로서비스의 경계를 찾는 데 사용되는 기본 매커니즘은 도메인 모델을 생성하기 위해 도메인 주도 설계를 사용하며 도메인 자체를 중심으로 진행한다.  
이제 마이크로서비스의 맥락에서 DDD가 작동하는 방식을 좀 더 폭넓게 이해해보자.  

실제 세계처럼 표현하려는 우리 프로그램의 열망은 새로운 것이 아니다.  
시뮬라와 같은 객체 지향 프로그래밍 언어는 실제 도메인을 모델링할 수 있도록 개발됐다.  
하지만 이러한 생각이 실제로 구체화되려면 프로그래밍 언어 능력 이상의 것이 필요하다.  

에릭 에반스의 도메인 주도 설계는 프로그램에서 문제 영역을 더 잘 표현하는 중요한 개념들을 제시했다.  
그 개념을 깊이 다루는 것은 이 책의 범위를 벗어나지만, 강조할 가치가 있는 DDD의 일부 핵심 개념을 간략히 설명한다.  


`보편 언어`
- 의사소통을 돕기 위해 코드와 도메인 설명에 사용할 공통 언어를 정의하고 채택한다.  

`애그리거트`
- 객체들의 집합이며 일반적으로 실제 세계 개념과 관련된 하나의 개체로 관리된다.  

`경계 콘텍스트`
- 더 큰 범위의 시스템에 대한 기능을 제공하지만 복잡성을 숨기는 비즈니스 도메인 내부의 명시적인 경계이다.  



## 2.4.1 보편 언어

보편 언어는 사용자가 쓰는 용어를 코드에서 동일하게 사용하려고 노력해야 한다는 개념과 관련된다.  

98p

## 2.4.2 애그리거트

DDD에서 애그리거트는 다소 혼동되는 개념이자 다양한 정의가 존재한다.  
애그리거트는 단순히 임의의 객체 모음일까?  

데이터 베이스에서 꺼내야할 가장 작은 단위는 무엇일까?  
필자에게 항상 효과가 있었던 모델은 먼저 애그리거트를 실제 도메인 개념의 표현으로 생각하는 것이었다.  

주문, 송장, 재고 물품 등을 생각해보자.  
애그리거트는 일반적으로 수명주기가 있으므로 상태 기계로 구현할 수 있다.  

뮤직코프 도메인의 예에서 주문 애그리거트에는 주문의 품목을 나타내는 여러 행의 품목이 포함될 수 있다.  
이 품목 행은 전체 주문 애그리거트의 일부로만 그 의미가 있다.  

우리는 애그리거트를 독립된 단위로 취급하길 원해서 애그리거트의 상태 전환을 처리하는 코드와 상태를 함께 묶으려 한다.  
따라서 하나의 마이크로서비스는 여러 애그리거트를 관리할 수 있지만, 하나의 애그리거트는 하나의 마이크로서비스에서 관리돼야 한다.  

하나의 마이크로서비스는 하나 이상의 서로 다른 종류의 애그리거트에 대한 수명 주기와 데이터 저장소를 처리한다.  
다른 서비스의 기능이 애그리거트 중 하나를 변경하려는 경우 해당 애그리거트에 직접 변경을 요청하거나 애그리거트 스스로 시스템의 다른 구성요소와 상호작용 해서 상태 전환이 이뤄지도록 해야 하며, 예를 들면 다른 마이크로서비스에서 발행한 이벤트를 구독함으로써 가능할 것이다.  

무엇보다 외부 시스템이 애그리거트 내의 상태 전이를 요청하면 애그리거트가 거절할 수도 있다는 사실을 반드시 이해해야한다.  
이상적으로는 비정상적 상태 전환이 불가능하도록 애그리거트를 구현하고 싶다.  

애그리거트는 다른 애그리거트와 관련성을 가질 수 있다.  
하나 이상의 주문과 관련된 고객 애그리거트가 있다.  
이러한 애그리거트들은 같은 마이크로서비스나 다른 마이크로서비스에서 관리할 수 있다.  

애그리거트 간의 관계가 하나의 마이크로서비스 범위 내에 존재할 때 관계형 데이터베이스를 사용한다면 외래 키 관계와 같은 것을 이용해 쉽게 저장할 수 있다.  
하지만 이러한 애그리거트간의 관계가 마이크로서비스 경계를 넘어 걸쳐 있다면 관계를 모델링할 방법이 필요하다.  

이제 애그리거트 ID를 로컬 데이터베이스에 직접 저장할 수 있다.  
예를 들어 고객에 대한 거래를 저장할 재무 원장을 관리하는 재무 마이크로서비스를 생각해보자.  
로컬에서 재무 마이크로서비스의 데이터베이스 내부에 해당 고객의 ID를 가진 고객ID 열이 존재할 수 있다.  
해당 고객에 대한 추가 정보를 얻으려면 그 ID를 사용해 고객 마이크로서비스를 조회해야 한다.  

이 개념의 문제는 명확하지 않다는 데 있다.  
사실 고객ID 열과 원격 고객 간의 관계는 완전히 암묵적이다.  
그 ID가 어떻게 사용됐는지 알아보려면 재무 마이크로서비스 자체 코드를 살펴보아야 한다.  
외부 애그리거트에 대한 참조를 더 명확한 방식으로 저장할 수 있다면 좋을 것이다.  

고객을 참조하기 위해 평범한 ID 대신 REST기반 시스템을 만들 때 사용하는 URI를 저장한다.  
이 방식의 장점은 두 가지다.  
관계의 특성이 명시적이며 REST 시스템에서는 이 URI를 직접 역참조해 관련 자원을 조회할 수 있다.  
하지만 REST기반 시스템이 아니라면 어떻게 해야 할까?  

사운드 클라우드는 교차 참조를 위한 모조 URI 스킴을 고안했다.  

예를들어 soundcloud:tracks:123은 ID가 123인 트랙에 대한 참조다.  
이것은 식별자를 살펴보는 사람에게는 훨씬 더 명확하지만 필요한 경우 마이크로서비스 애그리거트 간 상호 조회를 쉽게 작성하는 코드를 고려할 수 있을 만큼 유용한 스킴이기도 하다.  

## 2.4.3 경계 콘텍스트  

경계 콘텍스트는 일반적으로 보다 큰 구조적 경계를 나타낸다.  
그 경계의 범위 내에서는 명시적인 책임을 수행할 필요가 있다.  
이 개념은 다소 모호하므로 구체적인 예를 하나 살펴보자.  

뮤직코프에서 창고는 주문 배송 관리, 신규 재고 수령, 지게차 트럭의 분주한 이동 등이 일어나는 활동의 중심지이다.  
다른 곳에서는 재무 부서가 그다지 재미있지 않겠지만, 뮤직코프의 조직 내에서는 급여를 처리하고 선적 비용을 지불하는 등 중요한 기능을 수행한다.  

경계 콘텍스트는 내부 구현 사항을 숨긴다.  
또한 내부에서만 고려하면 되는 사항도 있다. (창고 외부의 사람들은 어떤 종류의 지게차가 사용되는지에 전혀 관심이 없다.)  
이와 같은 내부 고려 사항은 다른 사람들이 알 필요도 없고 신경을 써도 안되므로 외부 세계에서는 완전히 감춰져 있어야 마땅하다.  

구현 관점에서 경계 콘텍스트는 애그리거트를 하나 이상 포함한다.  
일부 애그리거트는 경계 콘텍스트 외부에 노출될 수 있고, 어떤 것들은 내부에 숨겨져 있을 것이다.  
애그리거트와 마찬가지로 경계 콘텍스트 사이에도 관계가 존재할 수 있다.  
서비스에 매핑될 때 이러한 종속성은 서비스 간의 종속성이 된다.  

잠시 뮤직코프 비즈니스로 돌아가보자.  
도메인은 우리가 운영하는 전체 비즈니스다.  
즉, 창고 부터 데스크까지, 재무에서 시작해 주문에 이르기까지 모든 것을 다룬다.  
우리 소프트웨어에서 모든 것을 모델링하거나 모델링하지 않을 수 있지만, 도메인은 우리가 운영하는 것이다.  
에릭 에반스가 언급한 경계 콘텍스트처럼 보이는 도메인의 일부를 생각해보자.  

`은닉 모델`  
뮤직 코프의 경우 재무부서와 창고를 2개의 독립된 경계 콘텍스트로 간주할 수 있다.  
둘 다 외부 세계에 대한 명시적인 인터페이스 (예: 재고 보고서, 급여 명세서 등)가 있으며, 자신들만 알아야 하는 세부 사항(예: 지게차, 계산기)이 있다.  

재무부서는 창고의 자세한 내부 동작을 몰라도 되지만, 몇 가지 사항은 알고 있어야 한다.  
예를 들어 계정을 최신 상태로 유지하려면 재고 수준을 알 필요가 있다.  

집품자, 재고 위치를 나타내는 선반 등과 같은 개념을 볼 수 있다.  
마찬가지로 총 계정 원장의 항목은 재무에 필수적이지만 외부에 공유되지는 않는다.  

하지만 기업 가치를 계산하려면 재무 직원이 보유한 재고에 대한 정보가 필요하다.  
따라서 재고 품목은 두 콘텍스트 간 공유 모델이 된다.  

그러나 창고 콘텍스트에서 재고 품목에 대한 모든 것을 맹목적으로 노출할 필요는 없다.  
창고 콘텍스트에 내부의 재고 품목에 선반 위치에 대한 참조가 포함되지만, 공유된 재고 표현에는 수량만 포함되고 있는 것을 볼 수 있다.  
따라서 내부 전용 표현과 노출용 외부 표현이 존재한다.  
종종 내부 표현과 외부 표현이 서로 다른 경우에는 혼선을 피하기 위해 이름을 다르게 지정하는 것이 좋다.  
이 상황에서 한 가지 대안은 공유할 재고 품목을 재고 수량으로 부르는 것이다.  

`공유 모델`  
둘 이상의 경계 콘텍스트에서 나타나는 개념도 존재할 수 있다.  
재고 품목이 두 곳에 모두 존재하는 것을 봤다면 무엇을 의미할까? 재고품목은 복제될까?  

이에 대해 생각해보려면, 개념적으로 재무와 창고 모두가 재고 품목에 대해 알아야만 한다.  
재무 부서는 회사 가치를 평가하기 위해 재고 가치를 알아야 하는 반면, 창고 부서에서는 발송할 주문을 포장하기 위해 창고에서 실물을 찾을 위치를 알 수 있도록 재고 품목에 대해 알아야 한다.  

이와같은 상황에서 재고 품목과 같은 공유 모델은 서로 다른 경계 콘텍스트에서 다른 의미를 가질 수 있어 다른 이름으로 불릴 수 있다.  
창고 부서에서는 재고 품목이라는 이름을 사용해도 되지만, 재무 부서에서는 자산이라고 부르는 것이 더 일반적일 수 있다.  

두 곳에서 모두 재고 품목에 대한 정보를 저장하지만 그 정보는 서로 다르다.  
재무 부서에서는 재고 품목의 가치에 대한 정보를 저장하고, 창고 부서는 해당 품목이 어느 곳에 있는지 찾을 수 있는 위치 정보를 저장한다.  

여전히 두 부서의 로컬 개념을 모두 해당 품목의 글로벌 개념에 연결해야 할 수 있으며, 이름이나 제공업체와 같은 해당 재고 품목에 대한 공통의 공유 정보를 조회해야 할 수도 있다.
  

## 2.4.4 애그리거트와 경계 콘텍스트를 마이크로서비스에 매핑

애그리거트와 경계 콘텍스트는 외부의 더 넓은 시스템과 상호작용하기 위해 잘 정의된 인터페이스로, 응집력의 단위를 제공한다.  
애그리거트는 우리 시스템에서 단일 도메인 개념에 중점을 둔 독자적인 상태 기계며, 관련된 애그리거트의 집합을 표현하는 경계 콘텍스트와 함께 더 넓은 세계에 대한 명시적 인터페이스를 사용한다.  

따라서 둘 다 서비스 경계로 잘 작동할 수 있다.  
처음에 언급했듯이 여러분은 작업하는 서비스의 수를 줄이길 원하며, 결과적으로는 전체 경계 콘텍스트를 포괄하는 서비스를 대상으로 삼아야 할 것이다.  
경험을 통해 서비스를 더 작은 서비스로 분해하기로 결정하게 될 때는 애그리거트 자체의 분리를 원치 않는다는 사실을 기억해야 한다.  
즉, 하나의 마이크로서비스는 하나 이상의 애그리거트를 관리할 수 있지만, 하나의 애그리거트가 둘 이상의 마이크로서비스에서 관리되는 상황은 원하지 않는다.  

`거북이 아래 거북이`  
처음에는 아마도 여러 개의 큰 경계 콘텍스트를 인지하겠지만, 이러한 경계 콘텍스트는 결과적으로 더 많은 경계 콘텍스트를 포함할 수 있다.  

예를들어 창고를 주문 이행, 재고 관리, 제품 수령과 연관된 기능으로 분해하는 방식이 그렇다.  
마이크로서비스의 경계를 고려할 때, 더 크고 대분화된 콘텍스트의 관점에서 먼저 생각해본 후 내부 콘텍스트의 이음새를 분리하는 장점을 찾는다면 내부 콘텍스트에 따라 세분화한다.  

심지어 나중에 전체 경계 콘텍스트를 모델링하는 서비스를 더 작은 서비스로 분해하기로 결정했더라도, 소비자에게 보다 큰 단위의 API를 제공하는 방식으로 이 결정을 여전히 외부 세계에 숨길 수 있는 기법이 존재한다.  

서비스를 더 작은 부분으로 분해하는 결정은 분명 구현 결정이므로, 가능하다면 숨기는 것이 좋을 것이다.  

예를 들어보면 창고를 재고와 배송으로 분리했다.  
외부 세계에서 볼때는 여전히 창고 마이크로서비스만 존재한다.  
내부적으로는 재고 서비스가 재고 품목을 관리하고 배송 서비스가 발송을 관리할 수 있도록 더 분해됐다.  

하나의 마이크로서비스 안에서 하나의 애그리거트의 소유권을 유지하길 원한다는 점을 기억하라.  

이는 정보 은닉의 한 형태다.  
즉, 이후 이러한 구현 세부사항이 다시 변경되더라도 소비자가 알 수 없게 함으로써 내부 구현에 관한 결정을 숨겼다.  

테스트를 단순화하기 위해 아키텍처를 분리할 수 있다는 것은 내포 방식을 선호하는 또 다른 이유다.  
예를 들어 창고를 소비하는 서비스를 테스트할 때 창고 콘텍스트 내부의 각 서비스를 스텁할 필요가 없고, 대신 더 큰 단위의 API만 있으면 된다.  
또한 이것은 더 큰 범위의 테스트를 시작해야 하는 엔드투엔드 테스트를 하기로 결정한다면 협업하는 다른 서비스 모두를 스텁할 수도 있다.  


## 2.4.5 이벤트 스토밍

106p




