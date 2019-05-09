# TLS Support
&nbsp;RabbitMQ는 TLS를 지원한다. TLS를 사용하여 클러스터의 노드 간 연결을 암호화할 수도 있다.
TLS는 클라이언트/서버 응용 프로그램이 네트워크로 통신을 하는 과정에서 도청, 간섭, 위조를 방지하기 위하여 설계되었다.
이 규약은 인터넷 같이 TCP/IP 네트워크를 사용하는 통신에 적용되며, 통신 과정에서 전송계층 종단간 보안과 데이터 무결성을 확보해준다. 인터넷을 사용한 통신에서 보안을 확보하려면 두 통신 당사자가 서로가 신뢰할 수 있는 자임을 확인할 수 있어야하며, 서로간의 통신 내용이 제 3자에 의해 도청되는 것을 방지해야 한다. 따라서 서로 자신을 신뢰할 수 있음을 알리기 위해 전자 서명이 포함된 인증서를 사용하며, 도청을 방지하기 위해 통신 내용을 암호화한다. 이러한 통신 규약을 묶어 정리한 것이 바로 TLS이다.
TLS에서 서버 및 클라이언트의 신원을 확인하는 데는 인증서가 사용되며, 인증서의 신뢰성은 인증서를 발급해 준 인증기관에 의해 결정된다.<br>
&nbsp;AMQP 0-9-1뿐만 아니라 RabbitMQ에서 지원하는 모든 프로토콜에 대해 TLS를 사용할 수 있다.
## RabbitMQ를 이용한 클라이언트 연결에 대한 TSL의 일반적인 접근법(Common Approaches to TLS for client Connections with RabbitMQ)
&nbsp;클라이언트 연결의 경우 두 가지 일반적인 접근 방식이 있다.
- TLS 연결을 처리하도록 RabbitMQ 구성
- 클라이언트 연결의 TLS terminating을 수행하기 위해 프록시나 load balancer를 사용하고 RabbitMQ 노드에 대한 plain TCP 연결을 이용한다.
## TLS 지원을 위한 Erlang/OTP 요구 사항<br>(Erlang/OTP Requirements for TLS Support)
&nbsp;TLS 연결을 지원하기 위해 RabbitMQ가 Erlang/OTP 설치에서 TLS 및 암호화 관련 모듈을 사용할 수 있어야 한다. TLS와 함께 사용할 Erlang/OTP 권장 버전은 가장 최근에 지원되는 Erlang이다. 이전 버전은 지원이 되더라도 일부 제한 사항이 있다.<br>
&nbsp;Erlang <u>asn1</u>, <u>crypto</u>, <u>public_key</u> 및 <u>ssl</u> 라이브러리(응용 프로그램)가 설치되어야 한다. Devian과 Ubuntu에서는 각각 erlang-asn1, erlang-crypto, erlang-public-key 및 erlang-ssl 패키지에 의해 제공된다.<br>
&nbsp;Erlang/OTP가 소스에서 컴파일되면, configure가 OpenSSL을 찾고 위의 라이브러리를 구축하도록 할 필요가 있다.<br><br>
- **TLS 기본사항 : 인증기관, 인증서, 키(TLS Basics: Certificate Authorities, Certificates, Keys)**<br>
&nbsp;TLS는 연결 트래픽을 암호화와 중간자 공격 중간자 공격(Man-in-the-Middle)에 대해 완화시킬 수 있는 인증(검증) 방식의 제공이라는 두 가지 주요 목적을 가지고 있다. 두 가지 모두 PKI(Public Key Infrastructure)로 알려진 일련의 역할, 정책 및 절차를 사용하여 수행된다.<br>
&nbsp;PKI는 암호학적으로 검증할 수 있는 digital identity의 개념을 기반으로 한다. 이러한 identity는 인증서 또는 인증서/키 쌍이라고 불린다. 모든 TLS 지원 서버에는 일반적으로 연결에서 보낸 트래픽을 암호화하는 데 사용되는 연결별 키를 계산하는 데 사용하는 자체 인증서/키 쌍이 있다. 그러나 클라이언트는 자체 인증서를 가지고 있을 수도 있고 가지고 있지 않을 수도 있다. RabbitMQ와 같은 메시징 및 도구와 관련하여 클라이언트가 인증서/키 쌍을 사용하여 서버에서 신원을 확인할 수 있도록 하는 것이 일반적이다.<br>
&nbsp;인증서/키 쌍은 OpenSSL과 같은 도구에 의해 생성되고 인증기관(CA)이라고 불리는 기관이 엔티티에 의해 서명된다. CA(Certificate Authorities)는 사용자가 사용하는 인증서를 발급한다. 인증서는 CA에 의해 서명될 때 신뢰 사슬(chain)을 형성한다. 이러한 체인에는 둘 이상의 CA가 포함될 수 있지만 궁극적으로 애플리케이션에서 사용된 인증서/키 쌍이 서명한다. CA 인증서의 체인은 일반적으로 단일 파일과 함께 배포되는데 이러한 파일을 CA 번들이라고 한다.<br>
&nbsp;다음은 하나의 루트 CA와 하나의 leaf(서버 또는 클라이언트)인증서가 있는 가장 기본적인 체인의 예이다.<br>
![image01](https://user-images.githubusercontent.com/45917477/57404010-ec8c2d00-7215-11e9-8f68-cc319437138b.png)<br>
&nbsp;중간 인증서가 있는 체인은 다음과 같다.<br>
![image02](https://user-images.githubusercontent.com/45917477/57404306-a1264e80-7216-11e9-93d0-fd0413813b37.png)<br>
&nbsp;TLS 지원 RabbitMQ 노드는 파일(CA 번들), 인증서(공개키) 파일 및 개인키 파일에서 신뢰할 수 있다고 간주되는 인증기관 인증서 집합을 가지고 있어야 한다. 파일들은 로컬 파일 시스템으로부터 읽을 수 있으며 RabbitMQ 노드 프로세스의 권한이 있는 사용자가 읽을 수 있어야 한다.<br><br>
- **CA, 인증서 및 키를 생성한는 짧은 경로(The Short Route to Generating a CA, Certificates, and Keys)**<br>
&nbsp;사용자가 CA certifica 번들 파일과 두 개의 인증서/키 쌍에 액세스할 수 있다고 가정했을 때, 인증서/키 쌍은 RabbitMQ 및 TLS-지원 포트에서 서버에 접속하는 클라이언트에 의해 사용된다. 인증기관과 두 개의 키 쌍을 생성하는 과정은 상당히 복잡하며 오류가 발생하기 쉽다. MacOS나 Linux에서 이러한 모든 것들을 쉽게 생성할 수 있는 방법은 tls-gen을 사용하는 것이다. <u>Python 3.5+</u>과 <u>PATH</u>의 <u>make</u>, <u>openssl</u>이 필요하다.<br>
&nbsp;<u>tls-gen</u>과 <u>tls-gen</u>이 생성하는 인증서/키 쌍은 자체 서명되며 개발 및 테스트 환경에만 적합하다는 점에 유의해야 한다. 대부분의 프로덕션 환경은 널리 신뢰받는 상용 CA에서 발급한 인증서와 키를 사용해야 한다.<br><br>
- **<u>tls-gen</u>의 기본 프로필 사용(Using tls-gen's Basic Profile)**<br>
&nbsp;다음은 CA를 생성하고 이를 사용하여 두 개의 인증서/키 쌍을 생성하는 예제이다. 하나는 서버용이고 다른 하나는 클라이언트용이다.<br>
![image03](https://user-images.githubusercontent.com/45917477/57404410-dc288200-7216-11e9-80de-d6f79c35e17d.png)<br>
&nbsp;기본 tls-gen profile로 생성된 인증서 체인은 다음과 같다.<br>
![image04](https://user-images.githubusercontent.com/45917477/57404438-eea2bb80-7216-11e9-916d-78208c083751.png)<br>

- **RabbitMQ에서 사용가능한 TLS 지원(Enabling TLS Support in RabbitMQ)**<br>
&nbsp;RabbitMQ에서 TLS 지원을 활성화하기 위해서는 노드는 인증기관 번들(하나 이상의 CA인증서가 있는 파일), 서버의 인증서 파일 및 서버 키의 위치를 알 수 있도록 구성되어야 한다. 또한 TLS Listener는 TLS-지원 클라이언트 연결에 대해 수신 대기할 포트가 무엇인지 알 수 있어야 한다.<br>
&nbsp;다음은 TLS와 관련된 필수 구성 설정이다.<br>

|Configuration|Description|
|---|---|
|listeners.ssl|TLS 연결을 수신 대기할 포트 목록이다. RabbitMQ는 단일 인터페이스 도는 다중 인터페이스에서 수신 대기한다.|
|ssl.options.cacertfile|인증기관(CA) 번들 파일 경로|
|ssl.options.certfile|서버 인증서 파일 경로|
|ssl_options.keyfile|서버 개인키 파일 경로|
|ssl_options.verify|피어 확인 사용 설정|
|ssl_options.fail_if_no_peer_cert|true로 설정했을 때, 클라이언트가 인증서를 제공하지 못하면 TLS 연결이 거부된다.|

- **인증서 및 개인키 파일 경로(Certificate and Private Key File Paths)**<br>
&nbsp;RabbitMQ는 구성된 CA 인증서 번들, 서버 인증서 및 개인키를 읽을 수 있어야 한다. 파일이 존재해야 하며 적절한 권한이 있어야 한다. 그렇지 않을 경우 노드는 TLS 사용 연결을 시작하지 못하거나 실패하게 된다.<br><br>
- **TLS가 활성화되어 있는지 확인하는 방법(How to Verify that TLS is Enabled)**<br>
&nbsp;노드에서 TLS가 활성화되어 있는지 확인하기 위해서는 노드를 다시 시작하고 로그 파일을 검사해야 한다. 다음과 같이 활성화된 TLS Listener에 대한 항목을 포함해야 한다.<br>
![image05](https://user-images.githubusercontent.com/45917477/57404508-0f6b1100-7217-11e9-84dd-12effede92cf.png)<br>
- **개인키 비밀번호 제공(Providing Private Key Password)**<br>
&nbsp;개인키는 선택적으로 암호로 보호할 수 있다. 암호를 제공하기 위해서는 password 옵션을 사용한다.<br>
![image06](https://user-images.githubusercontent.com/45917477/57404536-21e54a80-7217-11e9-9707-8db1e8ffb450.png)<br>
## TLS peer 확인(TLS Peer Verification)
- **피어 검증 작동 방식(How Peer Verification Works)**<br>
&nbsp;TLS 연결이 설정되면 클라이언트와 서버는 여러 단계를 거쳐 연결 협상을 수행한다. 첫 번째 단계는 피어가 선택적으로 인증서를 exchange한다. 인증서를 exchange한 후 피어는 선택적으로 자신의 CA 인증서와 제시된 인증서 간에 신뢰 체인을 설정하기 위해 시도할 수 있다. 이 프로세스는 피어 검증 또는 피어 유효성 검사로 알려져 있으며, 인증 경로 유효성 검사 알고리즘으로 알려진 알고리즘을 따른다.<br>
&nbsp;각 피어는 “리프(leaf)”(클라이언트 또는 서버) 인증으로 시작하고 적어도 1개의 인증기관(CA) 인증으로 계속되는 인증 체인을 제공한다. 여러 CA 인증서가 있는 경우 일반적으로 서명(signatures) 체인을 구성하며 이는 각 CA 인증서가 다음(next) 인증서에 의해 서명되었음을 의미한다. 예를 들어, 인증서 B가 A에 의해 서명되고 C가 B에 의해 서명되는 경우, 체인은 A, B, C로 구성된다.<br>
&nbsp;다음은 하나의 루트 CA와 하나의 리프(서버 또는 클라이언트) 인증서가 있는 가장 기본적인 체인의 예이다.<br>
&nbsp;중간 인증서가 있는 체인은 다음과 같다.<br>
&nbsp;피어 검증(확인) 중에 TLS 연결 클라이언트(또는 서버)는 피어가 제공하는 인증서의 체인을 통과하고 신뢰할 수 있는 인증서가 발견되었을 경우, 해당 피어를 신뢰할 수 있는 것으로 간주한다. 신뢰할 수 있는 유효한 인증서가 없으면 피어 검증에 실패하고 “Unknown CA” 또는 이와 유사한 오류(OpenSSL parlance의 “alert”)로 클라이언트 연결이 닫힌다. 경고 메시지는 서버에서 다음과 유사한 메시지와 함께 기록된다.<br>
&nbsp;모든 단계에서 인증 유효성도 검사된다. 만료되었거나 아직 유효하지 않은 인증서는 거부된다. 이 경우 TLS 경고는 다음과 같이 나타난다.<br><br>
- **신뢰할 수 있는 인증서(Trusted Certificates)**<br>
&nbsp;Erlang/OTP 및 RabbitMQ를 포함한 모든 TLS 지원 도구 및 TLS 구현은 인증서 집합을 신뢰할 수 있는 것으로 표시하는 방법을 제공한다. 리눅스와 다른 유닉스 계열 시스템에서 이것은 보통 superuser들에 의해 관리되는 디렉토리이다. 해당 디렉토리의 CA 인증서는 신뢰할 수 있는 것으로 간주되며, 인증서는 해당 인증서에 의해 발급된 인증서(클라이언트가 제공한 인증서 등)도 신뢰받는 것으로 간주된다. 신뢰할 수 있는 인증서 디렉토리의 위치는 배포판, 운영체제마다 다르다.<br>
&nbsp;Windows에서 신뢰할 수 있는 인증서는 certmgr과 같은 도구를 사용하여 관리된다.
 피어 검증을 수행할 때 RabbitMQ는 신뢰할 수 있는 루트 인증서(목록의 첫 번째 인증서)만 고려하고 중간 인증서는 무시된다. 중간 인증서도 신뢰할 수 있는 것으로 고려되는 경우 신뢰할 수 있는 인증서 저장소에 추가해야 한다.<br>
 &nbsp;서버 및 클라이언트에서 사용하는 것과 같은 최종(“leaf”) 인증서를 신뢰할 수 있는 인증서 디렉토리에 배치하는 것이 가능하지만 훨씬 더 일반적인 방법은 CA 인증서를 신뢰할 수 있는 인증서 목록에 추가하는 것이다.<br>
 &nbsp;여러 인증서를 서로 추가하고 단일 인증기관 번들 파일에서 사용하는 가장 일반적인 방법은 단순히 인증서를 연결하는 것이다.<br><br>
- **피어 확인 사용(Enabling Peer Verfication)**<br>
&nbsp;서버 측에서 피어 검증은 기본적으로 ssl_options.verify 및 ssl_options.fail_if_no_peer_c
ert의 두 가지 구성 옵션을 사용하여 제어된다. ssl_options.fail_if_no_peer_cert 옵션을 false로 설정하면 노드가 인증서를 제공하지 않는 클라이언트(예 : 인증서를 사용하도록 구성되지 않은 클라이언트)를 수락한다. ssl_options/verify 옵션을 verify_peer로 설정하면 클라이언트는 사용자에게 인증서를 전송하므로 노드가 피어 검증을 수행한다. verify_none으로 설정하면 피어 검증이 비활성화되고 인증서 교환이 수행되지 않는다.<br>
&nbsp;예를 들어 다음 구성은 피어 검증을 수행하고 인증서를 제공하지 않는 클라이언트를 거부한다.<br>
&nbsp;피어 검증이 클라이언트 라이브러리에서 정확히 구성되는 방법은 라이브러리마다 다르다. Java 및 .NET 클라이언트 섹션에서는 해당 클라이언트의 피어 검증을 다룬다. Production 환경에서의 피어 확인을 적극 권장하며, 특정 환경에서는 이를 사용하지 않도록 설정할 수 있다.<br>
&nbsp;따라서 인증서를 확인하지 않고도 암호화된 TLS 연결을 만들 수 있다. 클라이언트 라이브러리는 일반적으로 두 가지 작동 모드를 모두 지원한다.<br>
&nbsp;피어 확인이 사용가능하게 되면 클라이언트가 연결하려는 서버의 호스트 이름이 SAN(Subject Alternative Name) 또는 CN(Common Name)과 같이 서버 인증서의 두 필드 중 하나와 일치하는지 여부도 확인하는 것이 일반적이다. 일치하지 않는 경우, 피어 확인도 클라이언트에 의해 실패한다.<br>
&nbsp;이 때문에 인증서를 생성할 때 사용된 SAN 또는 CN 값을 파악하는 것이 중요하다. 인증서가 한 호스트에서 생성되어 다른 호스트에서 사용되는 경우 $(hostname) 값을 대상 서버의 올바른 호스트 이름으로 바꿔야한다.<br>
&nbsp;tls-gen은 두 값에 대해 로컬 시스템의 호스트 이름을 사용한다. 마찬가지로 수동 인증서/키 쌍 생성 섹션에서 로컬 시스템의 호스트 이름은 일부 OpenSSL CLI 도구 명령에 대해 <u>...-subj /CN=$(hostname)/...</u>로 지정된다.<br><br>
- **인증서 체인 및 검증 깊이(Certificate Chains and Vertification Depth)**<br>
&nbsp;중간 CA가 서명한 클라이언트 인증서를 사용할 경우 더 높은 검증 깊이를 사용하도록  RabbitMQ 서버를 구성해야한다. 깊이는 유효한 인증서 경로에서 피어 인증서를 따를 수 있는 자체 발행되지 않은 중간 인증서의 최대 수를 말한다. 따라서 깊이가 0인 경우 피어(예 : 클라이언트) 인증서는 신뢰할 수 있는 CA에 의해 직접 서명되어야 한다.<br>
&nbsp;다음은 RabbitMQ 서버의 인증서 검증 깊이를 구성하는 방법을 보여주는 예이다.<br><br>
## Java 클라이언트에서 TLS 사용(Using TLS in the Java Client)

- **Key Manager, Trust Manager 및 stores**<br>
&nbsp;Java 보안 프레임 워크에는 Key Manager, Trust Manager 및 Key Store라는 세 가지 주요 구성 요소가 있다.<br>
&nbsp;Key Manager는 피어가 인증서를 관리하는 데 사용된다. TLS 연결/세션 협상 중에 키 관리자는 원격 피어에 보낼 인증서를 제어한다.<br>
&nbsp;Trust Manager는 피어에 의해 원격 인증서를 관리하는 데 사용된다. TLS 연결/세션 협상 중에 Trust Manager는 원격 피어에서 신뢰할 수 있는 인증서를 제어한다. Trust Manager는 모든 인증서 체인 검증 논리를 구현하는 데 사용할 수 있다.<br>
&nbsp;Key Store는 인증서 저장소 개념의 Java 캡슐화이다. 모든 인증서는 Java 전용 2진 형식(JKS)으로 저장되거나 PKCS# 12형식으로 저장되어야 한다. 이러한 형식은 KeyStore 클래스를 사용하여 관리된다.<br><br>
- **피어 확인 연결 중 사용 가능(Connecting with Peer Verification Enabled)**<br>
&nbsp;Java 클라이언트가 서버를 신뢰하려면, 서버 인증서는 Trust Manager를 인스턴스화하는 데 사용될 trust store에 추가해야 한다. JDK는 인증서 저장소를 관리하는 <u>keytool</u>이라는 도구가 함께 제공된다. 인증서를 저장소로 가져오려면 <u>keytool -import</u>를 사용하면 된다.<br>
&nbsp;모든 인증서와 키는 store에서 고유한 이름을 가져야하며 keytool은 인증서가 신뢰되었는지 확인하고 암호를 요청한다. 암호는 임의의 변경 시도로부터 Trust Store를 보호한다.<br><br>
- **서버 호스트 이름 확인(Server Hostname Verification)**<br>
&nbsp;호스트 이름 확인은 ConnectionFactory # enableHostnameVerification() 메소드를 사용하여 별도로 사용하도록 설정해야 한다.<br>
&nbsp;이렇게 하면 클라이언트가 연결중인 호스트 이름에 대해 서버 인증서가 발급되었는지 확인할 수 있다. 인증서 체인 확인과 달리 이 기능은 클라이언트별로 제공된다.<br>
&nbsp;JDK 6의 경우, 호스트명 검증을 위해 Apache Commons HttpClient에 대한 의존성을 추가해야한다.<br>
&nbsp;또는 JDK 6 <u>ConnectionFactory#enableHostnameVerification(HostnameVerifier)</u>에서 <u>HostnameVerifier</u> 인스턴스를 선택할 수 있다.<br><br>
- **Java 클라이언트에서 TLS 버전 구성(Configuring TLS Version in Java client)**<br>
&nbsp;RabbitMQ 서버가 특정 TLS 버전만 지원하도록 구성할 수 있다. 그렇기 때문에 Java 클라이언트에서 기본 설정 TLS 버전을 구성해야 할 수도 있다. 이 작업은 프로토콜 버전 이름 또는 <u>SSLContext</u>를 허용하는 <u>ConnectionFactory#useSslProtocol</u> 오버로드를 사용하여 수행된다.<br>
## .NET 클라이언트에서 TLS 사용(Using TLS in the .NET client)
- **.NET Trust Store**<br>
&nbsp;.NET 플랫폼에서 원격 인증서는 여러 저장소 중 하나에 배치하여 관리한다. 이 저장소의 모든 관리는 Microsoft의 .NET 구현과 Mono에서 사용할 수 있는 ‘certmgr’ 도구로 수행된다.<br>
&nbsp;클라이언트 인증서/키 쌍을 별도의 PKCS#12파일로 제공하기 때문에 루트 인증기관의 인증서를 루트(Windows) 또는 Trust(Mono) 저장소로 가져오기만 하면 된다. 해당 저장소의 모든 인증서로 서명된 모든 인증서는 자동으로 신뢰된다.<br>
&nbsp;피어 검증을 수행하지 않고 TLS 연결을 사용하게 되어있는 Java 클라이언트와 달리 .NET 클라이언트는 기본적으로 피어 검증이 성공해야 한다. 검증을 억제하기 위해 애플리케이션은 시스템은 SslOptions에 <u>System.Net.Security.SslPolicyErrors.RemoteC
ertificateNotAvailable</u> 및 <u>System.Net.Security.SslPolicyErrors.RemoteCertificateChainErrors</u> 플래그 설정을 할 수 있다.<br><br>
- **Certmgr을 사용한 인증서 관리(Certificate Management with Certmgr)**<br>
&nbsp;<u>certmgr</u>은 지정된 저장소의 인증서를 관리하는 명령 줄 도구이다. 이러한 저장소는 사용자별 저장소 또는 시스템 전체가 될 수 있다. 관리 사용자만 시스템 전체 저장소에 대한 쓰기 권한을 가질 수 있다.<br>
&nbsp;사용자 <u>Root</u>(일부 .NET 구현에서는 <u>Trust</u>라고도 한다.)의 저장소에 인증서를 추가하는 예이다.<br>
&nbsp;대신 시스템 전체(컴퓨터) 인증서 저장소에 인증서를 추가하려면<br>
&nbsp;저장소에 추가한 후 –all 스위치를 사용하여 해당 저장소의 내용을 볼 수 있다.<br>
&nbsp;저장된 인증서를 삭제하기 위해서는<br><br>
- **연결 생성(Creating The Connection)**<br>
&nbsp;RabbitMQ에 대한 TLS 사용 연결을 만들려면 ConnectionFactory의 Parameters 필드에 몇 가지 새로운 필드를 설정해야 한다. 작업을 쉽게 하기 위해 설정해야 하는 다른 모든 필드의 Name Space처럼 작동하는 새로운 Field Parameters.Ssl이 있다. 필드는 다음과 같다.<br>

|Property|설명|
|---|---|
|<u>Ssl.CerPath</u>|서버가 클라이언트 측 확인을 기대할 경우 PKCS#12형식의 클라이언트 인증서에 대한 경로이며, 이것은 선택사항이다.|
|<u>Ssl.CertPassphrase</u>|PKCS#12형식으로 클라이언트 인증서를 사용하는 경우 이 필드에 암호를 지정해야 한다.|
|<u>Ssl.Enabled</u>|TLS 지원을 켜거나 끄는 부울 필드이며, 기본적으로 해제되어 있다.|
|<u>Ssl.ServerName</u>|.NET은 서버가 보내는 인증서의 SAN(Subject Alternative Name) 또는 CN(Common Name)과 일치해야 한다.|
- **.NET 클라이언트에서 TLS 피어 확인(TLS Peer Verification in .NET client)**<br>
&nbsp;TLS는 클라이언트 및 서버가 피어의 인증서 정보를 기반으로 서로 ID를 확인하는 방법인 피어 확인(유효성 검사)을 제공한다. 피어 확인이 활성화되면 일반적으로 연결하려는 서버의 호스트 이름이 서버 인증서의 CN필드와 일치해야 한다. 그렇지 않으면 인증서가 거부된다. 그러나 피어 검증은 일반적으로 CN 및 호스트 이름 일치에만 국한될 필요는 없다.<br>
&nbsp;RabbitMQ .NET 클라이언트에서 <u>RabbitMQ.client.SslOptions.CertificatevaliddationCallback</u>을 사용하여 RemoteCertificatevalidationCallback 대리인을 제공할 수 있다. 대리인은 응용 프로그램에 맞는 논리를 사용하여 피어(RabbitMQ 노드) ID를 확인하는데 사용된다. 이 값을 지정하지 않으면 AcceptablePolicyErrors 속성과 함께 기본 콜백이 사용되어 원격 서버 인증서가 유효한지 여부가 결정된다. <u>RabbitMQ.client.SslOptions.AcceptablePolicyErrors</u>의 <u>System.Net.Security.SslPolicyErrors.RemoteCertificateNameMismatch</u> 플래그를 사용하여 피어확인을 비활성화 할 수 있다(프로덕션 환경에서는 권장되지 않는다).<br>
&nbsp;<u>RabbitMQ.client.SslOption.CertificateSelectionCallback</u>은 피어 검증에 사용되는 로컬 인증서를 선택할 LocalCertificateSelectionCallback을 제공하는 데 사용할 수 있다.<br>
## 사용된 TLS 버전 제한(Limiting TLS Versions Used)
- **TLS 버전을 제한하는 이유(Why limit TLS Versions)**<br>
&nbsp;TLS는 시간이 지남에 따라 발전했으며 여러 버전이 사용되고 있다. 각 버전은 이전 버전의 단점을 토대로 제작되는데 대부분의 단점으로 인해 TLS(및 SSL)의 특정 버전에 영향을 주는 알려진 공격이 발생한다. 구형 TLS 버전을 비활성화하면 이러한 공격을 완화할 수 있다. 보안 요구 사항이 가장 높은 환경에서는 TLSv1.2만 지원하는 것이 일반적이다.<br><br>
- **TLS 버전을 제한하지 않는 이유(Why Not Limit TLS Versions)**<br>
&nbsp;TLS 버전을 TLSv1.2로 제한한다는 것은 이전 TLS 버전(예 : JDK6 또는 .NET4.0)을 지원하는 클라이언트만 연결할 수 있음을 의미한다.<br>
&nbsp;사용 가능한 TLS 프로토콜 버전을 제한하려면 versions 옵션을 사용하면 된다.<br><br>
- **사용 가능한 TLS 버전 확인(Verifying Enabled TLS Versions)**<br>
&nbsp;제공된 TLS 버전을 확인하려면 <u>openssl s_client</u>를 사용하면 된다.<br><br>
- **JDK 및 .NET용 TLS 버전 지원 표(TLS Version Support Table for JDK and .NET)**<br>
&nbsp;TLSv1.0을 비활성화하면 지원되는 클라이언트 플랫폼 수가 제한된다.

|TLS version|최소 JDK version|최소 .NET version|
|---|---|---|
|TLS 1.0|JDK 5(RabbitMQ Java client는 8 필요)|.NET 2.0(RabbitMQ .NET client는 4.5.1 필요)|
|TLS 1.1|JDK 5|.NET 4.5|
|TLS 1.2|JDK 7|.NET 4.5|

## Cipher Suites
- **사용 가능한 Cipher Suite 나열(Listing Available Cipher Suites)**<br>
&nbsp;실행 중인 노드의 Erlang 런타임에서 지원하는 암호 모음을 나열하려면 <u>rabbitmq-diagnost ics cipher_suites --openssl-format</u>을 사용해야 한다.<br>
&nbsp;이렇게 하면 OpenSSL 형식의 암호 그룹 목록이 생성된다.<br><br>
- **Cipher Suites 구성(Configuring Cipher Suites)**<br>
&nbsp;암호 그룹은 <u>ssl_options.ciphers</u> config 옵션(classic config 형식의 <u>rabbit.ssl_options.ciphers</u>)을 사용하여 구성된다.<br><br>
- **Cipher Suite Order**<br>
&nbsp;TLS 연결 협상 중에 서버와 클라이언트는 어떤 암호 스위트를 사용할 것인지를 협상한다. 서버의 TLS 구현이 선호도(암호 스위트 순서)를 지시하도록 하여 취약한 클라이언트가 의도적으로 약한 암호군을 공격하여 공격을 준비하는 것을 방지할 수 있다. 이를 위해서는 honor_cipher_order 및 honor_ecc_order를 true로 설정하면 된다.<br>
## 알려진 TLS 취약점 및 완화<br>(Known TLS vulnerabilities and Their Mitigation)
- **ROBOT**
&nbsp;ROBOT 공격은 RSA Cipher Suite을 사용하고 19.3.6.4 및 20.1.7 이전 버전의 Erlang/OT
P 버전에서 실행되는 RabbitMQ 설치에 영향을 준다. Erlang/OTP를 완화하려면 패치 버전으로 업그레이드하고 지원되는 암호 스위트 목록을 제한해야 한다.<br>
- **POODLE**
&nbsp;POODLE은 원래 SSL/TLS 공격으로 SSLv3을 손상시켰었는데 버전 3.4.0부터 RabbitMQ 서버는 SSLv3 연결을 허용하지 않는다. 따라서 Erlang 18.0 이상을 실행하여 TLS1.0 구현 취약점을 POODLE에 부작용 또는 TLSv1.0 지원을 비활성화 하는 것이 좋다.<br>
- **BEAST**
&nbsp;BEAST 공격은 TLSv1.0에 영향을 미치는 알려진 취약점이다. 이를 완화하기 위해서는 TLSv1.0 지원을 해제해야 한다.<br>
## TLS 설정 평가(Evaluating TLS Setups)
&nbsp;TLS에는 많은 구성 가능한 매개 변수가 있고 그 중 일부는 기록적인 이유로 기본에 적합하지 않은 기본값이 있으므로 TLS 설치 평가가 권장되는 방법이다. 예를 들어 POODLE, BEAST 등과 같은 알려진 공격을 받기 쉬운지 여부를 테스트하는 등 TLS 사용 서버 endpoints에서 다양한 테스트를 수행하는 여러 도구가 있다.<br>
- **testssl.sh**
&nbsp;testssl.sh는 HTTP를 제공하지 않는 프로토콜 endpoints과 함께 사용할 수 있는 분별있고 광범위한 TLS endpoints 테스트 도구이다. 이 도구는 많은 테스트를 수행한다. 모든 환경을 통과하면 모든 환경에 적합할 수도 있고 그렇지 않을 수도 있다. 예를 들어 많은 프로덕션 배포에서는 CRLs(Certificate Revocation lists)를 사용하지 않는다. 대부분의 개발 환경은 자체 서명된 인증서를 사용하므로 암호화 풀이 사용 가능한 최적의 집합에 대해 걱정할 필요가 없다.<br>
## Erlang 클라이언트에서 TLS 사용(Using TLS in the Erlang client)
&nbsp;RabbitMQ Erlang 클라이언트에서 TLS를 활성화하는 것은 네트워킹과 관련된 다른 설정을 구성하는 것과 비슷하다.<br>
- Erlang TLS 옵션
  - <u>cacertfile</u>의 옵션은 우리가 절대적으로 신뢰하고자 하는 루트 인증기관의 인증서를 지정한다.
  - <u>certfile</u>는 PEM 형식으로 클라이언트 자신의 인증서
  - <u>keyfile</u>은 PEM 형식으로 클라이언트의 개인키 파일<br><br>

 **<u>server_name_indication</u>** : 이 옵션을 서버에서 제공하는 인증서의 “Server Name Indication(서버 이름 표시)” 확인을 위해 TLS 연결을 설정할 서버의 호스트 이름으로 설정한다. 이렇게 하면 TLS 연결을 설정하는 동안 서버 인증서의 <u>CN=</u> 값이 확인된다. <u>server_name_indication</u>을 다른 호스트 이름으로 설정하거나 <u>disable</u>이라는 특수값을 사용하여 이 확인을 비활성화하면 이 동작을 무시할 수 있다. 기본적으로 SNI(Server Name Indication)는 활성화되어 있지 않다.<br>
**<u>verify</u>** : 이 옵션을 verify_peer로 설정하여 X509 인증서 체인 확인을 활성화한다. <u>depth</u> 옵션은 인증서 검증 깊이를 구성한다. 기본적으로 <u>verify</u>는 <u>verify_none</u>으로 설정되어 있으므로 인증서 체인 확인이 비활성화 된다.<br><br>
- **CA, 인증서 및 개인키 수동 생성(Manually Generating a CA, Certificates and Private Keys)**<br>
1.먼저 테스트 인증기관을 위한 디렉토리를 만든다.<br>
2.테스트 인증기관이 사용할 키와 인증서를 생성한다.<br>
3.테스트 인증기관이 사용할 키와 인증서를 생성한다.

# Production Checklist
## 가상 호스트, 사용자, 권한(Virtual Hosts, Users, Permissions)
- **가상 호스트(Virtual Hosts)**<br>
&nbsp;단일 테넌트 환경에서 RabbitMQ 클러스터가 프로덕션 환경에서 단일 시스템에 전원을 공급할 때, 기본 가상 호스트(/)를 사용하는 것이 좋다. 멀티 테넌트 환경에서는 각 테넌트/환경별로 별도의 가상 호스트를 사용해야 한다.<br>
- **사용자(Users)**<br>
&nbsp;프로덕션 환경에서는 기본 사용자(guest)를 삭제해야 한다. 기본 사용자는 잘 알려진 자격 증명을 가지고 있기 때문에 기본적으로 localhost에서만 연결할 수 있다. 원격 연결을 사용하는 대신 관리 권한과 생성된 암호가 있는 별도의 사용자를 사용하는 것이 좋다.<br>
&nbsp; 애플리케이션마다 별도의 사용자를 사용하는 것이 좋다. 예를 들어 모바일 앱, 웹 앱 및 데이터 집계 시스템을 사용하는 경우 별도의 사용자가 3명 있는 것이다. 이것은 여러 가지 일을 더 쉽게 만든다.<br>
 - 애플리케이션과 클라이언트 연결의 연관성
 - 세분화된 사용권한을 사용
 - 자격증명 롤 오버(예:주기적으로 도는 위반 시)<br><br>

 &nbsp;동일한 애플리케이션의 인스턴스가 여러 개 있는 경우 더 나은 보안(인스턴스마다 자격 증명 집합 보유)과 프로비저닝(provisioning)의 편의성(일부 또는 모든 인스턴스 간에 자격 증명 집합 공유)간에 절충된다. 동일하거나 유사한 기능을 수행하고 고정 IP 주소를 갖는 많은 클라이언트가 포함된 IoT 애플리케이션의 경우 X509 인증서 또는 원본 IP 주소 범위를 사용하여 인증하는 것이 좋다.<br><br>

## 모니터링 및 리소스 제한(Monitoring and Resource Limits)
&nbsp;RabbitMQ 노드는 물리적 리소스(예 : 사용 가능한 RAM의 양)와 소프트웨어(프로세스가 열 수 있는 최대 파일 핸들 수)와 같은 다양한 리소스에 의해 제한된다. 프로덕션 환경에 들어가기 전에 리소스 제한 구성을 평가하고 그 이후의 리소스 사용을 지속적으로 모니터링 하는 것이 중요하다.<br>
- **모니터링(Monitoring)**<br>
&nbsp; -인프라 및 커널 메트릭에서 RabbitMQ, 애플리케이션(응용 프로그램) 수준 메트릭에 이르기까지 시스템의 여러 측면을 모니터링 하는 것이 필수적이다. 모니터링은 시간 면에서 선행 투자가 필요하지만 문제를 포착하고 잠재적으로 문제가 될 수 있는 것을 조기에 인지하는 데 매우 효과적이다.<br><br>
- **메모리(Memory)**<br>
&nbsp;RabbitMQ는 소비자가 따라가지 못할 때 Resource-driven 알림을 사용하여 producer를 제한한다.<br>
&nbsp;기본적으로 RabbitMQ는 사용 가능한 메모리의 40% 이상을 사용하고 있음을 감지할 때 새 메시지를 수락하지 않는다.(OS가 보고 한 바와 같이) : <u>{vm_memory_high_watermark, 0.4}</u>. 이는 안전한 기본값이며 호스트가 전용 RabbitMQ 노드인 경우에도 이 값을 수정할 때 주의해야 한다.<br>
&nbsp;OS 및 파일 시스템은 시스템 메모리를 사용하여 모든 시스템 프로세스의 작업을 가속화한다. 이 목적을 위해 충분한 여유 시스템 메모리를 두지 않으면 OS 스와핑으로 인해 시스템 성능에 악영향을 미치며 RabbitMQ 프로세스가 종료될 수도 있다.<br>
&nbsp;기본 <u>vm_memory_high_watermark</u>를 조정할 때 몇 가지 권장 사항이 있다.<br>
 - RabbitMQ를 호스팅하는 노드는 항상 128MB 이상의 메모리가 있어야 한다.
 - 권장되는 <u>vm_memory_high_watermark</u> 범위는 <u>0.40~0.66</u>이다.
 - <u>0.7</u>보다 큰 값은 권장하지 않습니다. OS와 파일 시스템은 메모리의 30%이상을 남겨두어야 한다. 그렇지 않으면 페이징으로 인해 성능이 심각하게 저하될 수 있다.<br><br>
- **디스크 공간(Disk Space)**<br>
&nbsp;현재 50MB disk_free_limit default는 개발 및 튜토리얼에 매우 효과적이다. 디스크 공간이 부족하면 노드 고장이 발생하고 모든 디스크 쓰기가 실패하기 때문에 데이터 손실을 초래할 수 있다.<br>
&nbsp;기본 값이 50MB인 이유는 개발환경은 때때로 /var/lib를 호스팅하기 위해 매우 작은 분할을 사용하기도 하는데, 이는 부팅 직후 노드가 자원 알림(Resource Alarm) 상태가 된다는 것을 의미한다. 매우 낮은 default는 RabbitMQ가 모든 사람을 위해 즉시 작동하게 한다.<br>
 - {disk_free_limit, {mem_relative, 1.0}}은 최소 권장 값이며 사용 가능한 총 메모리 양으로 변환된다. 예를 들어 시스템 메모리가 4GB인 RabbitMQ 전용 호스트에서 사용 가능한 디스크 공간이 4GB 이하로 떨어지면 모든 publisher가 차단되고 새로운 메시지가 수신되지 않는다.<br>
 - {disk_free_limit, {mem_relative, 1.5}}가 더 안전한 생산값이다. 메모리가 4GB인 RabbitMQ 노드에서 사용 가능한 디스크 공간이 6GB이하로 떨어지면 디스크 알림(경보)이 해제될 때까지 모든 새 메시지가 차단된다.<br>
 - {disk_free_limit, {mem_relative, 2.0}}은 가장 보수적인 생산값이다.<br><br>
- **Open File Handles 제한(Open File Handles Limit)**<br>
&nbsp;운영 체제는 네트워크 소켓을 포함하는 동시에 열려있는 파일 핸들의 최대 수를 제한한다. 예상되는 동시 연결 수 및 큐 수를 허용하는 한도를 설정해야한다.<br>
&nbsp;개발 환경을 포함하여 효과적인 RabbitMQ 사용자를 위해 최소한 50K개의 열린 파일 디스크립터가 환경에 허용되는지 확인해야 한다.<br><br>
- **로그 수집(Log Collection)**<br>
&nbsp;가능한 경우 모든 RabbitMQ 노드 및 응용 프로그램의 로그를 수집하고 집계하는 것이 좋다. 비정상적인 시스템 동작을 조사 할 때 로그가 중요할 수 있다.<br>

## 응용 프로그램 고려사항(Application Considerations)
&nbsp;응용 프로그램이 설계되고 RabbitMQ 클라이언트 라이브러리를 사용하는 방식은 전반적인 시스템 복원력에 주요 기여한다. 자원을 비효율적으로 사용하거나 유출하는 애플리케이션은 결국 나머지 시스템에 영향을 미친다. 예를 들어 연결을 계속 열지만 애플리케이션은 클러스터 노드를 파일 디스크립터(descriptors)에서 배출하여 새로운 연결이 허용되지 않는다.<br>
&nbsp;이 섹션에서는 가장 일반적인 여러 가지 문제에 대해 설명한다. 이러한 문제의 대부분은 일반적으로 프로토콜별 또는 새로운 것이 아니다. 그러나 이러한 문제는 발견하기가 어려울 수 있다. 시스템의 적절한 모니터링은 문제가 있는 추세(예 : 채널 유출, 열악한 연결 관리로 인한 파일 설명자 사용 증가)를 조기에 발견 할 수 있는 유일한 방법이기 때문에 매우 중요하다.<br><br>
- **연결 관리(Connection Management)**<br>
&nbsp;메시징 프로토콜은 일반적으로 수명이 긴 연결을 사용한다. 일부 애플리케이션은 시작할 때 RabbitMQ에 연결하고 종료해야하는 경우에만 연결을 닫는다. 다른 애플리케이션들은 연결을 보다 동적으로 열고 닫는다. 후자의 경우, 더 이상 사용하지 않을 때 닫는 것이 중요하다.<br>
&nbsp;연결은 애플리케이션 개발자의 통제 범위를 벗어나는 이유로 닫을 수 있다. RabbitMQ가 지원하는 메시징 프로토콜은 TCP 스택보다 빠른 연결을 감지하기 위해 heartbeat라는 기능을 사용한다. 개발자는 네트워크 충돌 또는 시스템 부하가 상승할 때 오탐(false positive)를 유발할 수 있는 너무 낮은(5초미만의) heartbeat 제한 시간을 사용하는 것에 주의해야한다.<br><br>
- **연결 변동(Connection Churn)**<br>
&nbsp;메시징 프로토콜은 일반적으로 수명이 긴 연결을 사용한다. 일부 응용 프로그램은 새로운 연결을 열어 단일 작업(예 : 메시지 게시)을 수행하기 위해 새로운 연결을 연 후 닫을 수 있다. 이는 연결을 여는 것이 기존의 것을 재사용하는 것과 비교할 때 값비싼 작업이기 때문에 매우 비효율적이다. 이러한 작업 부하로 인해 연결이 끊어질 수도 있다. 높은 연결 변동을 경험한 노드는 커널 기본 값보다 훨씬 빠르게 TCP 연결을 해제하도록 조정해야한다. 그렇지 않으면 결국 파일 핸들이나 메모리가 부족하여 새로운 연결을 받지 않게 된다.<br>
&nbsp;적은 수명의 연결을 옵션으로 사용할 수없는 경우 연결 풀링을 사용하면 최대 리소스 사용량을 줄일 수 있다.<br><br>
- **연결 실패에서 복구(Recovery from Connection Failures)**<br>
&nbsp;일부 클라이언트 라이브러리 (예 : Java , .NET 및 Ruby )는 네트워크 장애 후 자동 연결 복구를 지원한다. 사용된 클라이언트가 이 기능을 제공하는 경우 자체 복구 매커니즘을 개발하는 대신 이 기능을 사용하는 것이 좋다.<br>
&nbsp;다른 클라이언트(예 : Go, Pika)는 자동 연결 복구 기능을 지원하지 않지만 연결 실패로부터 복구하는 방법을 보여주는 예를 제공한다.<br><br>
- **과도한 채널 사용(Excessive Channel Usage)**<br>
&nbsp;채널은 또한 클라이언트와 서버 모두에서 자원을 소비한다. 응용 프로그램은 가능한 경우 사용하는 채널수를 최소화하고 더 이상 필요하지 않은 채널을 닫아야한다. 연결을 닫으면 자동으로 모든 채널이 닫힌다.<br>

## 보안 고려사항(Security Considerations)
- **노드 간 그리고 CLI 도구 인증(Inter-node and CLI Tool Authentication)**<br>
&nbsp;RabbitMQ 노드 는 파일에 저장된 공유 암호를 사용하여 서로 인증한다. Linux 및 다른 UNIX와 같은 시스템에서는 RabbitMQ 및 CLI 도구를 실행할 OS 사용자에게만 쿠키 파일 액세스를 제한할 필요가 있다.<br>
&nbsp;값이 합리적으로 안전한 방법(예 : 값을 추측하기 위해 쉬운 값으로부터 계산되지 않음)으로 생성되는 것이 중요하다. 이는 일반적으로 초기 구축 시 구축 자동화 도구를 사용하여 이루어진다. 이러한 도구는 기본값 또는 placeholder 값을 사용할 수 있다. 하나의 노드에서 쿠키 파일을 생성하고 다른 모든 노드에 복사하는 런타임을 허용하는 것은 좋지 않은 방법이다. 생성 알고리즘을 알고 있기 때문에 생성된 값을 더 예측 가능하게 만든다.<br>
&nbsp;CLI 도구는 동일한 인증 메커니즘을 사용한다. 노드 간 및 CLI 통신 포트 액세스는 RabbitMQ 노드 또는 CLI 도구를 실행하는 호스트로 제한하는 것이 좋다.<br>
&nbsp;TLS와의 노드 간 통신을 보안하는 것이 좋다. 이는 CLI 도구가 TLS를 사용하도록 구성되어 있음을 의미한다.<br><br>
- **방화벽 구성(Firewall Configuration)**<br>
&nbsp;RabbitMQ에서 사용하는 포트는 크게 두 가지 범주 중 하나로 분류할 수 있다.
  - 클라이언트 라이브러리가 사용하는 포트(AMQP 0-9-1, AMQP 1.0,MQTT,STOMP,HTTP API)
  - 다른 모든 포트(노드 간 통신, CLI 도구 등)<br><br>

 &nbsp;후자의 범주의 포트에 대한 액세스는 일반적으로 RabbitMQ 노드 또는 CLI 도구를 실행하는 호스트로 제한되어야 한다.<br>
- **TLS**<br>
&nbsp;가능한 경우 트래픽을 암호화하려면 TLS 연결을 사용하는 것이 좋다. 피어 확인(인증)도 권장된다. 개발 및 QA 환경에서는 자체 서명된 TLS 인증서를 사용할 수 있다. 자체 서명된 인증서는 RabbitMQ 및 모든 응용 프로그램이 신뢰할 수 있는 네트워크에서 실행되거나 VMware NSX와 같은 기술을 사용하여 격리된 경우 프로덕션 환경에서 적합할 수 있다.<br>
&nbsp;RabbitMQ는 기본적으로 보안 TLS 구성을 제공하려고 하지만(예 : SSLv3가 비활성화됨) testssl.sh 와 같은 도구를 사용하여 TLS 구성(versions cipher suites 등)을 평가하는 것이 좋다. TLS는 RabbitMQ와 이를 사용하는 응용 프로그램의 CPU 사용량을 포함하여 전체 시스템 처리량에 상당한 영향을 줄 수 있다.<br>

## 클러스터링 고려 사항(Clustering Considerations)
- **클러스터 크기(Cluster Size)**<br>
&nbsp;클러스터 크기를 결정할 때는 몇 가지 요소를 고려해야 한다.<br>
  - 예상 처리량
  - 예상 복제(미러수)
  - 데이터 지역<br><br>

 &nbsp;클라이언트는 모든 노드에 연결할 수 있기 때문에 RabbitMQ는 메시지와 내부 작업에 대한 클러스터 간 라우팅을 수행해야 할 수 있다. 가능하다면 consumer와 producer가 동일한 노드에 연결되도록 해야 한다. 이렇게 하면 노드 간 트래픽이 감소할 수 있다. 마찬가지로 도움이 되는 것은 consumer들이 현재 큐 마스터를 호스팅하는 노드에 연결하도록 만드는 것이다. 데이터 지역성을 고려할 때 전체 클러스터 처리량은 중요하지 않은 볼륨에 도달할 수 있다.<br><br>
- **노드 시간 동기화(Node Time Synchronization)**<br>
&nbsp;RabbitMQ 클러스터는 일반적으로 참여 서버의 클럭이 동기화되지 않고 잘 작동할 것이다. 그러나 관리 UI와 같은 일부 플러그인은 메트릭 처리에 로컬 타임스탬프를 사용하고 노드의 현재 시간이 벗어나면 잘못된 통계를 표시할 수 있다. 따라서 서버가 클럭을 동기화 상태로 유지하기 위해 서버가 NTP(Network Time Protocol) 또는 이와 유사한 것을 사용하는 것이 좋다.<br>

# Distributed RabbitMQ
&nbsp;플러그인(예 : STOMP)를 통해 RabbitMQ에서 지원하는 AMQP 및 다른 메시지 프로토콜은 본질적으로 분산되어 있다. - 여러 컴퓨터의 응용프로그램이 인터넷을 통해서라도 단일 브로커에 연결되는 것은 매우 일반적이다.<br>
&nbsp;그러나 떄로는 RabbitMQ 브로커 자체를 배포하는 것이 필요하거나 바람직하다. 이를 달성하는 세 가지 방법이 있는데 Clustering, federation, shovel을 사용하는 것이다.<br>
- **Clustering**<br>
&nbsp;클러스터링은 단일 논리 브로커를 구성하기 위해 여러 시스템을 연결한다. 통신(Communication)은 Erlang 메시지 전달을 통해 이루어지므로 클러스터의 모든 노드는 동일한 Erlang 쿠키를 가져야한다. 클러스터의 시스템(machines) 간 네트워크 링크는 신뢰할 수 있어야 하며 클러스터의 모든 시스템은 동일한 버전의 RabbitMQ 및 Erlang을 실행해야 한다.<br>
&nbsp;가상 호스트, 교환(exchanges), 사용자 및 권한은 클러스터의 모든 노드를 거쳐 자동으로 미러링된다. 큐는 단일 노드에 위치하거나 여러 노드에 미러링 될 수 있다. 클러스터의 노드에 연결하는 클라이언트는 해당 노드에 있지 않더라도 클러스터의 모든 큐를 볼 수 있다.<br>
&nbsp;일반적으로 단일 위치에 있는 시스템을 사용하여 고가용성 및 처리량 향상을 위해 클러스터링을 사용한다.<br><br>
- **Federation**<br>
&nbsp;Federation은 한 브로커의 교환 또는 큐가 다른 브로커에 있는 교환 또는 큐에 게시된 메시지를 수신할 수 있다(브로커는 개별 시스템 또는 클러스터 일 수 있다). 통신(Communication)은 AMQP(선택적인 SSL 포함)를 통해 이루어지므로 두 개의 교환 또는 큐에 federate하기 위해서는 적합한 사용자 및 권한을 부여받아야 한다.<br>
&nbsp;Federation된 교환은 한 방향의 point-to-point 링크와 연결된다. 기본적으로 메시지는 Federation 링크를 통해서만 한 번 보내지지만 보다 복잡한 라우팅 토폴로지를 허용하도록 늘어날 수 있다. 일부 메시지는 링크를 통해 전달되지 않을 수도 있으며 만약 메시지가 Federation된 Exchange에 도달한 후 큐로 라우팅되지 않으면 메시지는 처음부터 전달되지 않는다.<br>
&nbsp;Federation된 큐는 마찬가지로 한 방향의 point-to-point 링크와 연결된다. 메시지는 comsumer를 따라 가기 위해 임의의 횟수만큼 federation된 큐 사이에서 이동될 것이다.<br><br>
- **The shovel**<br>
&nbsp;shovel와 함께 브로커를 연결하는 것은 개념적으로 federation과 연결하는 것과 비슷하다. 그러나 shovel은 낮은 레벨에서 작동한다.<br>
&nbsp;Federation은 교환 및 큐의 독선적인 분배를 제공하는 것을 목표로 하지만, shovel은 그저 한 브로커의 큐에서 메시지를 소비하고 다른 브로커의 exchange에 전달하는 것을 목표로 한다.<br>
&nbsp;일반적으로 Federation이 제공하는 것보다 더 많은 통제(Control)가 필요할 때 인터넷을 통해 브로커들을 연결시키기 위해 shovel을 사용한다.<br><br>
- **Summary**

|Federation/shovel|Clustering|
|---|---|
|브로커는 논리적으로 분리되어 있으며 다른 소유자가 있을 수 있다.|클러스터는 단일 논리 브로커를 형성한다.|
|브로커는 다른 버전의 RabbitMQ와 Erlang을 실행할 수 있다.|노드는 동일한 버전의 RabbitMQ와 Erlang을 실행해야 한다.|
|브로커는 신뢰할 수 없는 WAN 링크를 통해 연결될 수 있다. 통신은 AMQP를 통해 이루어지며(SSL에 의해 선택적으로 보안됨) 적정한 사용자와 권한을 설정해야 한다.|브로커는 신뢰할 수 있는 LAN 링크를 통해 연결해야 한다. 통신은 Erlang 노드 간 메시징을 통해 이루어지며 제공된 Erlang 쿠키가 필요하다.|
|브로커는 배열한 어떤 토폴로지에 연결될 수 있다. 링크는 단방향 또는 양방향일 수 있다.|모든 노드는 양쪽 방향의 다른 모든 노드에 연결된다.|
|CAP theorem에서 가용성(Availability) 및 Partition Tolerance(AP)를 선택한다.|CAP theorem에서 일관성(Consistency) 및 Partition Tolerance(CP)를 선택한다.|
|브로커의 일부 교환은 federation될 수 있는 반면 어떤 교환은 로컬일 수 있다.|클러스터링은 아무것도 아니다.|
|어떤 브로커에 연결하는 클라이언트는 해당 브로커의 큐만 볼 수 있다.|어떤 노드에 연결하는 클라이언트는 모든 노드의 큐를 볼 수 있다.|
# Reliable Delivery(신뢰적인 전달)
- **What Can Fail?**<br>
&nbsp;네트워크 문제는 아마도 가장 흔한 실패의 종류일 것이다. 네트워크가 실패할 뿐만 아니라 방화벽이 유휴 연결을 중단시킬 수 있고 네트워크 장애가 항상 즉시 감지되는 것은 아니다.<br>
&nbsp;연결 장애 외에도, 브로커와 클라이언트 어플리케이션은 언제든지 하드웨어 고장(또는 소프트웨어가 충돌할 수 있음)을 경험할 수 있다. 또한, 클라이언트 어플리케이션이 계속 실행되더라도, 논리 오류(logic errors)로 인해 채널 또는 연결 오류가 발생하여 클라이언트가 새로운 채널 또는 연결을 설정하고 문제로부터 복구하도록 할 수 있다.<br><br>
- **연결 실패(Connnection Failures)**<br>
&nbsp;연결에 실패할 경우 클라이언트는 브로커에 대한 새 연결을 설정해야 할 필요가 있다. 이전 연결에서 열린 어떤 채널은 자동으로 닫히므로 채널도 다시 열어야 한다.<br>
&nbsp;일반적으로 연결이 실패할 때, 클라이언트는 예외를 던지는 연결(또는 유사한 언어 구성)에 의해 알려진다. 또한 공식 Java와 .NET 클라이언트는 다른 Contexts에서 연결 실패에 대해 들을 수 있는 callback 방법을 추가로 제공한다. Java는 <u>Connection</u> 및 <u>Channel</u> 클래스 모두에서 <u>ShutdownListener</u>을 제공하고 .NET 클라이언트는 동일한 목적을 위한 <u>IConnection.ConnectionShutdown</u>과 <u>IModel.ModelShutdown</u> 이벤트를 제공한다.<br><br>
- **Acknowledgements and Confirms**<br>
&nbsp;메시지는 전달 중 손실될 수 있다. 손실된 메시지들은 재전송되어야 하는데 확인 응답(Acknowledgements)은 서버와 클라이언트가 재전송을 언제 해야 하는지 알 수 있게 한다. 확인 응답은 양방향으로 사용될 수 있는데 consumer가 메시지를 수신/처리했다는 것을 서버에 알리도록 하고, 서버가 producer에게 동일한 것을 나타내도록 한다. RabbitMQ에서는 후자의 경우를 “confirm”이라고 한다.<br>
&nbsp;물론, TCP는 패킷이 수신되었음을 보장하고 패킷이 수신될 때까지 재전송할 것이지만 그것은 네트워크 계층일 뿐, 확인 응답과 confirms는 메시지가 수신되고 작용했음을 나타낸다. 확인 응답은 메시지의 수신과 수신자가 수신에 대한 모든 책임을 지는 것을 전제로 하는 소유권 양도 신호를 의미한다.<br>
&nbsp;그러므로 Acknowledgements에는 semantics(의미)를 가지고 있다. 소비하는 어플리케이션은 필요한 모든 작업을 수행할 때까지 메시지를 승인해서는 안 된다.<br>
&nbsp;Acknowledgements의 사용은 한 번에 전달되는 보장한다. acknowlegments 없이, 메시지 손실은 게시 및 소비 작업 중 가능하며, 오직 한 번의 전달만 보장된다.<br><br>
- **Detecting Dead TCP Connections With Heartbeats**<br>
&nbsp;일부 유형의 네트워크 장애에서 패킷 손실은 중단된 TCP 연결이 운영 체제에서 감지될 수 있도록 비교적 긴 시간(예 : Linix의 기본 구성에서 약 11분)이 소요됨을 의미할 수 있다. AMQP 0-9-1은 애플리케이션 계층이 중단된 연결(및 완전히 응답하지 않는 피어)을 즉시 알 수 있도록 하기 위해 Heartbeat 기능을 제공한다. 또한 Heartbeat는 “idle(유휴)” TCP 연결을 종료시킬 수 있는 특정 네트워크 장비에 대해서도 방어한다.<br><br>
- **Clustering and High Availability**<br>
 &nbsp;만약 브로커가 하드웨어 장애를 견뎌 낼 수 있도록 해야 한다면 RabbitMQ의 클러스터링을 사용할 수 있다. RabbitMQ 클러스터에서는 모든 정의(definitions)(교환, 바인딩, 사용자 등)는 전체 클러스터에 걸쳐 미러링된다. 큐는 기본적으로 단일 노드에 상주하면서 다르게 작동하지만, 선택적으로 몇몇의 노드 또는 모든 노드에 미러링된다. 큐는 위치에 관계없이 모든 노드에서 볼 수 있고 도달할 수 있다.<br>
 &nbsp;미러링 된 큐는 구성된 모든 클러스터 노드에 걸쳐 내용을 복제하여 메시지 손실 없이 원활하게 노드 장애를 견딜 수 있다. 그러나 사용중인 애플리케이션은 큐에 오류가 발생하면 해당 사용자는 취소되고 다시 시작해야 한다.<br><br>
- **At Producer**<br>
&nbsp;Confirms를 사용할 때 채널 또는 연결 실패로부터 복구하는 producer는 브로커로부터 확인 응답을 받지 못한 메시지를 재전송 해야 한다. 브로커가 네트워크 장애 등으로 인해 producer에 도달하지 않은 확인을 전송했을 수 있으므로 메시지가 중복될 수 있다. 따라서 consumer 애플리케이션은 중복 제거를 수행하거나 idempotent 방식으로 수신 메시지를 처리해야 한다.<br><br>
- **메시지가 라우팅되는지 확인(Ensuring Messages are Routed)**<br>
&nbsp;어떤 상황에서는 producers가 메시지가 큐로 라우팅되는 것을 보장하는 것이 중요할 수 있다. 메시지가 알려진 단일 큐로 라우팅 되도록 하기 위해 producer는 대상 큐를 선언하고 직접 큐에 게시할 수 있다.<br>
&nbsp;또한 producers는 클러스터된 노드에 게시할 때 exchange에 바인딩된 하나 이상의 대상 큐가 클러스터에 미러가 있는 경우, 복제본과 마스터 큐 프로세스 간의 흐름 제어 때문에 노드 간에 네트워크 장애에 직면하여 지연이 발생할 수 있다.<br><br>
- **At the Consumer**<br>
&nbsp;네트워크 장애(또는 노드 충돌)가 발생하는 경우 메시지는 중복될 수 있으며 Consumer는 이를 처리할 준비가 되어 있어야 한다. 가장 간단한 방법은 Consumer가 분명하게 중복 제거를 처리하는 대신 idempotent 방식으로 메시지를 처리하도록 하는 것이다.<br>
&nbsp;만약 메시지가 consumer에게 전달된 다음 다시 큐에 넣어지면 RabbitMQ는 다시 전송될 때 <u>redelivered</u> 플래그를 설정한다. 이는 Consumer가 이전에 이 메시지를 본 적이 있다는 것을 암시한다. 반대로, 만약 <u>redelivered</u> 플래그가 설정되어 있지 않으면 이전에 메시지가 볼 수 없었다는 것을 의미한다.<br><br>
- **소비자 취소 알림(Consumer Cancel Notification)**<br>
&nbsp;어떤 상황에서는 서버가 소비한 큐가 삭제되거나 고장이 날 수 있기 때문에 서버가 consumer를 취소할 수 있어야 한다. 이 경우 consumer는 다시 소비해야하지만 이미 본 메시지를 다시 볼 수 있다.<br>
&nbsp;소비자 취소 알림은 AMQP에 대한 RabbitMQ 확장이므로 일부 클라이언트에서 지원되지 않을 수 있다.<br><br>
- **처리할 수 없는 메시지(Messages That Cannot Be Processed)**<br>
&nbsp;소비자가 메시지를 처리할 수 없다고 판단하면, <u>basic.reject</u>(또는 <u>basic.nack</u>)을 사용하여 메시지를 거부할 수 있다.<br><br>
- **분산된 RabbitMQ(Distributed RabbitMQ)**<br>
&nbsp;RabbitMQ는 신뢰할 수 없는 네트워크를 통해 노드 배포를 지원하는 2개의 플러그인을 제공한다. Federation과 shovel. 두 가지 모두 AMQP 클라이언트로 구현되며 둘 다 기본적으로 confirms과 acknowledgements를 사용하므로 confirms 및 acknowledgements를 사용하도록 구성하면 필요할 때 재전송한다.<br>
&nbsp;클러스터를 federation 또는 shovel과 연결할 때는 federation 링크와 shovel이 노드 고장을 허용하는지 확인하는 것이 바람직하다. federation은 다운스트림 클러스터에 링크를 자동으로 배포하고 다운스트림 노드의 장애가 발생하면 장애를 복구한다. 업스트림 노드에 장애가 발생할 때 새로운 업스트림에 연결하려면 업스트림에 여러 개의 중복 URIs를 지정하거나 TCP load balancer를 통해 연결한다.<br>
&nbsp;shovel을 사용할 때는 소스나 대상에 불필요한 브로커를 명시하는 것은 가능하지만, shovel 자체를 불필요한 것으로 만드는 것은 불가능하다.<br>

# Backup and restore
- **두 가지 유형의 노드 데이터(Two Types of Node Data)**<br>
&nbsp;모든 RabbitMQ 노드에는 해당 노드에 있는 모든 정보를 저장하는 데이터 디렉토리가 있다. 데이터 디렉토리에는 definitions(메타데이터, 스키마/토폴로지)와 메시지 저장소 데이터라는 두 가지 유형의 데이터가 포함된다.<br><br>
- **Definitions(Topology)**<br>
&nbsp;노드와 클러스터는 스키마, 메타 데이터 또는 토폴로지로 생각될 수 있는 정보를 저장한다. 사용자, 가상 호스트, 큐, 교환, 바인딩, 런타임 매개변수는 모두 이 범주(catagory)에 속한다.<br>
&nbsp;Definitions는 HTTP API, CLI 도구와 클라이언트 라이브러리(apps)에서 수행되는 선언(declarations)을 통해 내보내거나 가져올 수 있다.<br>
&nbsp;Definitions는 내부 데이터베이스에 저장되고 모든 클러스터 노드에 걸쳐 복제된다. 클러스터의 모든 노드에는 모든 definitions의 자체 복제본을 가지고 있다. 일부 definitions가 변경되면 단일 트랜잭션(single transaction)에서 모든 노드에 대해 업데이트가 수행된다. 백업의 맥락(context)에서 이는 실제로 definitions는 같은 결과로 어떤 클러스터 노드로부터 내보낼 수 있음을 의미한다.<br><br>
- **Messages**<br>
&nbsp;메시지는 메시지 저장소에 저장된다(“message store”를 사용자에게 투명한 단일 엔티티인 메시지의 내부 저장소로 정의한다). 각 노드는 자체 데이터 디렉토리를 가지고 있으며 해당 노드에서 마스터가 호스팅되는 큐에 대한 메시지가 저장된다. 메시지는 큐 미러링을 사용하는 노드 간에 복제될 수 있다. 메시지는 노드의 데이터 디렉토리의 서브 디렉토리에 저장된다.<br><br>
- **Data Lifecycle**<br>
&nbsp;Definitions는 일반적으로 정적이지만 메시지는 publisher에서 consumer로 지속적으로 전달된다.<br>
&nbsp;백업을 수행할 때 첫 번째 단계는 Definitions만 백업할 것인지, 아니면 메시지 저장소도 백업을 할 것인지 여부를 결정하는 것이다. 왜냐하면 메시지는 종종 수명이 짧고 가능한 한 일시적으로 머무르기 때문에 실행중인 노드에서 메시지를 백업하는 것은 매우 바람직하지 않으므로 일관성 없는 데이터의 snapshot이 발생할 수 있기 때문이다. Definitions는 실행중인 노드에서만 백업할 수 있다.<br><br>
- **Backing Up Definitions**<br>
&nbsp;Definitions는 JSON 파일로 내보내거나 수동으로 백업될 수 있다. 대부분의 경우 Definitions 내보내기/가져오기가 이를 수행하는 최적의 방법이다. 수동 백업은 노드 이름이나 호스트 이름을 변경할 경우 추가 단계가 필요하다.<br><br>
- **Exporting 내보내기(Exporting Definitions)**<br>
&nbsp;Definitions는 HTTP API를 사용하여 JSON 파일로 내보내진다<br>
  - Overview 페이지에 Definitions 창이 있다.
  - rabbitmqadmin은 definitions을 내보내는 명령을 제공한다.
  - <u>GET/api/definitions</u> API endpoint를 직접 호출할 수 있다.<br><br>

 &nbsp;Definitions은 특정 가상 호스트 혹은 전체 클러스터(혹은 독립된 노드)에 대해 내보낼 수 있다. 단일 가상 호스트 Definitions만 내보내는 경우 일부 정보(예 : 클러스터 사용자 및 권한)가 결과 파일에서 제외된다.<br>
&nbsp;내보낸 사용자 데이터는 해시 함수 정보와 암호 해시를 포함한다.
- **Definitions 가져오기(Importing Definitions)**<br>
&nbsp;Definitions가 있는 JSON 파일은 동일한 세 가지 방법으로 가져올 수 있다.<br>
  - Overview 페이지의 Definitions 창이 있다.
  - rabbitmqadmin은 Definitions를 가져오는 명령을 제공한다.
  - <u>POST/api/definitions</u> API endpoint를 직접 호출할 수 있다.<br><br>

 &nbsp;또한 <u>load_definitions</u> 구성 매개 변수를 통해 노드 부트 시 로컬 파일에서 definitions를 로드할 수도 있다.<br>
 &nbsp;Definitions 파일을 가져오면 동일한 definitions 집합(예 : 사용자(users), 가상호스트, 권한, 토폴로지)을 가진 브로커를 생성하기에 충분하다.<br>
- **수동으로 Definitions 백업(Manually Backing Up Definitions)**<br>
&nbsp;Definitions는 노드의 데이터 디렉토리에 위치한 내부 데이터베이스에 저장된다. 디렉토리 경로를 얻으려면 실행중인 RabbitMQ 노드에 대해 다음 명령을 실행하면 된다.<br>
&nbsp;`rabbitmqctl eval 'rabbit_mnesia:dir()'`<br><br>
&nbsp;만약 노드가 실행 중이 아니라면 기본 데이터 디렉토리를 검사할 수 있다.<br>
  - Debian 및 RPM 패키지의 경우 : <u>/var/lib/rabbitmq/mnesia</u>
  - Windows의 경우 : <u>%APP_DATA%\RabbitMQ\db</u>
  - MacOS 및 일반 UNIX 패키지의 경우 : <u>{installation_root}/var/lib/rabbitmq/mnesia</u><br><br>
- **수동 Definitions 백업에서 복원(Restoring from a Manual Definitions Backup)**<br>
&nbsp;내부 노드 데이터베이스는 노드 이름을 특정 레코드에 저장한다. 노드 이름이 변경되면 rabbitmqctl 명령을 사용하여 변경 사항을 반영하도록 데이터베이스를 먼저 업데이트해야 한다.<br>
&nbsp;`rabbitmqctl rename_cluster_node <oldnode> <newnode>`<br><br>
&nbsp;이 명령은 클러스터의 여러 노드가 동시에 이름이 변경되는 경우 여러 개의 이전 이름/새 이름 쌍을 사용할 수 있다. 새 노드가 백업된 디렉토리와 일치하는 노드 이름으로 시작할 때 필요에 따라 업그레이드 단계를 수행하고 부팅을 진행해야 한다.<br><br>
- **메시지 백업(Backing Up Messages)**<br>
&nbsp;노드에서 메시지를 백업하려면 먼저 중지시켜야 한다. 미러링된 큐가 있는 클러스터의 경우 백업을 수행하려면 전체 클러스터를 중지해야 할 필요가 있다. 한 번에 한 노드를 중지하면 단일 실행 노드를 백업할 때와 마찬가지로 메시지가 유실되거나 중복될 수 있다.<br><br>
- **수동으로 메시지 백업(Manually Backing Up Messages)**<br>
&nbsp;현재 메시지를 백업하는 유일한 방법이다. RabbitMQ 버전 3.7.0부터는 모든 메시지 데이터는 <u>msg_stores/vhosts</u> 디렉토리에 결합되고 가상호스트당 서브디렉토리에 저장된다. 각 가상호스트 디렉토리는 해시와 함께 명명되고 가상호스트 이름을 가진 <u>.vhost</u> 파일을 포함하므로 특정 가상호스트의 메시지 세트는 별도로 백업될 수 있다.<br>
&nbsp;RabbitMQ 3.7.0 이전 버전에서 메시지는 노드 데이터 디렉토리(<u>queues</u>, <u>msg_store_persistent</u> 및 <u>msg_store_transient</u>) 아래 각각의 디렉토리에 저장된다. 또한 노드가 정상적으로 중지된 경우 복구 메타데이터를 포함하는 <u>recovery.dets</u> 파일도 있다.<br><br>
- **수동 메시지 백업에서 복원(Restoring from a Manual Messages Backup)**<br>
&nbsp;노드가 부팅되면 데이터 디렉토리 위치를 계산하고 메시지를 복원한다. 메시지를 복원하려면 브로커에 이미 모든 definitions이 있어야 한다. 알 수 없는 가상 호스트와 큐에 대한 메시지 데이터는 로드되지 않으며 노드에서 삭제할 수 있다. 따라서 수동으로 메시지 디렉토리를 백업할 때는 definition 파일을 가져오거나 전체 노드 데이터 디렉토리 백업을 통해 대상 노드(복원중인 파일)에서 definitions이 이미 사용 가능한지 확인하는 것이 중요하다.<br>
&nbsp;노드의 데이터 디렉토리가 수동으로 백업된 경우, 노드는 모든 definitions과 메시지와 함께 시작해야 한다.

# Alarms(Memory and Disk Alarms)
&nbsp;RabbitMQ가 클라이언트 네트워크 소켓에서 읽는 것을 멈추게하는 두 가지 상황이 있는데, 이는 OS에 의해 죽는 것을 피하기 위해서이다.<br>
- 메모리 사용이 설정된 제한을 초과할 때
- 디스크 공간이 설정된 한계 아래로 떨어질 때<br>

&nbsp;두 경우 모두 서버는 연결을 일시적으로 차단한다. 서버는 메시지를 게시한 연결된 클라이언트의 소켓에서 읽기를 일시 중지한다. 연결(Connection) heartbeat 모니터링도 비활성화된다. 모든 네트워크 연결은 rabbitmqctl 및 관리 플러그인에서 blocking으로 표시하며 이는 게시를 시도하지 않았고 그러므로 계속(continue) 혹은 blocked 될 수 있음을 의미하고, 이는 플러그인이 게시되었고 현재 일시 중지되었음을 의미한다. 호환이 되는 클라이언트는 차단이 되면 알림을 받게 된다.<br><br>
- **클라이언트 알림(Client Notification)**<br>
&nbsp;최신 클라이언트 라이브러리는 connection.blocked notification(프로토콜 확장)을 지원한다. 그래서 애플리케이션은 애플리케이션이 차단된 시점을 모니터링 할 수 있다.<br><br>
- **클러스터의 경보(Alarms in Clusters)**<br>
&nbsp;클러스터에서 RabbitMQ를 실행할 때 메모리 및 디스크 알람은 클러스터 전체에 표시되며 만약 한 노드가 제한을 초과하면 모든 노드가 연결을 차단한다.<br>
&nbsp;여기서의 목적은 producer를 멈추게 하는 것이지만 consumer가 영향을 받지 않게 하는 것이다. 그러나 프로토콜이 producer와 consumer가 동일한 채널에서, 그리고 단일 연결의 다른 채널에서 작동할 수 있도록 허용하기 때문에, 이 논리는 어쩔 수 없이 불완전하다. 실제로 throttling은 그저 지연으로 볼 수 있기 때문에 대부분의 애플리케이션에 어떤 문제를 일으키지 않는다. 그럼에도 불구하고, 허용하는 다른 설계 고려 사항은 생산(producing)하거나 혹은 소비(consuming)하기 위해 개별 연결만 사용하는 것이 바람직하다.

## Memory Alarms
&nbsp;RabbitMQ 서버는 시작할 때 컴퓨터에 설치된 총 RAM의 양을 감지하고 rabbitmqctl set_vm_memory_high_watermark fraction이 실행된다. 기본적으로 RabbitMQ 서버가 사용 가능한 RAM의 40% 이상을 사용하면 메모리 경고가 발생하고 메시지를 게시하는 모든 연결이 차단된다. 메모리 알람이 해제되면 정상적인 서비스가 재개된다.<br>
&nbsp;기본 메모리 임계값은 설치된 RAM의 40%로 설정된다. 이는 40% 이상을 사용하는 것을 RabbitMQ 서버가 막지는 못하지만 그저 publishers가 조절되는 지점(point)일 뿐이다. Erlang의 garbage collector는 최악의 경우 메모리 사용량을 두 배로 늘릴 수 있다(기본적으로 RAM의 80%).<br>
- **메모리 임계값 구성(Configuting the Memory Threshold)**<br>
&nbsp;흐름 제어가 발생되는 메모리 임계값은 구성 파일을 편집하여 조절할 수 있다. 다음 예에는 임계값을 기본 값 0.4로 설정하는 예이다.<br>
&nbsp;기본값 0.4는 설치된 RAM의 40% 또는 사용 가능한 가상 주소 공간의 40% 중 작은 값을 의미한다. 예를 들어 32비트 플랫폼에서 4GB의 RAM을 설치한 경우 4GB의 40%는 1.6GB이지만 32비트 Windows는 일반적으로 프로세스를 2GB로 제한하므로 임계값은 실제로 2GB의 40%(820MB)이다.<br>
&nbsp;또는 메모리 임계값은 노드에서 사용하는 RAM의 절대적인 제한을 설정하여 조절될 수 있다. 다음 예는 임계값을 1073741824바이트(1024MB)로 설정한다.<br>
&nbsp;절대적인 한계가 설치된 RAM 또는 사용 가능한 가상 주소 공간보다 큰 경우 임계값은 더 작은 쪽으로 설정된다.<br>
&nbsp;메모리 제한은 RabbitMQ 서버가 시작될 때 RABBITMQ_NODENAME.log 파일에 추가된다.<br>
&nbsp;메모리 제한은 <u>rabbitmqctl status</u> 명령을 사용하여 쿼리할 수도 있다.<br>
&nbsp;임계값은 브로커가 <u>rabbitmqctl set_vm_memory_high_watermark fraction</u> 명령 또는 <u>rabbitmqctl set_vm_memory_high_watermark absolute memory_limit</u> 명령을 사용하여 실행되는 동안 변경될 수 있다. 이 명령은 브로커가 멈출 때까지 적용된다. 효과가 브로커가 재시작시에도 지속되어야 할 때도 해당되는 구성 설정을 변경해야 한다. 메모리 제한은 RAM 시스템의 총량이 쿼리되기 때문에 이 명령을 임계값을 변경하지 않고 실행될 때 hoy-swappable RAM이 있는 시스템에서 변경할 수 있다.<br><br>
- **모든 게시 사용 중지(Disabling All Publishing)**<br>
&nbsp;<u>0</u> 값을 지정하면 메모리 알람이 즉시 해제되므로 모든 게시 연결이 차단되며, rabbitmqctl set_vm_memory_high_watermark 0을 사용한다.<br><br>
- **제한된 주소 공간(Limited Address Space)**<br>
&nbsp;64bit OS(또는 PAE가 있는 32bit OS)의 32bit Erlang VM에서 RabbitMQ를 실행하면 주소 지정이 가능한 메모리가 제한된다. 서버가 이를 감지하고 다음과 같은 메시지를 기록한다.<br>
&nbsp;메모리 알림 시스템은 완벽하지 않다. 게시를 중지하면 대개 더 이상의 메모리가 사용되는 것을 막을 수 있지만 다른 것들이 계속해서 메모리 사용을 증가시킬 수 있다. 일반적으로 이런 일이 일어나고 물리적인 메모리가 고갈되면 OS는 swap을 시작한다. 그러나 제한된 주소 공간을 사용하여 실행하면 제한을 초과하여 실행하면 VM이 충돌할 수 있다. 따라서 64bit OS에서 실행할 때는 64bit Erlang VM을 사용하는 것이 좋다.<br><br>
- **페이징 임계값 구성(Configuring the Paging Threshold)**<br>
&nbsp;브로커가 high watermark에 도달해 publishers를 차단하기 전에 큐에서 내용을 디스크로 페이징하도록 지시함으로써 메모리를 확보하려고 시도한다. 영구적인 메시지 및 일시적인 메시지는 모두 페이지 아웃된다(영구 메시지는 이미 디스크에 있지만 메모리로부터 제거될 것이다).<br>
&nbsp;기본적으로 이는 브로커가 high watermark에 이르는 경로의 50%일 때 발생한다(즉, 기본 high watermark는 0.4이며, 메모리의 20%가 사용되었을 때). 이 값을 변경하려면 vm_me
mory_high_watermark_paging_ratio 구성을 기본값인 0.5에서 수정해야 한다.<br>
&nbsp;위의 구성은 사용된 메모리의 30%에서 페이징을 시작하고 게시자(publisher)를 40%에서 차단한다.<br>
&nbsp;<u>vm_memory_high_watermark_paging_ratio</u>를 1.0보다 큰 값으로 설정할 수도 있다. 이 경우 큐는 디스트에 내용을 페이징하지 않는다. 이 때문에 메모리 알림이 울리게 하면 producer가 차단된다.<br><br>
- **인식할 수 없는 플랫폼(Unrecognised platforms)**<br>
&nbsp;RabbitMQ 서버가 시스템을 인식 할 수 없는 경우 RABBITMQ_NODENAME.log 파일에 경고를 추가한다. 그런 다음 1GB이상의 RAM이 설치되어 있다고 가정한다.<br>
&nbsp;이 경우, <u>vm_memory_high_watermark</u> 구성 값은 가정된 1GB RAM의 크기를 조정하는 데 사용된다. <u>vm_memory_high_watermark</u>의 기본 값을 0.4로 설정하면 RabbitMQ의 메모리 임계값이 410MB로 설정되므로 RabbitMQ가 410MB 이상의 메모리를 사용할 때마다 producer를 조절한다. 따라서 RabbitMQ가 플랫폼을 인식하지 못하는 경우 실제로 8GB RAM이 설치되어 있고 서버가 3GB를 초과하여 사용할 때 RabbitMQ가 producer를 제한하게하려면 <u>vm_memory_high_watermark</u>를 3으로 설정해야한다.

# Virtual Hosts
&nbsp;RabbitMQ는 connnections, exchanges, queues, binding, 사용자 권한, 정책과 기타 몇 가지 항목이 가상 호스트, 논리적 엔티티 그룹에 속해 있는 multi-tenant 시스템이다. Apache의 가상 호스트나 Nginx의 서버 블록에 익숙하다면, 아이디어는 비슷하다. 그러나 한 가지 중요한 차이점이 있다. Apache의 가상 호스트는 configuration 파일에 정의되어 있지만 RabbitMQ의 경우는 그렇지 않다. 대신 rabbitmqctl 또는 HTTP API를 사용하여 가상 호스트를 생성하고 삭제한다.
- **논리적 및 물리적 분리(Logical and Physical Separation)**<br>
&nbsp;가상 호스트는 자원(resources)의 논리적 그룹화 및 분리를 제공한다. 물리적 리소스의 분리는 가상 호스트의 목표가 아니며 구현 세부 사항으로 간주되어야 한다.<br>
&nbsp;예를 들어 RabbitMQ의 리소스 사용 권한은 가상 호스트별로 범위가 지정되며, 사용자는 전역(global) 권한이 없으며 하나 이상의 가상 호스트에 대한 사용 권한만 가지고 있다.<br>
&nbsp;따라서 사용자 사용 권한에 대해 이야기할 때 어떤 가상 호스트에 적용되는지 명확히 하는 것이 매우 중요하다.<br><br>
- **가상 호스트 및 클라이언트 연결(Virtual Hosts and Client Connections)**<br>
&nbsp;가상 호스트에는 이름이 있다. AMQP 0-9-1 클라이언트를 RabbitMQ에 연결할 때 연결할  가상 호스트 이름을 지정한다. 만약 인증이 성공하고 제공된 사용자 이름에 가상 호스트에 대한 사용 권한이 부여되면 연결이 설정된다.<br>
&nbsp;가상 호스트에 대한 연결은 해당 가상 호스트에서 교환, 큐, 바인딩 등에 대해서만 작동할 수 있다. 서로 다른 가상 호스트에서의 큐와 교환의 “interconnection(상호 연결)”은 애플리케이션이 동시에 두 가상 호스트에 연결할 때만 가능하다.<br><br>
- **가상 호스트 생성(Creating a Virtual Hosts)**<br>
&nbsp;가상 호스트는 CLI 도구 또는 HTTP API endpoint를 사용하여 생성할 수 있다.<br>
&nbsp;새로 생성된 가상 호스트에는 사용자 사용 권한이 없으므로 사용자가 가상 호스트에 연결하고 사용할 수 있도록 하려면 가상 호스트를 사용할 모든 사용자에게 사용 권한을 부여해야 한다(예 : rabbitmqctl set_permissions 사용).<br>
  - CLI 도구 사용<br>
 &nbsp;가상 호스트는 rabbitmqctl의 <u>add_vhost</u> 명령을 사용하여 생성할 수 있다.<br>
 &nbsp;다음은 qa1이라는 가상 호스트를 만드는 예이다.<br>
  - HTTP API 사용<br>
 &nbsp;가상 호스트는 PUT /api/vhosts/{name} HTTP API endpoint(엔드 포인트)를 사용하여 생성될 수 있다. {name}은 가상 호스트의 이름을 말한다.<br>
 &nbsp;다음은 curl을 사용하여 rabbitmq.local:15672의 노드에 연결하여 가상 호스트 vh1을 생성하는 예이다.<br><br>
- **가상 호스트 삭제(Deleting a Virtual Hosts)**<br>
&nbsp;가상 호스트는 CLI 도구 또는 HTTP API 엔드 포인트를 사용하여 만들 수 있다. 가상 호스트를 삭제하면 가상 호스트의 모든 엔티티(큐, 교환, 바인딩, 정책, 권한 등)가 영구적으로 삭제된다.<br>
  - CLI 도구 사용<br>
 &nbsp;가상 호스트는 rabbitmqctl의 <u>delete_vhost</u> 명령을 사용하여 삭제할 수 있다.<br>
 &nbsp;다음은 qa1이라는 가상 호스트를 삭제하는 예이다.<br>
  - HTTP API 사용<br>
 &nbsp;<u>DELETE/api/vhosts/{name}</u> HTTP API 엔드 포인트를 사용하여 가상 호스트를 삭제할 수 있다. 여기서 {name}은 가상 호스트의 이름이다.<br>

  &nbsp;다음은 curl을 사용하여 rabbitmq.local:15672의 노드에 연결하여 가상 호스트 vh1을 삭제하는 예입니다.<br><br>
- **가상 호스트와 STOMP(Virtual Hosts and STOMP)**<br>
&nbsp;AMQP 0-9-1과 마찬가지로 STOMP는 가상 호스트의 개념을 포함한다.<br><br>  
- **가상 호스트와 MQTT(Virtual Hosts and MQTT)**<br>
&nbsp;AMQP 0-9-1 및 STOMP와 달리 MQTT에는 가상 호스트라는 개념이 없다. MQTT 연결은 기본적으로 단일 RabbitMQ 호스트를 사용한다. 클라이언트가 특정 클라이언트 라이브러리를 수정하지 않고도 특정 가상 호스트에 연결할 수 있게 해주는 MQTT 관련 규칙 및 기능이 있다.<br><br>
- **Limits**<br>
&nbsp;경우에 따라 가상 호스트에서 허용되는 최대 큐 또는 동시 클라이언트 연결 수를 제한해야 한다. RabbitMQ 3.7.0에서는 호스트 별 제한을 통해 이를 수행할 수 있다.<br>
&nbsp;이러한 제한은 rabbitmqctl 또는 HTTP API를 사용하여 구성할 수 있다.<br><br>
- **rabbitmqctl을 사용하여 제한 구성(Configuring Limits Using rabbitmqctl)**<br>
&nbsp;<u>rabbitmqctl set_vhost_limits</u>는 가상 호스트 제한을 정의하는 데 사용되는 명령이다.<br><br>
- **최대 연결 제한(Configuring Max Connection Limit)**<br>
&nbsp;<u>vhost_name</u>에서 동시 클라이언트 연결의 총 수를 제한하려면 다음과 같이 하면 된다.<br>
&nbsp;가상 호스트에 대한 클라이언트 연결을 비활성화하려면 제한을 0으로 설정하면 된다.<br>
&nbsp;한계를 높이려면 음수 값으로 설정하면 된다.<br><br>
- **최대 큐의 수(Configuring Max Number of Queues)**<br>
&nbsp;<u>vhost_name</u>의 전체 큐의 수를 제한하려면 다음과 같이 하면 된다.<br>
&nbsp;한계를 높이려면 음수 값으로 설정하면 된다.

# Clustering
## High Available(Mirrored) Queues
- **Queue mirroring이란?(What is Queue Mirroring)**<br>
&nbsp;큐는 선택적으로 여러 노드에서 미러링될 수 있다. 미러링 된 각 큐는 하나의 master 및 하나 이상의 mirror로 구성된다. master는 흔히 마스터 노드라고 하는 하나의 노드에서 호스트된다. 각 큐에는 자체 마스터 노드가 있다. 주어진 큐에 대한 모든 작업은 먼저 큐의 마스터 노드에 적용된 다음 미러로 전달된다. 여기에는 큐에 게시하고, 소비자에게 메시지를 전달하고, 소비자로부터 수신 확인을 추적하는 작업이 포함된다.<br>
&nbsp;큐 미러링은 노드 클러스터를 의미합니다. 큐에 게시된 메시지는 모든 미러에 복제되며 소비자는 연결되는 노드와 상관없이 마스터에 연결되고 미러는 마스터에서 확인된 메시지를 삭제합니다. 따라서 큐 미러링은 가용성을 향상시키지만 노드에 로드를 분산시키지 않습니다.<br>
&nbsp;큐 마스터를 호스팅하는 노드가 실패하면 가장 오래된 미러는 동기화된 동안 새 마스터로 승격되며 큐 미러링 매개변수에 따라 비동기 미러도 승격될 수 있습니다.
### 미러링 구성방법(How Mirroring is Configured)
&nbsp;미러링 매개변수는 정책(policy)을 사용하여 구성된다. 정책은 이름에 의해 하나 혹은 그 이상의 큐를 연결시키며 일치하는 큐의 특징의 총 집합에 추가된 definition을 포함한다.

- **미러링을 제어하는 큐 인수(Queue Arguments that Control Mirroring)**<br>
&nbsp;큐는 정책(policy)을 통해 미러링을 사용할 수 있다. 정책은 언제든지 변경될 수 있다. 미러링되지 않은 큐를 만든 다음 나중에 미러링하도록 하는 것이 유효하다. 미러링되지 않은 큐와 미러링된 큐에는 차이가 있다. 미러링되지 않은 큐는 추가 미러링 인프라(infrastructure)가 없고 높은 처리량을 제공한다.<br>
&nbsp;큐가 미러링되도록 하려면 큐와 일치하는 정책을 생성하고 정책키 <u>ha-mode</u> 및 <u>ha-params</u>을 설정해야 한다.<br><br>
- **큐가 미러링되어 있는지 확인하는 방법(How to Check if a Queue is Mirrored?)**<br>
&nbsp;미러링된 큐에는 관리 UI의 큐 페이지에 정책 이름과 추가 복제본(미러)의 수가 표시된다.<br>
&nbsp;큐의 마스터 노드와 해당 온라인 미러가 큐 페이지에 나열된다.<br>
&nbsp;큐 페이지에 미러가 나열되지 않으면 큐가 미러링되지 않은 것이다.<br>
&nbsp;새 큐 미러가 추가하면 이벤트가 기록된다.<br>
&nbsp;<u>rabbitmqctl list_queues</u>를 사용하여 큐 마스터 및 미러를 나열할 수 있다.<br>
&nbsp;미러링 할 것으로 예상되는 큐가 아닌 경우, 이는 대개 큐의 이름이 미러링을 제어하는 정책에서 지정된 이름과 일치하지 않거나 다른 정책이 우선순위를 가짐을 의미한다.
### Queue Masters, Master Migration, Data Locality
- **큐 마스터 위치(Queue Master Location)**<br>
&nbsp;RabbitMQ의 모든 큐는 홈 노드를 가진다. 이 노드를 큐 마스터라고 하는데 모든 큐 작업은 먼저 마스터를 거친 후 미러로 복제된다. 이는 메시지의 FIFO 순서를 보장하는 데 필요하다. 큐 마스터는 여러 전략을 사용하여 노드 간에 분산될 수 있다. 사용되는 전략은 세 가지 방식으로 제어된다.<br>
&nbsp;즉, <u>x-queue-master-locator</u> 큐 선언 인수 사용하고, <u>queue-master-locator</u> 정책 키를 설정하거나 또는 <u>the configuration file</u>에서 <u>queue_master_locator</u> 키를 정의한다. 가능한 전략과 설정 방법은 다음과 같다.
  - 최소한의 바인드된 마스터를 호스팅하는 노드를 선택한다. : <u>min_masters</u>
  - 큐가 연결되어 있음을 선언한 클라이언트 노드를 선택한다. : <u>client-local</u>
  - 임의 노드 선택 : <u>random</u><br><br>
- **"node"정책 및 마스터 마이그레이션(“nodes” Policy and Migrating Masters)**<br>
&nbsp;“nodes” 정책을 설정하거나 수정하면 기존 마스터가 새 정책에 나열되지 않은 경우 없어질 수 있다. 메시지 손실을 막기 위해 RabbitMQ는 적어도 하나의 다른 미러가 동기화될 때까지(오랜 시간이더라도) 기존 마스터를 계속 유지합니다. 그러나 일단 동기화가 발생하면 노드가 실패한 것처럼 처리가 진행됩니다. 즉, consumer는 마스터와의 연결이 끊어지고 다시 연결해야 합니다.<br><br>
- **독점 큐 미러링(Mirroring of Exclusive Queues)**<br>
&nbsp;Exclusive 큐는 큐가 선언된 연결이 닫히면 삭제됩니다. 이런 이유 때문에 독점 큐를 미러링하는 것이 유용하지 않습니다. 호스팅하는 노드가 중단되면 연결이 닫히고 큐를 어쨌든 삭제해야 하기 때문입니다. 이러한 이유 때문에 독점 큐는 결코 미러링되지 않습니다.<br><br>
- **클러스터에서 비 미러링된 큐 동작(Non-mirrored Queue Behavior in a Cluster)**<br>
&nbsp;큐의 마스터 노드(큐 마스터를 실행하는 노드)가 사용 가능한 경우, 모든 큐 작업(예 : 선언, 바인딩 및 소비자 관리, 큐에 메시지 라우팅)은 모든 노드에서 수행될 수 있습니다. 클러스터 노드는 작업을 클라이언트에 투명하게 마스터 노드로 라우팅합니다.<br>
&nbsp;큐의 마스터 노드를 사용할 수 없게 되면 미러되지 않은 큐의 동작은 내구성에 따라 달라집니다. 내구성 큐는 노드가 돌아올 때까지 사용할 수 없게 되며 사용할 수 없는 마스터 노드가 있는 영구 큐의 모든 작업은 다음과 같은 서버 로그의 메시지와 함께 실패합니다.<br><br>
- **미러링된 큐 구현 및 의미(Mirrored Queue Implementation and Semantics)**<br>
&nbsp;미러링된 각 큐에는 각각 다른 노드에 하나의 마스터 및 여러 개의 미러가 있습니다. 미러는 발생하는 작업을 마스터와 동일한 순서로 마스터에 적용하여 동일한 상태를 유지합니다. 게시 이외의 모든 작업은 마스터에만 적용되며 마스터는 작업의 효과를 미러에 브로드캐스트합니다. 따라서 미러링된 큐에서 소비하는 클라이언트는 사실 마스터에서 소비됩니다.<br>
&nbsp;미러링이 실패하면 일부 bookkeeping이외에는 할 일이 거의 없습니다. 마스터는 주인으로 남고 어떠한 조치나 실패에 대해 consumer에게 알릴 필요가 없습니다. 미러 실패는 즉시 감지되지 않을 수 있으며, 연결당 흐름 제어 메커니즘의 중단으로 인해 메시지 게시가 지연될 수 있습니다.<br><br>
- **게시자 확인 및 전달(Publisher Confirms and Transactions)**<br>
&nbsp;미러링된 큐는 게시자 확인 및 트랜잭션을 모두 지원합니다. 선택한 시맨틱은 확인 및 트랜잭션의 경우 모두 작업이 큐의 모든 미러에 걸쳐있습니다. 따라서 트랜잭션의 경우 트랜잭션이 큐의 모든 미러에 적용될 때 tx.commit-ok는 클라이언트에만 리턴됩니다. 마찬가지로 게시자 확인의 경우 producer가 모든 미러에서 승인한 경우에만 producer에게 메시지가 확인됩니다.<br><br>
- **흐름 제어(Flow Control)**<br>
&nbsp;RabbitMQ는 신용 기반 알고리즘을 사용하여 메시지 게시 속도를 제한합니다. producer는 큐의 모든 미러에서 크레딧을 받을 때 게시할 수 있습니다. 이 맥락에서의 크레딧은 게시 허가를 의미합니다. 크레딧을 발행하지 못하는 미러는 producer를 지연시킬 수 있으며, producer는 모든 미러에서 크레딧이 발행되거나 나머지 노드가 미러를 클러스터에서 연결 해제한다고 생각할 때까지 차단됩니다. Erlang은 주기적으로 모든 노드에 틱(tick)을 전송하여 이러한 연결 해제를 감지합니다. 틱 간격은 net_ticktime 설정으로 제어할 수 있습니다.<br><br>
- **마스터 실패 및 소비자 취소(Master Failures and Consumer Cancellation)**<br>
&nbsp;미러링된 큐에서 소모하는 클라이언트는 자신이 소비한 큐가 페일 오버되었음을 알고자 할 수 있습니다. 미러링된 큐가 페일 오버되면 어떤 메시지가 어떤 소비자에게 전송되었는지에 대한 정보가 손실되므로 모든 미확인 메시지가 redelivered 플래그가 설정된 상태로 재전송됩니다. consumers는 이것이 일어날 것임을 알고 싶어 할지도 모릅니다.<br>
&nbsp;그렇다면 <u>x-cancel-on-ha-failover</u> 인수를 true로 설정하여 사용할 수 있습니다. 페일오버 시 consumer의 소비가 취소되고 소비자 취소 알림이 전송됩니다. 그런 다음 다시 소비하기 위해 <u>basic.consume</u>을 재발행하는 것은 소비자의 책임입니다.<br><br>
- **비동기 미러(Unsynchronised Mirrors)**<br>
&nbsp;노드는 언제든지 클러스터에 참여할 수 있습니다. 큐의 구성에 따라 노드가 클러스터에 참여할 때 큐는 새 노드에 미러를 추가할 수 있습니다. 이 시점에서 새 미러는 비어 있을 것입니다. 즉, 기존 미러의 내용을 포함하지 않습니다. 이러한 미러는 큐에 게시된 새 메시지를 수신하므로 시간이 지남에 따라 미러된 큐의 꼬리를 정확하게 나타냅니다. 미러된 큐에서 메시지가 유출되면 새 미러에서 메시지가 누락된 큐의 헤드 크기는 결국 미러의 내용이 마스터의 내용과 정확하게 일치할 때까지 축소됩니다. 이 시점에서 미러는 완전히 동기화된 것으로 간주할 수 있지만 큐의 기존 헤드를 소모하는 측면에서 클라이언트의 작업으로 인해 미러가 발생했음을 유의해야 합니다.<br>
&nbsp;새로 추가된 미러는 큐가 명시적으로 동기화되지 않은 경우 미러를 추가하기 전에 있던 큐 내용의 중복성이나 가용성을 추가로 제공하지 않습니다. 명시적 동기화가 발생하는 동안 큐가 응답하지 않게 되므로 메시지가 유출되는 활성 큐가 자연스럽게 동기화되고 비활성 큐만 명시적으로 동기화하도록 허용하는 것이 좋습니다.<br>
&nbsp;자동 큐 미러링을 활성화할 때 관련 큐의 예상 디스크 데이터 세트를 고려해야 합니다. 대규모 데이터 세트(예: 수십 기가 바이트 이상)가 있는 큐는 새로 추가된 미러에 복제해야 하므로 네트워크 대역폭 및 디스크 I/O와 같은 클러스터 리소스에 상당한 부하를 줄 수 있습니다.<br>
&nbsp;미러 상태(동기화 여부) 확인<br>
&nbsp;수동으로 큐 동기화<br>
&nbsp;또는 진행중인 동기화 취소<br>
&nbsp;이 기능은 관리 플러그인을 통해서도 사용할 수 있다.<br><br>
- **실패시 비동기 미러 승격(Promotion of Unsynchronised Mirrors on Failure)**<br>
 &nbsp;기본적으로 큐의 마스터 노드가 실패하거나 피어와의 연결이 끊어지거나 클러스터로부터 제거되는 경우 가장 오래된 미러가 새로운 마스터로 승격된다.
# Access Control(Authentication, Authorisation, Access Control)
- **기본 가상 호스트 및 사용자**<br>
&nbsp;서버가 처음 실행을 시작하고 해당 데이터베이스가 초기화되지 않았거나 삭제된 것을 감지했을 때 다음과 같은 자원(resources)으로 새 데이터베이스를 초기화한다.<br>
  - 가상 호스트 이름 <u>/</u>
  - <u>guest</u>의 기본 비밀번호를 가진 <u>guest</u>라는 사용자, <u>/</u> 가상호스트에 대한 전체 액세스 권한이 부여된다.<br><br>
- **"guest"라는 사용자는 localhost를 통해서만 연결할 수 있다.**<br>
&nbsp;기본적으로 guest 사용자는 원격 호스트에서 연결할 수 없다. loopback 인터페이스(즉, localhost)를 통해서만 연결할 수 있다. 이것은 AMQP 0-9-1 및 플러그인을 통해 활성화된 다른 모든 프로토콜에 모두 적용된다.<br>
&nbsp;프로덕션 시스템에서 이 문제를 해결하는 데 권장되는 방법은 필요한 가상 호스트에 액세스할 수 있는 권한을 가진 새로운 사용자 또는 사용자 집합을 만드는 것이다. 이는 CLI 도구, HTTP API 또는 definitions import(정의 가져오기)를 사용하여 수행할 수 있다.<br>
&nbsp;이것은 configuration files의 loopback_users 항목을 통해 구성된다. guest 사용자가 원격 호스트에서 연결할 수 있게 하려면 loopback_users 구성을 none으로 설정해야 한다.<br><br>
- **권한 작동 방식(How Permissions Work)**<br>
&nbsp;RabbitMQ 클라이언트는 서버에 대한 연결을 설정하고 인증할 때, 작동하려는 가상 호스트를 지정한다. 이 단계에서 서버는 사용자가 가상 호스트에 액세스하기 위한 권한이 있는지 여부를 확인하고 그렇지 않으면 연결 시도를 거부하여 첫 번째 수준의 액세스 제어가 진행된다.<br>
&nbsp;자원(즉, 교환 및 큐)은 특정 가상 호스트 내부의 명명된 엔티티로 동일한 이름은 각 가상 호스트의 다른 리소스를 의미한다. 특정 작동이 자원에 대해 수행될 경우 두 번째 레벨의 액세스 제어가 진행된다.<br>
&nbsp;RabbitMQ는 자원에 대한 구성, 쓰기 및 읽기 작업을 구분한다. 구성작업은 자원을 만들거나 파괴하거나, 해당 자원의 행동을 변경한다. 쓰기 작업은 자원에 메시지를 넣고 읽기 작업은 자원에서 메시지를 검색한다. 자원에 대한 작업을 수행하려면 사용자는 적절한 권한을 부여받아야 한다.<br><br>
- **사용자 태그 및 관리 UI 액세스(User Tags and Management UI Access)**<br>
&nbsp;사용자는 태그를 가질 수 있다. 현재 관리 UI 액세스만 사용자 태그에 의해 제어된다. 태그는 rabbitmqctl을 사용하여 관리되며 새로 생성된 사용자에게는 기본적으로 태그가 설정되어 있지 않다.<br><br>
- **주제 승인(Topic Authorisation)**<br>
&nbsp;버전 3.7.0부터 RabbitMQ는 topic 교환에 대한 topic authorisation을 지원한다. topic 교환에 게시된 메시지의 라우팅 키는 게시 승인이 적용될 때 고려된다. Topic authorisation 은 STOMP와 MQTT 같은 프로토콜을 대상으로 한다.<br>
&nbsp;topic authorisation은 producer를 위한 기존의 checks 위에 추가 레이어이다. topic 유형 exchange에 메시지를 게시하는 것은 <u>basic.publish</u>와 라우팅 키 checks를 모두 거치게 된다. <br>
&nbsp;topic authorisation은 topic consumer에게도 적용될 수 있다. 프로토콜이 따라 다르게 작동한다는 점에 유의해야 한다. Topic authorisation의 개념은 MQTT 및 STOMP와 같은 topic 지향 프로토콜에 대해서만 의미가 있다. 예를 들어, AMQP 0-9-1의 경우 consumer는 큐로부터 소비하므로 표준 자원 권한이 적용된다. 또한 AMQP 0-9-1에서 AMQP 0-9-1 topic 교환과 큐/교환 간의 바인딩 라우팅 키는 구성된 topic 권한과 비교하여 확인된다.<br>
&nbsp;기본 승인 백엔드를 사용할 경우 topic 교환에 게시하거나 topic으로부터 소비하는 것은 topic 사용 권한이 정의되지 않은 경우 항상 승인된다. 이 승인 백엔드에서는 Topic authorisation이 선택 사항이므로 교환을 허용할 필요가 없다. 따라서 Topic authorisation을 사용하려면 하나 이상의 교환에 대한 topic 권한을 선택하고 정의해야한다.<br>
&nbsp;내부(기본) 권한 부여 백엔드는 권한 패턴의 변수 확장을 지원한다. 지원되는 변수는 username , vhost 및 client_id이다.<br><br>
- **백엔드 결합(Combining Backends)**<br>
&nbsp;<u>auth_backends</u> 구성 키를 사용하여 <u>authn</u> 또는 <u>authz</u>에 다중 백엔드를 사용할 수 있다. 여러 개의 인증 백엔드를 사용할 경우 체인의 백엔드에 의해 반환되는 첫 번째 긍정적인 결과는 최종 결과로 간주된다.<br>
&nbsp;<u>rabbit_auth_backend_internal</u>의 <u>internal</u> 별칭을 사용한다. 별칭은 다음과 같은 별칭을 사용할 수 있다.<br>
  - <u>internal</u> for <u>rabbit_auth_backend_internal</u>
  - <u>ldap</u> for <u>rabbit_auth_backend_ldap</u>
  - <u>http</u> for <u>rabbit_auth_backend_http</u>
  - <u>amqp</u> for <u>rabbit_auth_backend_amqp</u>
  - <u>dummy</u> for <u>rabbit_auth_backend_dummy</u><br><br>

 &nbsp;인증과 권한 부여를 위해 LDAP 백엔드를 사용하도록 RabbitMQ를 구성한 것이다. 내부 데이터베이스는 참조하지 않는다.<br>

 &nbsp;이렇게 하면 LDAP를 먼저 확인한 다음 LDAP를 통해 사용자를 인증할 수 없는 경우 내부 데이터베이스로 다시 넘어간다.

# Authentication Mechanisms
&nbsp;RabbitMQ는 다양한 SASL 인증 메커니즘을 지원한다. 서버에는 <u>PLAIN</u>, <u>AMQPLAIN</u> 및 <u>RABBIT-CR-DEMO</u>의 세 가지 메커니즘이 내장되어 있으며, 플러그인에서 <u>rabbit_auth_mechanism</u> 동작을 구현하여 고유한 인증 메커니즘을 구현할 수도 있다.<br>
- **내장 매커니즘(Built-in Mechanisms)**<br>
&nbsp;내장 매커니즘은 다음과 같다.<br>
  - <u>PLAIN</u> : SASL PLAIN 인증. RabbitMQ 서버 및 클라이언트에서 기본적으로 활성화되며 대부분의 다른 클라이언트의 기본값이다.<br>
  - <u>AMQPLAIN</u> : PLAIN의 비표준 버전은 이전 버전과의 호환성을 유지한다. RabbitMQ 서버에서 기본적으로 활성화된다.<br>
  - <u>EXTERNAL</u> : 인증은 x509 인증서 피어 확인, 클라이언트 IP 주소 범위 등과 같은 대역 외 매커니즘을 사용하여 수행된다. 이러한 매커니즘은 일반적으로 RabbitMQ 플러그인에 의해 제공된다.<br>
  - <u>RABBIT-CR-DEMO</u> : challenge-response 인증을 나타내는 비표준 메커니즘. 이 메커니즘은 <u>PLAIN</u>과 동등한 보안 기능을 제공하며 RabbitMQ 서버에서 기본적으로 활성화 되지 않는다.<br><br>
- **서버 매커니즘 구성(Mechanism Configuration in the Server)**<br>
&nbsp;<u>rabbit</u> 응용 프로그램의 구성 변수 <u>auth_mechanisms</u>는 클라이언트를 연결하는 데 설치된 메커니즘을 결정한다. 이 변수는 기본적으로 ['PLAIN', 'AMQPLAIN']과 같이 메커니즘 이름에 해당하는 원자 목록이어야 한다. 서버 측 목록은 특정 순서로 간주되지 않는다.<br><br>
- **클라이언트 매커니즘 구성(Mechanism Configuration in the Client)**<br>
 - **Java의 매커니즘 구성(Mechanism Configuration in Java)**<br>
 &nbsp;Java 클라이언트는 <u>javax.security.sasl</u> 패키지를 기본적으로 사용하지 않는다. 이는 Oracle 이외의 JDK에서 예측할 수 없으며 완전히 Android에서 누락되어 있기 때문이다. <u>SaslConfig</u> 인터페이스로 구성된 RabbitMQ 관련 SASL 구현이 있다. 클래스 <u>DefaultSaslConfig</u>는 SASL 구성을 일반적인 경우보다 편리하게 하기 위해 제공된다. <u>JDKSaslConfig</u>클래스는 <u>javax.security.sasl</u>에의 브릿지로서 기능하기 위해서 제공되고 있다.<br>
 &nbsp;<u>ConnectionFactory.getSaslConfig()</u> 및 <u>ConnectionFactory.setSaslConfig(SaslConfig)</u>는 인증 매커니즘과 상호작용하기 위한 주요 방법이다.
 - **Erlang의 매커니즘 구성(Mechanism Configuration in Erlang)**<br>
 &nbsp;Erlang 클라이언트는<u>amqp_auth_mechanisms</u> 모듈에 자체 SASL 메커니즘 구현을 제공한다. <u>#amqp_params{}</u> 레코드는 네트워크 연결의 우선순위에 따라 인증 기능의 목록을 제공할 수 있다.
 - **.NET의 매커니즘 구성(Mechanism Configuration in .NET)**<br>
 &nbsp;.Net 클라이언트는 <u>AuthMechanism</u> 및 <u>AuthMechanismFactory</u> 인터페이스를 기반으로 자체 SASL 메커니즘 구현을 제공한다. <u>ConnectionFactory.AuthMechanisms</u> 속성은 인증 메커니즘 팩토리의 우선순위 목록이다.<br><br>
- **권한 부여 문제 해결(Troubleshooting Authorisation)**<br>
&nbsp;rabbitmqctl list_permissions을 사용하여 주어진 가상 호스트에서 사용자의 권한을 검사할 수 있다.

# Lazy Queues
&nbsp;RabbitMQ 3.6.0부터 브로커는 Lazy Queues 개념을 사용한다. 지연 큐의 주요 목표 중 하나는 매우 긴 큐(수백만개의 메시지)를 지원할 수 있다는 것이다. 큐는 여러 가지 이유로 길어질 수 있다.

 - 소비자가 오프라인 상태/ 파손/ 정비 중단 상태
 - 갑작스런 메시지 수신 급증, 생산자가 소비자를 앞지를 때
 - 소비자가 정상보다 느릴 때<br>

&nbsp;기본적으로 큐는 메시지가 RabbitMQ에 게시될 때 채워지는 메시지의 메모리 내 캐시를 유지한다. 이 캐시의 개념은 consumer에게 가능한 한 빨리 메시지를 전달할 수 있다는 것이다. 영구 메시지는 브로커에 들어가 동시에 RAM에 보관될 때 디스크에 기록될 수 있다는 점에 유의해야 한다.<br>
&nbsp;브로커가 메모리를 확보해야 한다고 생각할 때마다 이 캐시의 메시지는 디스크로 페이징된다. 디스크에 메시지를 일괄적으로 페이징하는 데 시간이 걸리고 큐 프로세스가 차단되므로 페이징하는 동안 새 메시지를 수신할 수 없다. 비록 최근 버전의 RabbitMQ가 페이징 알고리즘을 개선시켰지만, 이 상황은 여전히 페이징 알고리즘을 페이징해야 할 수 있는 큐에 수백만 개의 메시지가 있는 사용 사례에 적합하지 않다.<br>
&nbsp;그리고 지연 큐들은 메시지를 가능한 빨리 디스크로 옮기려고 시도한다. 이는 대부분의 경우 정상 작동 시 RAM에 보관되는 메시지 수가 상당히 적다는 것을 의미한다. 이는 디스크 I/O 증가시키는 비용에서 발생한다.<br>
- **지연 큐 만들기(Making a Queue Lazy)**<br>
&nbsp;큐는 <u>default</u> 모드 도는 <u>lazy</u> 모드로 실행되도록 만들 수 있다.<br>
  - <u>queue.declare</u> 인수를 통해 모드 설정
  - 큐 정책 사용<br>

 &nbsp;policy 및 queue 인수가 모두 큐 모드를 지정하면 queue 인수는 policy 값보다 우선순위를 갖는다.<br>
- **선언시 인수 사용(Using Arguments at the Time of Declaration)**<br>
&nbsp;큐 모드는 <u>x-queue-mode</u> 큐 선언 인수에 원하는 모드를 지정하는 문자열을 제공하여 설정할 수 있다.<br>
&nbsp;유효한 모드는

  - <u>"default"</u>
  - <u>"lazy"</u><br>

&nbsp;선언하는 동안 모드를 지정하지 않으면 “default”로 간주된다.<br>
&nbsp;이 예는 큐 모드가 “lazy”로 설정된 큐를 선언하는 예이다.<br>
- **Policy 사용(Using a policy)**<br>
&nbsp;Policy를 사용하여 큐 모드를 지정하려면 키 <u>queue-mode</u>를 정책 정의에 추가해야 한다.<br>
&nbsp;다음 표는 lazy-queue라 불리는 큐를 lazy 모드로 작동시킨다.<br>

|rabbitmqctl|rabbitmqctl set_policy Lazy "^lazy-queue$" '{"queue-mode":"lazy"}' --apply-to queues|
|---|---|
|**rabbitmqctl(Windows)**|**rabbitmqctl set_policy Lazy "^lazy-queue$" "{""queue-mode"":""lazy""}" --apply-to queues**|
- **런타임에 큐 모드 변경(Changing Queue Mode at Runtime)**<br>
&nbsp;큐 모드가 정책을 통해 구성된 경우 큐를 삭제하고 다시 선언할 필요 없이 런타임에서 변경할 수 있다. “lazy-queue”라는 이름의 큐를 기본(non-lazy) 모드로 사용하려면 일치하는 정책을 업데이트하여 다른 <u>queue-mode</u>를 지정하면 된다.

|rabbitmqctl|rabbitmqctl set_policy Lazy "^lazy-queue$" '{"queue-mode":"default"}' --apply-to queues|
|---|---|
|**rabbitmqctl(Windows)**|**rabbitmqctl set_policy Lazy "^lazy-queue$" "{""queue-mode"":""default""}" --apply-to queues**|
- **지연 큐의 성능 고려 사항(Performance Considerations for Lazy Queues)**<br>
  - 디스크 사용률<br>
 &nbsp;지연 큐는 producer가 메시지를 일시적으로 게시한 경우에도 가능한 한 빨리 메시지를 디스크로 이동시킬 것이다. 그러면 일반적으로 I/O 사용률이 높아진다.<br>
 &nbsp;일반적인(regular) 큐는 메시지를 더 오랫동안 메모리에 보관한다. 이로 인해 더 많은 데이터를 디스크에 한 번에 기록해야 하기 때문에 디스크 I/O가 지연되는 결과를 초래한다.<br><br>
- **RAM 사용률(RAM Utilization)**<br>
&nbsp;모든 사용 사례에서 정확한 숫자를 제공하는 것은 불가능하지만, 이는 일반 및 지연 큐 간의 RAM 사용량의 차이를 보여주는 단순한 테스트이다.<br>

|Number of messages|Message body size|Message type|Producers|Consumers|
|---|---|---|---|---|
|1,000,000|1,000bytes|persistent|1|0|
&nbsp;위의 메시지를 처리한 후 기본 및 지연 큐의 RAM 사용률<br>

|Queue mode|Queue process memory|Messages in memory|Memory used by messages|Node memory|
|---|---|---|---|---|
|<u>default</u>|257MB|386,307|368MB|734MB|
|<u>lazy</u>|159KB|0|0|117MB|
&nbsp;두 큐 모두 1,000,000개의 메시지를 유지하고 1.2GB의 디스크 공간을 사용한 것을 확인할 수 있다.<br>
- **런타임에 큐 모드 전환(Switching Queue Mode at Runtime)**<br>
&nbsp;<u>default</u> 큐를 <u>lazy</u> 큐로 변환할 때 큐에서 메시지를 디스크로 이동해야 할 때와 같은 성능 영향을 받게 된다.<br>
&nbsp;일반 모드에서 지연 모드로 변환하는 동안 큐는 RAM에 저장된 모든 메시지를 먼저 디스크에 표시한다. 해당 작업이 진행되는 동안 게시하는 채널로부터 더 이상 메시지를 수신하지 않는다. 초기 페이지아웃이 완료되면 큐에서 게시, 확인 및 다른 명령을 수신하기 시작한다.<br>
&nbsp;큐가 지연모드에서 기본 모드로 변경되면 서버가 다시 시작된 후 큐가 복구될 때와 동일한 프로세스가 수행된다.<br><br>
- **혼합된 메시지 크기의 지연 큐(Lazy Queues with Mixed Message Sizes)**<br>
&nbsp;처음 10,000개의 메시지의 모든 메시지가 <u>queue_index_embed_msgs_below</u> 아래 값이고 나머지는 이 값보다 큰 경우 노드 시작 시 처음 10,000개만 메모리에 로드된다.<br><br>
- **인터리브 메시지의 지연 큐(Lazy Queues with Interleaved Message)**<br>
&nbsp;주어진 인터리브된 메시지 크기 :<br>

|Position in queue|Message size in bytes|
|---|---|
|1|5,000|
|2|100|
|3|5,000|
|4|200|
|...|...|
|79|4,000|
|80|5,000|

&nbsp;<u>queue_index_embed_msgs_below</u> 아래 값의 처음 20개 메시지만 노드 시작 시 메모리로 로드된다. 이 시나리오에서 메시지는 21KB 의 시스템 메모리를 사용하고 큐 프로세스는 다른 32KB의 시스템 메모리를 사용한다. 큐 프로세스가 시작을 완료하는데 필요한 전체 시스템 메모리는 53KB이다.

# Firehose(Message Tracing)
&nbsp;RabbitMQ에는 “Firehose” 기능이 있다. 관리자는 이를 통해 게시 및 delivery 통지되어야 하는 교환을 (노드별로, 호스트별로) 설정할 수 있다.<br>
- **Firehose 활성화**<br>
1.어떤 노드와 어떤 가상 호스트를 활성화할지 결정한다.<br>
2.선택한 가상 호스트에서 큐를 생성하고 교환 amq.rabbitmq.trace topic exchange에 바인드한 후 consuming을 시작한다.<br>
3.<u>rabbitmqctl trace_on</u>을 실행한다.<br><br>
- **Firehose 비활성화**<br>
1.<u>rabbitmqctl trace_off</u>를 실행한다.<br>
2.Firehose가 사용하는 큐를 정리한다.<br>
&nbsp;Firehose 상태는 지속되지 않으며 서버 시작 시 기본 값은 off가 된다.<br><br>
- **Firehose 알림 형식**<br>
&nbsp; Firehose는 주제 교환에 메시지를 게시하는 <u>amq.rabbitmq.trace</u>와<br>
  - 브로커를 입력하는 메시지의 경우 <u>"publish.exchangename"</u> 또는 브로커를 나가는 메시지의 경우 <u>"delivery.queuename"</u> 어느 한쪽의 라우팅 키
  - original 메시지의 본체에 해당하는 본문
  - original 메시지에 대한 metadata를 포함하는 헤더 : <br>


 |Header|Type|Description|
 |---|---|---|
 |exchange_name|longstr|메시지가 게시된 exchange의 이름|
 |routing_keys|array|라우팅 키와 CC 및 BCC 헤더의 내용|
 |properties|table|내용 속성(content properties)|
 |node|longstr|추적 메시지가 생성된 Erlang 노드|
 |redelivered|signedint|메시지에 재전달 플래크가 설정되었는지|
