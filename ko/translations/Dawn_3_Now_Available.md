# EOSIO Dawn 3.0 출시

블록원은 EOSIO의 첫 번째 [기능 완료 프리-릴리즈인 Dawn 3.0 출시](https://github.com/EOSIO/eos/releases/tag/dawn-v3.0.0)를 알려드리게 되어 기쁩니다. 이번 프리-릴리즈는 2018년 6월 출시 예정된 EOSIO 1.0을 향한 주요 마일스톤입니다. 전 세계 곳곳에서 일하고 있는 우리 팀 개발자들은 EOSIO를 블록체인 어플리케이션을 구현하는 가장 강력한 플랫폼으로 만들기 위해 24시간 내내 일해오고 있습니다. EOSIO Dawn 2.0을 출시한 지 4개월이 지났고 우리는 보여드릴 게 많습니다.

최고의 블록체인 아키텍처를 구현하는 작업은 우리가 배우는 만큼 디자인이 변경되는 과정입니다. Dawn 3.0에 구현한 많은 기능들은 EOSIO 백서 첫 버전에 고려조차 하지 않았지만, 플랫폼을 기대한 성능으로 끌어올리고 유연하고 개발하기 쉽게 만드는 과정을 거치면서 이런 기능들을 발견했습니다.

## 확장 기능 - Scalability Features

확장성은 시장 요구에 맞도록 확장할 수 있는 능력입니다. 개발의 모든 단계에서 우리 팀은 앞으로 필요할 확장성을 염두에 두고 설계에 녹여왔습니다. 그렇기는 하지만, Dawn 3.0은 EOSIO의 확장성을 확보할 잠재적인 최적화의 일부만 구현했습니다. 향후 구현 과정에서 하드 포킹 없이 처리량을 높일 수 있는 병렬 처리(parallel computation)를 이용할 수 있도록 EOSIO를 설계해왔습니다.

### 블록체인간 통신 - Inter-blockchain Communication
블록체인간 통신은 업계에서 side-chain, plasma, sharding 등의 제안을 통해 모색해온 성배와도 같은 결정적인 확장 기능입니다. 이를 이용해 어느 블록체인이 입증된 안전한 방법으로 다른 블록체인에서 일어난 이벤트의 진위 여부를 검증할 수 있습니다. 목표는 단일 블록체인 범위에서 일어나는 스마트 컨트랙트 사이의 내부 통신만큼 블록체인간 통신을 안전하게 구현하는 것인데, 우리가 보기엔 그 목표에 도달했습니다.

우리의 관점에서 블록체인간 통신은 스마트 컨트랙트로 라이트 클라이언트를 구현할 수 있느냐 하는 문제에 지나지 않습니다. 라이트 클라이언트는 블록체인 전체를 검증하지 않고 블록체인의 트랜잭션을 검증할 수 있습니다. 즉, 지분 증명 블록체인을 효율적이고 안전한 라이트 클라이언트 검증으로 구현하면 됩니다. 이런 고려 없이 구현해놓은 뒤에 라이트 클라이언트 검증을 구현하기는 불가능하기 때문에, 라이트 클라이언트 검증을 프로토콜 디자인에 녹여야 합니다.

### 꼭 필요한 헤더만 검증 - Sparse Header Verification
종래의 라이트 클라이언트는 모든 블록 헤더를 처리하고 각 블록 헤더에 대응하는 증거를 검증해야 합니다. EOSIO는 1초에 블록을 2개씩 만들기 때문에, 블록체인은 블록 헤더를 처리하기 위해 최소한 초당 처리량 2건을 만족해야 합니다. 이런 경우라면 블록체인간 통신이 상대적으로 자주 일어나지 않는 시나리오에서는 확장성이 떨어집니다. 이 문제를 풀기 위해 비잔틴 장애에 내성이 있고 꼭 필요한 헤더만 검증(sparse-header validation)하는 최초의 블록체인을 구현했습니다. 구체적으로 살펴보면, 라이트 클라이언트를 속이기 위해서 2/3 (21개 중 15개) 이상의 블록 프로듀서가 배반해야 합니다. 거기에, 활동하는 블록 프로듀서 목록에 변경이 있고 관련된 블록체인간 메시지를 포함하는 블록 헤더만 처리합니다. 이렇게 해서 비잔틴 장애 허용 라이트 클라이언트를 유지하는 오버헤드를 크게 낮추고 블록체인간 통신 효율을 크게 높힐 수 있습니다.

### 컨텍스트-프리 액션 - Context Free Actions
컨텍스트-프리 액션은 효율적인 블록체인간 통신을 가능하게 하는 핵심 기능입니다. 블록체인 상태에 의존하지 않지만 트랜잭션에 포함되는 특별한 액션인데, 왜냐하면 "컨텍스트에서 자유”롭기 때문입니다. 컨텍스트-프리 액션의 예를 들면 서명의 머클 증명을 검증하는 작업이 있습니다. 이런 종류의 연산은 상태를 확인하지 않아도 되기 때문에, 병렬 처리로 쉽게 검증할 수 있고 리플레이 과정에서 제외할 수 있습니다.

모든 컨텍스트-프리 액션은 트랜잭션 내용 중 특별한 생략 가능 데이터 섹션을 참조할 수 있습니다. 즉, 커다란 머클 증명을 생략할 수 있어서, 블록체인 리플레이 과정에서 필요한 값비싼 연산 작업을 건너뛸 수 있습니다.

이러한 컨텍스트-프리 액션을 이용해서 블록체인간 통신의 오버헤드 대부분을 병렬로 처리할 수 있습니다. 기밀 거래(confidential transaction), 집합 증명(bulletproof), zkSNARK와 같은 많은 계산량이 필요한 정보 비공개 기법을 병렬 처리하고 오버헤드를 제거할 수 있습니다. 컨텍스트-프리 액션 사용을 장려하기 위해, 사용자가 일반적인 트랜잭션의 일부가 아니라 컨텍스트-프리 액션의 일부로 연산을 사용하면 블록 프로듀서는 CPU 사용량의 일부만 과금할 것입니다.

### 이벤트로 사용하는 컨텍스트-프리 인라인 액션 - Context-Free Inline Actions as Events
EOS Dawn 2.0 개발자들이 기다린 기능 중 하나는 외부에서 처리된 이벤트를 효율적으로 생성하는 방법입니다. 이더리움에서는 이런 종류의 이벤트로 컨트랙트의 내부 동작에 대한 구조화된 정보를 리포트하는 데 사용합니다. 컨텍스트-프리 액션을 구현했기 때문에, 잠재적으로 컨텍스트-프리 인라인 액션을 사용할 수 있게 되었습니다. 인라인 액션은 컨트랙트 코드를 통해 생성되고 현재의 트랜잭션의 일부로 실행되는 액션입니다. 컨텍스트-프리 인라인 액션은 처리 비용이 낮고 병렬로 처리할 수 있습니다. 모든 인라인 액션이 머클 증명 루트에 포함되기 때문에, 외부 서비스와 다른 블록체인에 증명 가능한 알림을 보내는 용도로 이 액션을 사용할 수 있습니다.

### 트랜잭션 압축 - Transaction Compression
압축 가능한 데이터가 많은 트랜잭션의 종류가 많습니다. 그 가운데 절대 피할 수 없는 예가 컨트랙트 웹어셈블리 코드입니다. 다른 예로는 ABI 명세와 계정/컨트랙트가 참조하는 리카도 컨트랙트(Ricardian contract)입니다. 소셜 미디어와 같은 일부 어플리케이션은 블록체인에 압축 가능한 유저 콘텐츠를 올리고 싶어 할 수도 있습니다.

트랜잭션 압축을 이용해서 블록체인은 많은 수의 트랜잭션을 효율적으로 저장하고 전송하며 압축이 가능하지 않은 데이터를 담은 트랜잭션 대신 압축 가능한 데이터를 담은 트랜잭션을 사용하는 사용자에게 덜 과금할 수 있습니다.

### Interpreter & Just-In-Time Compilation
Dawn 2.0에서 크게 바뀐 부분 중 하나는 웹어셈블리 런타임의 추상화 구현입니다. Dawn 3.0은 더 빠른 웹어셈블리 [JIT 컴파일러](https://github.com/WebAssembly/wasm-jit-prototype) 대신 [Binaryen](https://github.com/WebAssembly/binaryen) 웹어셈블리 인터프리터를 기본으로 사용합니다. 성능은 떨어지지만, 필요할 때 더 나은 성능의 JIT 환경으로 쉽게 바꿀 수 있게 만든, 안정성과 표준성을 높인 결정이었습니다. 인터프리터는 Dawn 2.0이 직면한 가장 큰 도전 과제인 컨트랙트 컴파일으로 발생하는 지연을 해결했습니다.  앞으로 백그라운드로 컨트랙트를 컴파일하고 최적화하는 동안에 인터프리터를 이용해서 새로 배포된 컨트랙트를 느리지만 대기 시간을 짧도록 실행할 수 있습니다. 이런 이중 구현은 모든 유닛 테스트로 컴파일된 코드와 인터프리터 코드 모두를 테스트한다는 사실을 의미합니다.  그래서 인터프리터와 JIT 컴파일러를 동시에 사용하는 하이브리드 방식을 도입하기 전에 잠재적인 비결정적이거나 비표준적인 동작을 감지할 수 있습니다.

### 자원 계량 사용 제한 - Resource Metering Rate Limiting
Dawn 3.0에서 완전히 새로운 자원 사용 제한 시스템을 갖게 되었습니다. 아마 가장 큰 변화는 객관적인 인스트럭션 개수 계산(objective instruction-counting) 알고리즘의 도입입니다. EOSIO 개발에 착수한 초반에는, 자원 사용에 대해 완전히 주관적인 사용 제한과 강제(subjective rate limiting and enforcement)가 가능한 방식을 운용하려는 목표를 가졌습니다. 우리가 발견한 사실은 주관적 강제(subjective enforcement)의 비용이 객관적 강제(objective approach)와 거의 동일하다는 점입니다. 우리는 이제 하이브리드 방식을 사용합니다. 사용자는 객관적으로 측정된 사용량에 대해 과금하고, 블록 프로듀서는 컨트랙트에 주관적인 실행 소요 시간 제한을 겁니다. 이와 같은 주관적인 제한이 객관적 측정 결과 편차를 노린 남용을 방지합니다.

이런 결정을 내린 이유 중 하나는, 각각의 트랜잭션이 과거에 가능했던 연산보다 더 많은 연산을 처리할 수 있기 때문입니다. 과거의 모델에서 1 ms 이하에 실행해야 했던 모든 트랜잭션에 비해, 이제 실행에 100 ms가 소요되는 단일 트랜잭션을 블록에 포함시키는 게 이론적으로 가능합니다.

자원 사용 제한에 포함된 또 다른 변화는, 토큰 정의가 필요할 때 자원 제한을 따로 설정하도록 분리했다는 점입니다. 이를 통해 EOSIO를 이용해서 토큰 사용이 전혀 필요하지 않는 프라이빗하고 접근승인이 필요한 블록체인을 만들 수 있습니다. 퍼블릭 블록체인은 지분 참여(staking)라는 방법을 통해 제한을 구현한 시스템 컨트랙트를 채택할 수 있고, 커뮤니티는 실제 할당이 이루어지는 방식과 별개로 자원 할당 방식을 동적으로 업그레이드할 수 있습니다.

### 500ms Block Interval & BFT DPOS
Dawn 3.0부터 블록 생성 주기를 3초에서 0.5초로 변경했습니다. 이로써 블록 확정까지 걸리는 시간을 크게 줄였습니다. BFT DPOS와 함께 사용하면 트랜잭션은 1초 이하로 변경 불가 확정 상태에 도달할 수 있습니다. 변경 불가 상태까지 걸리는 시간은 블록체인간 통신에서 중요한 함의를 가지는데, 외부 블록체인에 기록된 증명을 가져와서 이용하려면 변경 불가 확정 상태에 도달한 후여야 하기 때문입니다. 두 개의 EOSIO 기반 블록체인은 3초 이내에 왕복(round-trip) 통신이 가능해야 합니다. 비슷한 패턴의 통신이 이더리움에서는 9분, 비트코인에서는 3시간 이상 소요됩니다. 

BFT DPOS는 아직 구현되지 않았는데, 하드 포크하지 않아도 가능한 최적화 구현이라서 그렇습니다. EOSIO 1.0을 출시하기 전에 BFT DPOS를 구현할 예정입니다.

### BIOS Architecture
Dawn 2.0에서 가장 많이 바뀐 부분이 BIOS 아키텍처입니다. Dawn 3.0에서는 블록체인 비즈니스 로직의 대부분을 하드 포크 없이 커뮤니티가 동적으로 업데이트할 수 있는 스마트 컨트랙트로 옮겨서 구현했습니다. 기본 EOSIO으로 생성한 블록체인은 단일 프로듀서로 동작하고 토큰, 투표, 위임 지분 증명 기능이 없습니다. 코어 블록체인 코드에 구현된 유일한 기능은 계정을 생성하고 컨트랙트를 배포하고 자원 제한 한도를 걸 수 있는 승인(permission) 시스템입니다. 블록체인을 위임 지분 증명 시스템으로 만들어주는 토큰, 투표, 지분 참여(stake), 자원 할당을 포함하는 모든 기능은 이제 웹어셈블리 기반의 시스템 컨트랙트로 구현됩니다.

이와 같은 새로운 아키텍처 아래 우리 팀은 블록체인의 정적인 비-웹어셈블리 부분을 구현하는 데 집중할 수 있었습니다. 이 부분은 안정성을 위해 가장 중요한 부분이고 업그레이드하기 가장 어려운 부분입니다.  EOSIO Dawn 3.0과 EOSIO 1.0 출시 사이에 시스템 컨트랙트, 지분 참여, 투표 기능의 최종적인 세부 구현을 완성할 것입니다.

## 보안 기능 - Security Features
보안은 모든 컴퓨팅 시스템에서 중대한 주제이며, 우리는 EOSIO를 시장에 나온 가장 보안성 높은 블록체인이 되도록 설계해왔습니다. 보안은 해킹, 하드웨어 고장, 하드웨어 유실, 패스워드 유실의 리스크를 감안해야 하는 다차원 문제입니다. 하드웨어 지갑은 해킹을 방어하는 데 좋지만, 고장났을 경우 계정에 접근할 수 없게 됩니다. 하드웨어 지갑의 내용을 종이에 옮겨서 백업으로 삼아도 잃어버릴 수 있고 도난당할 수 있습니다.

### 보안 지연 트랜잭션 - Security Delayed Transactions
Dawn 3.0의 가장 중요한 기능 중 하나는 액션마다 사용자가 지연 시간을 설정할 수 있는 기능입니다. 지연 시간이 설정되면, 트랜잭션은 그 내용이 적용되기 전 몇 시간이나 며칠 동안 블록체인에 브로드캐스트되어야 합니다. 지연 기간 동안 사용자는 상위 권한으로 계정을 리셋할 수 있는 조치를 강구할 수 있고, 트랜잭션을 취소할 수 있습니다. 이 기능은 조치를 취하기엔 너무 늦어버릴 때까지 해킹당했다는 사실을 모르는 다른 블록체인에 비하면 중요한 개선입니다.

### 패스워드 복구 - Lost Password Recovery
모든 계정은 "owner"와 "active"라는 적어도 두 가지 권한(permission)을 가집니다. owner 권한은 멀티시그를 구성하는 M개의 비밀키 중 N개의 비밀키가 되어야 하는데, N개의 키 중 owner key를 포함하지 않는 키는 없어야 합니다. owner 권한은 active 키를 잃어버렸거나 도난당했을 때 active 권한을 리셋할 수 있습니다.

owner 키를 잃어버렸거나 멀티시그의 일부를 구성하는 파트너가 비협조적이라면, 해당 계정의 owner 권한이 30일 동안의 활동이 없었음을 확인한 후 해당 계정의 active 권한으로 owner 권한 리셋을 요청할 수 있습니다. owner 권한은 active 권한을 업데이트하는 방식으로 owner 권한 리셋 요청이 온지 7일 내에 대항할 수 있습니다.

이 모델을 이용하면 하나 이상의 하드웨어 지갑으로 관리하는 owner 권한은 해킹과 장비 고장이 발생해도 안전할 것입니다. 만약 장비가 하드웨어가 달린 애플 아이폰이고 지문/안면 인식으로 안전하게 비밀키를 보관한다면, 공격자는 멀티시그 파트너를 뚫고 물리적으로 휴대폰을 훔치고 지문이나 얼굴을 훔쳐야 합니다. 이상적으로는 멀티시그 파트너도 생체인식을 이용한 보안 하드웨어 장치를 사용하고 있을 것입니다.

### 거래 제안 시스템 - Transaction Proposal System
멀티시그 사용이 훨씬 쉬워졌습니다. 일반적인 트랜잭션이 요구하는 만료 제한 시간 동안 모든 서명을 모으지 않고, 각 사용자가 편한 시간에 독립적으로 자신의 권한을 추가하거나 제외할 수 있습니다. 거래 제안 시스템으로 누구나 트랜잭션을 제안할 수 있고, 트랜잭션과 연관된 당사자들이 단순히 승인하면 됩니다. 자신의 승인을 비롯한 정족수를 모으는 시간 동안, 자신의 승인을 철회할 수도 있습니다.

이런 시스템을 구현하려고 컨트랙트가 계정 권한의 집합이 특정 트랜잭션을 허용하기 충분한지를 평가할 수 있는 새로운 API를 추가했습니다. 이를 통해 하드 포크 없이 새로운 웹어셈블리를 배포해서 멀티시그 프로세스를 업그레이드할 수 있습니다.

## 컨트랙트 개발 단순화 - Simplified Contract Development
EOSIO의 주요 목표 중 하나는 가능한한 컨트랙트 개발을 단순하고 괴롭지 않게 만드는 것입니다. C++로 메소드와 클래스를 구현할 줄 아는 개발자라면 boilerplate 약간으로 스마트 컨트랙트를 구현할 수 있어야 합니다.

단순한 몇 줄로 "hello world" 컨트랙트를 단순화할 수 있어서 기쁩니다. 컨트랙트 ABI 생성과 클래스에 구현한 메소드로 사용자 액션을 디스패치하는 과정을 툴체인으로 자동화했습니다. 지금까지 컨트랙트 개발은 이 정도로 쉽지 않았습니다.

```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>
using namespace eosio;

struct hello : public contract {
    using contract::contract;
 
    void hi( name user ) {
        print( “Hello, “, user );
    }
};

EOSIO_ABI( hello, (hi) )
```

### 부동소수점 지원 - Floating Point Support
스마트 컨트랙트 개발을 단순화하는 데는 개발자들이 필요한 수학 알고리즘을 구현하기 편하게 해주는 작업이 있었습니다. 블록체인 개발의 가장 어려운 측면 중 하나는 부동소수점 연산과 제곱, 제곱근, 삼각 함수가 부족했다는 점입니다. 모든 계산을 에러나기 쉽고 메모리를 많이 사용하는 고정소수점 대신 부동소수점을 사용하면 Bancor 같은 알고리즘 구현이 무척 쉬워집니다.

하드웨어 부동소수점의 비결정론적인 특성을 보완하기 위해 웹어셈블리 컨트랙트에서 투명하게 사용하고 있는 소프트웨어로 구현한 부동소수점 라이브러리를 통합했습니다. 부동소수점을 소프트웨어로 구현해서 복잡한 계산의 경우 고정소수점을 사용하는 경우에 비해 너무 크지 않은 비용으로 결정론적으로 작동하는 혜택과 개발의 편의를 얻을 수 있습니다. 많은 경우, 고정소수점은 결정론적 부동소수점에 비해 오류가 발생하기 쉽거나 메모리 사용이 더 많습니다.

### C++ Standard Template Library Support
EOSIO Dawn 3.0 출시를 위해 C++ 표준 템플릿 라이브러리의 대부분을 지원할 수 있도록 대단히 노력했습니다. 지원이 되면 개발자는 익숙한 툴, 라이브러리, 알고리즘을 사용할 수 있고, 그와 동시에 이런 알고리즘을 표준에 맞지 않게 구현한 코드가 일으킬 수 있는 잠재적인 버그를 제거할 수 있습니다.

### 예약 트랜잭션 - Scheduled Transactions
예약 트랜잭션(scheduled transaction)을 이용해서, 컨트랙트 실행에 충분한 대역폭을 지분 참여(stake)해두었다면, 개발자는 영원히 동작하는 컨트랙트를 작성할 수 있습니다. 다른 플랫폼은 적절한 시점에 컨트랙트를 구동시키기 위해 체인 바깥에 장치를 마련해야만 합니다. 예약 트랜잭션을 사용하면, 개발자가 컨트랙트를 계속 실행시키기 위해 자체 서버를 운영하지 않아도 되기 때문에 효율적이고 이용하기도 쉬워집니다.

### Automatic Scope Detection
EOSIO Dawn 2.0의 모든 트랜잭션은 접근할 데이터 범위를 꼭 선언해야 했습니다. 개발자가 구현할 때 실수하기 쉬웠고 코드가 길어지는 편이었습니다. Dawn 3.0에서는 블록 프로듀서가 데이터 접근 범위를 결정하는 책임을 지고, 충돌이 있을 경우 해결합니다. 그래서 모든 트랜잭션이 더 작아졌으며 사용자, 개발자, 풀 노드가 감당했던 스케줄링 오버헤드를 블록 프로듀서가 책임지게 되었습니다.

### MultiIndex Database API
EOSIO Dawn 3.0은 boost::multi_index_container를 본따서 만든 새로운 데이터베이스 API를 구현했습니다. 이 API를 이용해서 여러 개의 키로 정렬된 데이터베이스 테이블을 다루고, 원하는 값을 찾고, 검색시 하한/상한을 이용하고, 데이터베이스에서 앞으로 뒤로 iterate(반복)하는 작업이 수월해졌습니다. 새로운 API는 테이블을 스캔하는 성능을 극적으로 향상시켜주는 이터레이터 인터페이스를 사용합니다.

게다가 64비트 부동소수점 뿐만 아니라 64비트, 128비트, 256비트, 512비트 정수형도 인덱스로 사용할 수 있습니다.  EOSIO 1.0 출시 전에 문자열 인덱스를 사용하는 기능을 구현할 것입니다. 테이블 하나에 인덱스한 필드를 무한히 가질 수 있기 때문에 유연성과 개발 편의의 측면에서 중대한 개선입니다.

## 성능과 초당 처리량 결과 - Performance & TPS results
실제 성능은 우리 팀이 면밀히 모니터해온 중요한 주제고, 이번에 내놓는 결과에 매우 기쁩니다. 앞으로 최적화를 구현함에 따른 성능의 하한과 상한을 이해하기 위해 몇 가지 다른 설정으로 소프트웨어의 성능을 벤치마크해왔습니다. 모든 테스트는 연산 복잡성의 측면에서 EOSIO를 이용한 토큰 이체가 비트코인 이체나 이더리움 ERC20 토큰 이체와 공평한 조건에서의 비교라고 가정합니다.

### 비교 기준 수치 - Worst Case - 1000 TPS
아무런 최적화 없는 비교 기준(baseline) 성능입니다. 싱글 쓰레드로 서명 검증을 하는 인터프리터로 동작하는 멀티 노드 네트워크를 사용할 때 1000 TPS를 유지할 수 있습니다.

### 평균 수치 - Average Case - 3000 TPS
JIT 컴파일러를 활성화시키면, 싱글 쓰레드로 서명 검증을 하는 인터프리터로 동작하는 멀티 노드 네트워크로 3000 TPS를 유지할 수 있습니다.

### 최대 수치 - Best Case - 6000 TPS
병렬 서명 검증을 구현하면, 병렬 처리 수준과 서명의 수가 증가함에 따라 서명당 처리 소요 시간이 0에 가까워질 것이라고 가정할 수 있습니다. 서명 검증을 건너뛰는 방법으로 이런 시나리오를 시뮬레이션할 수 있습니다. 이 모델에서 JIT 컴파일러를 사용한 멀티 노드 네트워크로 6000 TPS까지 도달할 수 있습니다.

### 이론상 수치 - Theoretical Case - 8000 TPS
시뮬레이션 시나리오에서 P2P 네트워크 코드 동작을 빼고, 서명 검증을 끈 상태에서 JIT를 사용하는 CPU가 할 수 있는 작업에만 초점을 맞춰봅시다. 그러면 싱글 쓰레드로 8000 TPS까지 도달할 수 있습니다.  단일 체인에서 이 수치 이상의 성능을 얻기 위헤서는 웹어셈블리의 병렬 실행과 현재 구현보다 더 개선된 스케줄러를 구현해야 합니다. 동일한 시나리오에서 인터프리터 대신 JIT을 사용하면 2700 TPS에 도달할 수 있습니다. 이 결과를 보면 JIT를 활성화하는 상대적으로 단순한 변경만으로 토큰 이체 성능을 3배 높힐 수 있습니다. 모든 테스트 결과는 맥북 2.8GHz i7에서 실행한 결과입니다.

### 제한 없는 처리량 - Unlimited Transactions Per Second
보통의 "초당 트랜잭션 수" 정의로는 서로 비교가 안 되는 성질을 가진 둘을 비교하게 하는 상황을 자주 만듭니다. 블록체인간 통신을 사용하면 원하는 수의 블록체인으로 작업량을 분산시킬 수 있습니다. 서로 다른 블록체인 사이에 토큰을 확실하고 안전하게 이체할 수 있습니다. 같은 (또는 서로 다른) 블록 프로듀서들이 동시에 운영하는 1000개의 블록체인이 있다면 초당 수백만 건의 트랜잭션을 볼 수 있을 것입니다. 이 모습은 다른 블록체인에서 제시한 이론적인 확장 계획이 실제로 현실화된다는 점을 보여줍니다.

EOSIO 소프트웨어로 런칭된 서로 다른 블록체인 퍼블릭 네트워크를 운영하는 블록 프로듀서들이 사용자 수요를 만족시킬 수 있을 만큼 많은 블록체인을 운영하기를 강력히 권합니다. 모든 블록체인이 지분 참여와 자원 할당을 하는데 동일한 토큰을 기초 단위로 사용할 수 있을 것입니다. 이렇게 해서 단일 토큰으로 최대한 가능한 네트워크 효과를 일으키고 높은 시장가치를 가진 토큰들이 만들어 온 신뢰와 안정성이라는 경제적인 인센티브를 누릴 수 있을 것입니다.

거래소, 환전소, 소셜 미디어 등의 서비스는 동시에 작동하는 여러 블록체인을 이용해서 연산을 쉽게 분산시킬 수 있습니다.

## 앞으로 나아갈 길 - The Road Ahead
EOSIO Dawn 3.0 출시를 진행하면서 코어 플랫폼 기능의 안정성을 높히는 데 중점을 두었습니다. 다음 달에는 지분 참여, 투표, 거버넌스 기능을 모두 구현할 마지막 시스템 컨트랙트를 준비할 것입니다. 또한 토큰 표준을 완성할 것입니다.

이 시스템 컨트랙트가 마음에 들 정도로 충분히 다듬어지면 새로운 퍼블릭 테스트넷을 오픈할 예정입니다. 그때까지 사용할 수 있도록, 자체 테스트 네트워크를 구동시키고 어플리케이션를 구현하는 과정을 굉장히 단순화했습니다. 개발자들의 혼동을 최소화하기 위해 새로운 테스트넷을 준비하는 동안 현재의 퍼블릭 테스트넷을 몇 주 동안 꺼둡니다.

## 요약 - Summary
EOSIO Dawn 3.0은 API를 안정화하고 "기능 구현 완료" 상태로 준비한 개발자 릴리즈입니다. 현재의 플랫폼이 어플리케이션를 실제로 개발하려는 개발자들이 어플리케이션 개발을 시작해도 될 만큼 충분히 안정적이라고 믿습니다. **우리가 1년 전에 상상해왔던 것 이상으로 EOSIO는 훨씬 더 강력해지고 개발하기 쉬워졌습니다.**

우리 팀은 점점 커지고 있고, 개발은 기록적인 속도로 진행되고 있습니다. 우리 저장소는 지난달 깃헙에 있는 모든 C++ 저장소 중 가장 활발한 저장소 가운데 하나였습니다. 모든 작업이 6월로 예정된 EOSIO 1.0의 고품질 공개 출시 일정에 맞게 순로조이 진행되고 있습니다!

## 번역 정보

* https://medium.com/eosio/eosio-dawn-3-0-now-available-49a3b99242d7 (2018.04.06 KST)
