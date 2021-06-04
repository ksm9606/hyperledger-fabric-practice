# hyperledger-fabric-practice
하이퍼레저 패브릭 튜토리얼 공부


# 하이퍼레저 패브릭이란?
하이퍼레저 패브릭은 허가형 프라이빗 블록체인(Permissioned Private Blockchain)의 형태를 가진다. 누구나 자유롭게 참여가 가능한 기존의 퍼블릭 블록체인과 달리, 하이퍼레저 패브릭에서는 인증 관리 시스템에 의해 허가된 사용자만이 블록체인 네트워크에 참여할 수 있다. 따라서 패브릭 네트워크에 참여한 노드들은 이미 시스템에 의해 허가된, 신뢰를 가진 노드로 볼 수 있고 퍼블릭 블록체인에서 사용하는 악의적인 노드를 검증하기 위한 복잡한 합의 알고리즘 등을 필요로 하지 않는다. 단지 원장에 접근하려는 사용자가 허가된 노드인가, 그러한 권한이 있는가, 트랜잭션이 제대로 구성되어 있는가를 검증하는 정도로 충분하다. 물론 필요에 의해 원하는 합의 알고리즘을 네트워크 내에서 선택적으로 사용할 수도 있다.
 패브릭에서는 모든 노드가 동일한 원장으로 정보를 공유할 수 있고, 비즈니스 목적에 맞게 공유하고자 하는 노드 간에만 별도의 원장을 생성하는 것도 가능하다. 기존의 블록체인 네트워크에서는 참여한 모든 노드에게 원장에 기록되어 있는 정보가 공유되었다. 하지만 기업 입장에서 비즈니스를 하다 보면 모두에게 공유하고 싶지 않은 민감한 정보들이 생기기 마련이다. 하이퍼레저 패브릭은 네트워크 내에서 목적에 맞는 별도의 원장을 생성할 수 있는 채널(Channel)을 제공함으로써 기업이 사용하기 용이하도록 고안되었다.


# 하이퍼레저 패브릭 구성요소
(Hyperledger Fabric Component)

하이퍼레저 패브릭의 특징을 구현하기 위한 구성요소에는 크게 분산원장, 체인코드, peer, orderer가 있다.

 블록체인 기술의 핵심인 분산원장(Distributed Ledger)이 있다. 공유하고자 하는 데이터의 변화를 모두 기록해둔 것이 원장이다. 원장은 현재의 상태를 저장해 놓은 데이터베이스인 월드 스테이트(World State)와 상태변화에 대한 모든 로그 기록이 저장 되어있는 블록체인(Blockchain) 부분으로 나뉘어있다. 추가로 원장에 새로운 내용을 업데이트 하거나 기존의 내용을 읽어 오기 위해 필요한 것이 바로 체인코드(Chaincode)이다.

 이러한 원장과 체인코드를 관리하며 패브릭 네트워크를 구성하는 노드를 peer라 부른다. 패브릭 네트워크 참여자들은 peer에 설치되어 있는 체인코드 실행 요청을 통해 peer에 저장된 원장에 데이터를 읽거나 쓸 수 있다. 이러한 요청은 보통 사용자의 편의를 위해 체인코드와 함께 개발되는 dapp을 통해 이루어진다. 체인코드 실행을 요청하는 트랜잭션이 발생하면 3단계(execution - ordering - validation)의 과정을 거쳐 원장에 기록되고 사용자에게 결과를 반환한다. peer는 수행하는 역할에 따라 크게 다음과 같이 4가지로 구분된다.

Endorsing peer - 체인코드 시뮬레이션을 통해 트랜잭션이 적절한지 판단하는 역할을 한다. 3단계의 과정 중 execution에 해당한다.

Committing peer - 모든 peer가 수행하는 역할로, 최신 블록에 대한 검증을 한다. 위의 3단계의 과정 중 validation에 해당한다.

Anchor peer - 다른 조직과의 통신을 위해 다른 조직의 peer와 통신하는 역할을 한다.

Leader peer - orderer와 연결되어 최신 블록을 전달받아 조직 내 다른 peer들에게 전송하는 역할을 한다.  

Endorsing peer들이 시뮬레이션을 통해 적절하다고 판단한 트랜잭션들을 모아서 정렬한 후 실제 블록을 생성하는 노드를 orderer라 한다. 현재 하이퍼레저 패브릭에서 orderer가 트랜잭션의 순서를 정렬하는 방법에도 solo와 kafka방식으로 2가지가 존재한다. solo방식은 보통 테스트용으로 orderer 하나가 정렬 및 블록 생성의 모든 과정을 담당하는 방식이다. kafka방식은 분산 메시징 시스템인 kafka cluster를 통해 orderer가 트랜잭션을 정렬하고 블록을 생성하는 방식이다. orderer는 블록을 생성한 후 자신에게 연결되어 있는 leader peer들에게 블록을 전달하고, leader peer들이 다시 자신이 속한 채널의 peer들에게 블록을 전달하면 peer들은 블록을 검증한 후 자신의 원장에 추가시키게 된다. 패브릭에서는 체인코드 실행을 요청하는 트랜잭션부터 원장에 기록되는 과정을 통틀어 합의라고 부른다.

 위에서 패브릭은 허가된 사용자만이 참여할 수 있는 허가형 블록체인이라 하였다. 패브릭에서는 사용자의 권한 및 인증을 위해 MSP(Membership Service Provider)라는 인증 관리 시스템을 사용하는데, 여기에는 네트워크 내 노드의 역할과 권한 등이 정의되어 있다.

 이러한 MSP를 발급하고 관리하는 역할을 하는 기관을 CA(Certificate Authority)라고 한다. 사용자를 인증해 주는 것은 중요한 역할이기 때문에 CA는 보통 신뢰 있는 기관이 담당하는데, 하이퍼레저 패브릭에서는 Fabric-CA 노드가 그 역할을 수행한다.

 하이퍼레저 패브릭에서는 CA노드를 통해 1차적으로 사용자의 서명과 권한 등을 확인하고, peer를 통해 원장에 기록되기 전에 보증 정책(Endorsement Policy)을 준수하는지 확인하는 과정을 거친다. 보증 정책은 보통 해당 트랜잭션이 지정된 peer들의 허가를 받아야 한다는 내용인데, 원장을 공유하는 채널별로 참여자들은 다양한 방식으로 보증 정책을 설정할 수 있다.f
 
 

