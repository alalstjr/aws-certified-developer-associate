- [IAM: Users & Groups](#IAM:-Users-&-Groups)
  - [IAM User 만들기](#IAM-User-만들기)
  - [정책](#정책)
  - [Password Policy](#Password-Policy)
    - [가상 MFA 장치](#가상-MFA-장치)
    - [U2F 보안 키](#U2F-보안-키)
  - [MFA 설정](#MFA-설정)
  - [AWS 액세스 키, CLI 및 SDK](#AWS-액세스-키,-CLI-및-SDK)
    - [CLI 란 무엇일까?](#CLI-란-무엇일까?)
    - [SDK 란 무엇일까?](#SDK-란-무엇일까?)
  - [CLI Install](#CLI-Install)
  - [AWS CLI 연습](#AWS-CLI-연습)
  - [역할](#역할)
    - [IAM 액세스 관리자](#IAM-액세스-관리자)
    - [IAM 모범 사례](#IAM-모범-사례)
- [EC2 기초](#EC2-기초)
  - [인스턴스 유형](#인스턴스-유형)
    - [범용의 인스턴스](#범용의-인스턴스)
    - [컴퓨팅 최적화 인스턴스](#컴퓨팅-최적화-인스턴스)
    - [메모리 최적화의 인스턴스](#메모리-최적화의-인스턴스)
    - [스토리지 최적화 인스턴스](#스토리지-최적화-인스턴스)
  - [보안 그룹](#보안-그룹)
    - [SSH](#SSH)
  - [온-디맨드 인스턴스](#온-디맨드-인스턴스)
  - [예약 인스턴스](#예약-인스턴스)
    - [정기 예약 인스턴스](#정기-예약-인스턴스)
    - [스팟 인스턴스](#스팟-인스턴스)
    - [전용 호스트](#전용-호스트)
    - [전용 인스턴스](#전용-인스턴스)
- [EBS 볼륨](#EBS-볼륨)
  - [스냅샷](#스냅샷)
  - [AMI](#AMI)
  - [인스턴스 스토어](#인스턴스-스토어)
  - [EBS 볼륨](#EBS-볼륨)
    - [IOPS](#IOPS)
    - [스토리지](#스토리지)
    - [st1과 sc1](#st1과-sc1)
    - [EBS 다중 연결](#EBS-다중-연결)
  - [EFS](#EFS)
    - [성능 모드](#성능-모드)
    - [처리량 모드](#처리량-모드)
  - [EBS vs EFS](#EBS-vs-EFS)
  - [EBS](#EBS)
  - [EFS](#EFS)

# IAM: Users & Groups

IAM 은 `Identity and Access Management` 의 약자로  
IAM 에서는 사용자를 생성하고  
그룹에 배치하기 때문에 글로벌 서비스에 해당됩니다  

AWS에서는 모든 사용자에게 모든 것을 허용하지 않습니다  
그러면 엉망이 될 겁니다  
새로운 사용자가 너무 많은 서비스를 실행하여 큰 비용이  
발생하거나, 보안 문제를 야기할 수 있기 때문이죠  
따라서 AWS에서는 최소 권한의 원칙을 적용합니다  
즉 사용자가 꼭 필요로 하는 것 이상의 권한을 주지 않는 것입니다.  

## IAM User 만들기

처음으로 할 일은 IAM 사용자를 생성하는 것이죠

https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users

Users 항목으로 이동해 Add users 를 선택하겠습니다  
보시는 것처럼 상단의 계정 이름을 클릭해 보시면  
현재 루트 계정을 사용 중이죠  
루트 사용자는 계정에 대한 모든 권한을 가지고 있습니다  
그렇기 때문에 위험한 계정이 될 수도 있죠  
따라서 별도의 관리자 계정을 만드는 것이 좋습니다  
루트 계정은 정말로 반드시 꼭 필요할 때만 사용할 겁니다  

그룹 이름은 admin으로 하겠습니다  
admin 그룹에 배치된 사용자는  
이 그룹에 부여된 권한을 승계하게 됩니다  
그리고 권한은 정책을 통해 정의되죠  
admin 그룹에 연결할 정책은 AdministratorAccess 입니다
이 정책은 admin 그룹에 속한 모든 사용자가
계정의 관리자 역할을 하도록 허용할 겁니다  

Next: Tags를 클릭합니다  
AWS에서는 어디에서든 태그를 찾을 수 있는데요  
사용자의 접근을 추적, 조직, 제어할 수 있도록 도와주는 정보입니다    
이 강의에서는 굳이 여기저기에 태그를 만들지 않을 겁니다  
다만 사용자를 위한 태그 생성 방법을 보여드리죠  
그 특정 사용자에 대해 단순히 정보를 추가하는 것입니다  
예를 들어, 이 사용자가 속한 부서는 엔지니어링이라고 표시할 수 있죠

## 정책

~~~
{
    "Version": "2012-10-17",
    "Id": "S3-Account-Permissions"
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": ["arn:aws:iam::123123123:root"]
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
            ],
            "Resource": ["arn:aws:s3:::mybuckey/*"]
        }
    ]
}
~~~

IAM 정책 구조 요소는 버전 숫자를 포함하는데요  
보통은 2012-10-17로 정책 언어 버전입니다  
정책을 식별하는 `ID` 도 있는데 선택 사항이고요  
문장들도 구성 요소입니다  
문장은 하나일 수도 여러 개일 수도 있는데  
문장에는 아주 중요한 부분들이 있죠  
먼저 `Sid` 는 문장 ID로 문장의 식별자이고 역시 선택 사항입니다  
오른쪽에 보시면 1번이라고 나와있죠  
정책에서 `Effect` 는 문장이 특정 API에 접근하는 걸 허용할지 거부할지에 대한 내용입니다  
`Principal` 은 특정 정책이 적용될 사용자, 계정, 혹은 역할로 구성됩니다
이 예시에선 AWS 계정의 루트 계정에 적용이 되죠  
`Action` 은 `Effect` 에 기반해 허용 및 거부되는 API 호출의 목록입니다  
그리고 `Resource` 는 적용될 `Action` 의 리소스의 목록으로  
이 예시에선 버킷이지만 다른 것들도 될 수도 있겠죠

## Password Policy

이 그룹과 사용자들의 정보가 침해당하지 않도록 보호해야겠죠  
다요소 인증, `MFA` 입니다  
`MFA` 는 여러분이 알고 있는 비밀번호와  
여러분이 가지고 있는 보안 장치를 함께 사용하는 것입니다  
`MFA` 의 장점은 사용자가 해킹을 당해 비밀번호가  
누출된 상황이라고 해도 해커에게는 로그인을 위해 휴대전화 등  
사용자 소유의 물리적 장치가 추가로 필요해질 테니  
계정이 침해당하지 않는다는 점입니다  

AWS 에서의 `MFA 장치 옵션` 으로는 어떤 것들이 있을까요?

### 가상 MFA 장치

`Google Authenticator` 를 사용할 수 있는데 하나의 휴대전화에서만 사용이 가능하죠
`Authy` 는 여러 장치에서 사용이 가능합니다  
장치의 개수가 다를 뿐 작동 방식은 동일합니다  
개인적으로 저는 `Authy` 를 사용하는데 컴퓨터와  
휴대전화에서 같이 사용할 수 있기 때문이죠  
`Authy` 는 하나의 장치에서도 토큰을 여러 개 지원합니다  
즉, 가상 `MFA` 장치를 사용하면  
루트 계정, IAM 사용자 또 다른 계정, 그리고  
또 다른 IAM 사용자가 지원되는 식으로 가상 MFA 장치에  
원하는 수만큼의 계정 및 사용자 등록이 가능합니다  

### U2F 보안 키

이는 물리적 장치로 예를 들어  
Yubico 사의 YubiKey 가 있죠 Yubico 는 AWS 의 제3자 회사로  
AWS 제공 장치가 아니라 제3자 회사의 장치입니다  
이렇게 `물리적 장치를 사용하면 전자 열쇠`에 달고 다닐 수 있으니  
사용이 상당히 편리할 수 있겠죠  
YubiKey 는 하나의 보안 키에서 여러 루트 계정과 IAM 사용자를  
지원하기 때문에 하나의 키로도 충분합니다

### 하드웨어 키 팝 MFA 장치

역시 AWS 의 제3자 회사인 Gemalto 의 제품입니다  
만약 미국 정부의 클라우드인 AWS GovCloud 를  
사용하시는 경우라면, MFA 숫자를 실시간으로 보여주는 특수한 키 팝(작은 물건)이 필요한데요  
역시 SurePassID 라는 제3자 회사가 제공하고 있죠  

## MFA 설정

왼쪽의 `Account settings (계정 설정)` 에 들어가서  
`Change password policy (암호 정책 변경)` 를 누르면  
여기서 비밀번호 정책 적용이 가능하죠    
비밀번호 최소 길이 지정이 가능하고  
적어도 하나의 대문자 및 소문자 숫자를 포함하도록 합니다  

오른쪽에 있는 계정 이름을 클릭하고  
`My Security Credential(보안 자격 증명)` 로 들어갑니다  
https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/security_credentials

`Multi-factor authentication (MFA) - 멀티 팩터 인증` 를 클릭하고  
휴대전화를 사용할 거니까 Virtual MFA device 를 선택하겠습니다

## AWS 액세스 키, CLI 및 SDK

Create access key 버튼으로 액세스 키를 생성하면  
곧 다운로드 권한이 주어질 겁니다  
직장에서 보안 문제를 피하기 위해서는  
절대 액세스 키를 공유하지 마세요 여러분만 알고 계셔야 합니다  

### CLI 란 무엇일까?

`CLI` 는 명령줄 인터페이스를 의미하며  
`AWS CLI` 는 명령줄 셸에서 명령어를 사용하여  
AWS 서비스들과 상호작용할 수 있도록 해 주는 도구입니다

`CLI` 를 사용하면 `AWS 서비스의 공용 API 로 직접 액세스가 가능`합니다
그리고 `CLI` 를 통해 리소스를 관리하는 스크립트를 개발해  
일부 작업을 자동화할 수 있죠  
`CLI` 는 오픈 소스로, GitHub 에서 모든 소스 코드를 찾으실 수 있으며  
AWS 관리 콘솔 대신 사용되기도 합니다  

### SDK 란 무엇일까?

`SDK 는 소프트웨어 개발 키트입니다`  
특정 언어로 된 라이브러리의 집합인데요  
따라서 프로그래밍 언어에 따라 개별 `SDK` 가 존재합니다  
이 방식을 사용해서도 역시 AWS 서비스나 API 에  
프로그래밍을 위한 액세스가 가능하도록 해줍니다  
하지만 `SDK 는 터미널 내에서는 사용하는 것이 아니라  
코딩을 통해 애플리케이션 내에 심어 두어야 하는 겁니다`  
애플리케이션 내에 자체적으로 AWS SDK 가 있는 거죠  
다양한 프로그래밍 언어를 지원하죠  
JavaScript Python, PHP, .NET  
Ruby, Java, Go Node.js, C++ 등을 지원합니다

## CLI Install

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html

## AWS CLI 연습

사용자 -> 생성된 IAM 유저 계정 클릭   
IAM 사용자인 상태이며 이제 Security credentials (보안 자격 증명) 로 이동합니다
액세스 키는 CLI, 즉 명령줄 인터페이스를 사용할 때 굉장히 유용합니다  
혹은 AWS 에 프로그래밍 언어의 구현을 위해 SDK 를 사용할 때도 유용하죠  
Create access key 를 클릭합시다 액세스 키는 기밀입니다  
오직 생성 시에만 표시 및 다운로드가 가능하죠  

먼저 AWS CLI 부터 구성해야 하는데요  
aws configure 라고 입력하겠습니다  
그러면 액세스 키 ID를 입력하라고 나오네요  
생성된 액세스 키를 붙여 넣어 입력한 후 Enter 를 누르고요  
이번엔 암호 액세스 키를 입력하라는 안내가 나옵니다  
역시 이렇게 입력해 주겠습니다  
기본 리전 이름은 가까운 리전을 말하는 겁니다  
강의 전체가 eu-west-1에서 진행될 테니 eu-west-1로 지정해 주겠습니다  
여러분들은 각자의 리전을 선택해 입력하시면 됩니다  

그럼 어떻게 작동하는지도 한 번 봐야겠죠
aws iam list-users 를 입력 후 Enter 를 누르면
제 계정의 모든 사용자를 목록으로 보여 줄 겁니다

## AWS 클라우드쉘

`CloudShell` 은 화면 우측 상단의 이 아이콘입니다  
사용 할 수 있는 리전에 있는지 확인하세요 모든 리전에서 가능한 건 아니거든요  
AWS CloudShell FAQs에 가시면 사용이 불가능한 리전이 나와 있습니다

## 역할

EC2 인스턴스는 AWS에서 어떤 작업을 수행하려고 할 수 있습니다  
그러기 위해서는 EC2 인스턴스에 권한을 부여해야 합니다  
이를 위해 IAM 역할을 만들어 이들을 하나의 개체로 만듭니다  
EC2 인스턴스가 AWS에 있는 어떤 정보에 접근하려고 할 때  
IAM 역할을 사용하게 될 것입니다  
만약 IAM 역할의 권한을 올바르게 부여한 경우  
하려고 하는 호출에 접근하게 될 것입니다  

역할은 여러분이 신뢰하는 개체에 권한을 부여하기 위해 사용됩니다

### IAM 액세스 관리자

이것은 사용자 수준에서 가능합니다 액세스 관리자는  
사용자에게 부여된 서비스의 권한과  
해당 서비스에 마지막으로 액세스한 시간이 보입니다  
최소권한의 원칙에 따랐을 때 매우 도움 되는 정보입니다  
해당 도구를 사용하여 어떤 권한이 사용되지 않는지 볼 수 있고  
따라서 사용자의 권한을 줄여 최소권한의 원칙을 지킬 수 있습니다

왼쪽 하단에 Credential report 가 있고 Download Report 를 클릭하여  
보고서를 다운로드합니다 CSV 파일이 다운로드될 거예요  
사용자가 언제 생성되었는지, 비밀번호가 활성화되었는지,  
비밀번호를 마지막으로 언제 사용했는지, 마지막으로 언제 변경되었는지,  
비밀번호 변경 주기를 활성화한 경우 다음 주기는 언제인지  
MFA 가 활성화되었는지 출력됩니다.

다음으로는 IAM 액세스 관리자를 이야기하겠습니다  
Users를 클릭합니다  
오른쪽에 Access Advisor 가 있습니다  
마지막에 사용된 서비스를 보여줄 것입니다  
보통 최근 4시간 동안의 활동 내역이 보입니다  
만약 보이지 않는다면 4시간이 지났기 때문이죠

### IAM 모범 사례

루트 계정은 AWS 계정을 설정할 때를 제외하고 사용하지 마세요  
사용자를 그룹에 넣어 해당 그룹에 권한을 부여할 수 있어요  
따라서 그룹 수준에서 보안을 관리할 수 있습니다  
또한 비밀번호 정책을 강력하게 만들어야 합니다  
다요소 인증(MFA)을 사용한다.  
AWS 서비스에 권한을 부여할 때마다 역할을 만들고 사용해야 합니다  
가상 서버인 EC2 인스턴스를 포함해서요  
AWS를 프로그래밍할 경우, 즉, CLI나 SDK를 사용할 경우  
반드시 액세스 키를 만들어야 합니다 액세스 키는 비밀번호와 같습니다

# EC2 기초

이것은 일래스틱 컴퓨트 클라우드의 약자로 AWS에서 제공하는 서비스형 인프라스트럭처입니다  
즉, EC2는 하나의 서비스가 아닙니다  
높은 수준에서 보면 많은 것을 포함하고 있습니다  
가상 머신을 EC2에서 임대할 수 있는데 이를 EC2 인스턴스라고 합니다  

데이터를 가상 드라이브 또는 EBS 볼륨에 저장할 수 있고  
일래스틱 로드 밸런서로 로드는 분산시킬 수 있습니다  
또 오토 스케일링 그룹(ASG)을 통해 서비스를 확장할 수 있습니다  

즉, EC2 사용자 데이터 스크립트를 사용하여 인스턴스를 부트스트래핑할 수 있습니다  
부트스트래핑이 무슨 뜻일까요?  
이는 머신이 작동될 때 명령을 시작하는 것을 말합니다  
스크립트는 처음 시작할 때 한 번만 실행되고 다시 실행되지 않습니다

## 인스턴스 유형

m5.2xlarge 라는 유형의 인스턴스를 예로 들어 보겠습니다  
`m 을 인스턴스 클래스`로 부르겠습니다  
그리고 이 인스턴스 클래스는 범용의 인스턴스입니다  
`5는 인스턴스의 세대`를 뜻합니다  
즉, AWS 가 하드웨어를 계속 개선해서 새로운 세대의 하드웨어를 출시하고  
m5 이후에 m 유형의 인스턴스 클래스를 개선하면  
m6가 되는 것입니다 2xlarge 는 인스턴스 클래스 내에서 크기를 나타냅니다  
small 로 시작해 large, 2xlarge 4xlarge 등의 크기가 있죠  
인스턴스의 크기를 나타내며 크기가 클수록 인스턴스에  
더 많은 메모리와 CPU 를 가지게 됩니다

### 범용의 인스턴스

먼저, 범용의 인스턴스는 웹 서버나 코드 저장소와 같은 다양한 작업에 적합합니다  
컴퓨팅, 메모리, 네트워킹 간의 균형도 잘 맞습니다

### 컴퓨팅 최적화 인스턴스

이제 컴퓨팅 최적화 인스턴스는 컴퓨터 집약적인 작업에 최적화된 인스턴스입니다  
그러면 고성능 프로세서는 어디에 사용할까요?  
일부 데이터의 일괄 처리에 사용하거나  
미디어 트랜스코딩 작업 시 혹은 고성능 웹 서버가 필요하거나  
고성능 컴퓨팅이라는 HPC 작업을 할 때 그리고 머신 러닝이나  
전용 게임 서버가 있을 때 사용합니다  
모두 훌륭한 CPU와 컴퓨팅을 요구하는 작업이며  
EC2 인스턴스는 이런 특성을 가지고 있습니다  
그리고 컴퓨터 최적화의 모든 인스턴스는  
C로 시작하는 이름을 가지고 있습니다  
C5, C6 등이죠  

### 메모리 최적화의 인스턴스

다음으로 메모리 최적화의 인스턴스를 살펴보겠습니다  
이 유형의 인스턴스는 메모리에서 대규모 데이터셋을  
처리하는 유형의 작업에 빠른 성능을 제공합니다  
메모리는 RAM을 뜻하고 사용 사례를 살펴보면  
대부분 인 메모리 데이터베이스가 되는  
고성능의 관계형 또는 비관계형의 데이터베이스에 사용하고  
일래스틱 캐시를 예로 들 수 있는  
분산 웹스케일 캐시 저장소에도 사용합니다  
즉, BI에 최적화된 인 메모리 데이터베이스와  
대규모 비정형 데이터의 실시간 처리를 실행하는  
애플리케이션에도 사용합니다

### 스토리지 최적화 인스턴스

로컬 스토리지에서 대규모의 데이터셋에  
액세스할 때 적합한 인스턴스입니다  
스토리지 최적화 인스턴스의 사용 사례로는  
고주파 온라인 트랜잭션 처리인 OLTP 시스템에 사용되며
관계형과 비관계형인 NoSQL 데이터베이스에 사용합니다  
데이터베이스 섹션에서 더 자세히 살펴보겠습니다  
예를 들어, 레디스(Redis) 같은  
메모리 데이터베이스의 캐시나 데이터 웨어하우징 애플리케이션과 분산 파일 시스템에 사용됩니다

## 보안 그룹

`EC2 인스턴스에 들어오고 나가는 트래픽을 제어`합니다
보안 그룹은 간단한데요 허용 규칙만 포함합니다  
출입이 허용된 것이 무엇인지 확인할 수 있고  
IP 주소를 참조해 규칙을 만들 수 있습니다  

### SSH

시큐어 셸이라는 의미로 다음 강의에서 살펴봅니다  
포트는 22번 포트로 Linux에서 EC2 인스턴스로 로그인하도록 합니다  
파일 전송 프로토콜인 FTP의 포트는 21번 포트이며 파일 공유 시스템에 파일을 업로드하는데 사용됩니다

SFTP도 22번 포트를 사용하는 이유는 무엇일까요? SSH를 사용해서 업로드하기 때문이고 
보안 파일 전송 프로토콜이 되기 때문입니다

원격 데스크톱 프로토콜인 RDP의 3389번 포트이며
윈도우 인스턴스에 로그인할 때 사용됩니다

### SSH 연결하기

SSH는 명령줄 인터페이스 도구로  
Mac과 Linux에서 사용할 수 있고  
Windows10 이상의 버전에서 사용할 수 있습니다  
Windows10 이하의 버전이라면  
퍼티(PuTTY)를 사용하면 됩니다  
퍼티는 SSH와 동일한 것입니다  
SSH를 사용해야 할 때 Windows에서는 퍼티를 사용합니다  
퍼티는 모든 버전의 Windows에서 사용 가능합니다  

이제 머신에 접속하기 위해  
ssh ec2-user@와  
35.180.100.144를 입력하고 키를 참조하도록 하기 위해  
머신에 접속할 수 있는 키를 이 중간에 입력합니다  
키를 사용하려면 -i를 입력하고 EC2Tutorial.pem을 입력합니다  
그러면 ssh -i EC2Tutorial.pem가 되고  
그다음은 인스턴스 사용자 이름과 인스턴스 IP가 오게 됩니다  
Enter를 클릭하면 다른 경고문이 나타납니다  
비보호의 개인 키 파일이라는 경고가 나타났습니다    
이 부분이 시험에 정말 잘 나옵니다  
파일을 처음 다운로드하면  
`0644 라는 권한이 생기는데 권한이 너무 열려 있어서 개인 키가 유출될 수 있습니다`
다른 사람이 개인 키에 접근할 수 있어 bad permissions가 나타나고  
SSH를 머신에 연결할 수 없도록 합니다  
해결하는 방법이 자주 출제되며 `해결하려면 chmod 0400` 를 입력하고  
이렇게 키 이름을 참조합니다

## 온-디맨드 인스턴스

`사용한 만큼 지불하는 옵션`이죠  
비용이 초당 청구됩니다 EC2 인스턴스가 실행된  
초반 1분 이후부터 1초당 비용이 청구되죠  
온-디맨드는 클라우드에 가장 적합한 방식으로  
`가격은 높지만, 선결제나 장기 약정이 필요 없습니다`  
사용 해지, 중지, 시작이 언제든 가능하죠  
온-디맨드 방식은  
`애플리케이션 작동 방식을 예측할 수 없는`  
연속적인 단기 워크로드에 적합합니다  

간단 예시  
첫 번째로 온-디맨드는 원할 때면 언제든
리조트에 묵을 수 있는 겁니다
전액을 지불해야 하지만 원하면 언제나 방을 얻을 수 있죠

## 예약 인스턴스

온-디맨드와 비교하면 약 75%의 비용을 절약할 수 있고
`1년 혹은 3년 중에서 선택할 수 있죠`  
3년의 기간을 선택하면 해당 인스턴스를 더 오랜 기간동안  
사용할 의향이 있다고 AWS에 알리는 것과 같습니다  

종종 오랜 시간 동안 사용해야 하는 서버가 있는 경우에는
이를 AWS에 알림으로써 비용을 절감할 수 있는데요
이럴 때 사용 가능한 첫 번째 옵션이 바로 예약 인스턴스입니다
예약 인스턴스는 최소 1년 이상 사용해야 합니다
약속된 기간이죠
예약 인스턴스에는 세 가지 종류가 있는데
첫 번째로는 단순한 예약 인스턴스로
데이터베이스 같은 장기 워크로드에 사용됩니다
그리고 전환형 예약 인스턴스가 있는데
시간이 지난 후 다른 종류의
인스턴스로 바꿀 수 있는 유연형 인스턴스죠

예약 인스턴스가 적합한 곳으로는  
애플리케이션이 안정된 상태로 사용되는, 즉 데이터베이스 등이 해당되죠  
3년간 데이터베이스를 사용해야 하는 상황이라면  
이런 인스턴스를 예약함으로써 비용을 크게 절감할 수 있을 겁니다  

간단 예시  
예약 인스턴스는 오랜 기간 호텔에 머무는 것으로  
미리 계획을 세웁니다  
할인도 괜찮게 받습니다  
한 달 정도 길게 머무는 손님이니까요  

### 정기 예약 인스턴스

그리고 정기 예약 인스턴스가 있습니다
예를 들어 일 년 내내는 아니지만 (정기 예약 인스턴스는 현재 사용되지 않으나 시험에는 나올 수 있습니다)
일 년 동안 매주 목요일 (정기 예약 인스턴스는 현재 사용되지 않으나 시험에는 나올 수 있습니다)
오후 3시부터 6시까지 서버가 필요한 경우죠 (정기 예약 인스턴스는 현재 사용되지 않으나 시험에는 나올 수 있습니다)

### 스팟 인스턴스

온-디맨드 인스턴스에 비교하면 최대 90%까지 할인이 되죠  
여러분이 지불하고자 하는 가격이  
현재 스팟 인스턴스의 가격보다 낮다면  
`인스턴스가 언제든 중단될 수 있다는 겁니다`

저렴한 단기 워크로드 용 인스턴스지만  
손실 가능성이 있으며 신뢰성이 낮습니다  

그럼 어떤 유형의 워크로드가 적합할까요?  
단발성 데이터 분석인 배치 로드  
그리고 이미지 프로세싱 이미지 변환이 중단되어  
한 이미지 변환에 실패해도 나중에 다시 시작하면 되니까요  
그리고 분산된 워크로드가 있는경우

간단 예시  
아시다시피 일부 호텔은 밤에 객실이 빌 때  
아주 공격적인 할인을 선보입니다  
객실을 비워 두면 손해를 볼 것이고 사업이 망할 수도 있죠  
하지만 이 호텔은 조금 이상합니다  
만약 호텔이 여러분보다 객실비를  
더 많이 지불할 수 있는 손님을 찾게 되면  
여러분을 쫓아낼 수 있거든요  
이러면 이상한 호텔이긴 하겠지만 스팟 인스턴스가  
이런 경우와 비슷하답니다

### 전용 호스트

Amazon 전용 호스트는 EC2 인스턴스를 갖춘 유저 중심의 물리적 서버입니다  
즉, AWS 데이터 센터 내 하나의 서버 전체를 임대하는 거죠  
`전용 호스트를 사용하면 준수 요건의 처리가 쉽고`   
기존의 `서버 결합 소프트웨어 라이센스`의 사용이 가능하기 때문에  
비용을 절감할 수 있습니다  

전체 서버를 독자적으로 이용하게 되니 비용은 더 올라갑니다  
만약 복잡한 라이센스 모델의 소프트웨어를 사용하거나  
자가 라이센스를 가진 경우, 혹은 강력한 규제나 규정 준수 요건이  
있을 때도 상당히 도움을 주는 옵션이죠  

간단 예시  
이 경우 리조트 전체를 혼자서 예약합니다  
규정 준수 관련 이유로 이웃이 없어야 하거나  
서버 결합 라이센스를 가지고 있는 등의 이유 때문이죠  

### 전용 인스턴스

여러분의 전용 하드웨어에서 실행되는 EC2 인스턴스를 의미합니다  
같은 계정의 다른 인스턴스와 하드웨어를 공유하며  
인스턴스가 어떻게 배치될지에 대해서는 여러분이 간섭할 수 없습니다  
다시 말해, 전용 하드웨어가 있어도  
해당 하드웨어의 근본에는 접근할 수가 없습니다  
전용 호스트의 약한 버전이라고 할까요

# EBS 볼륨

EBS 볼륨은 일래스틱 블록 스토어의 줄임말입니다  
네트워크 USB 스틱이라고 생각하시면 됩니다  
USB 스틱처럼 한 컴퓨터에서 꺼내, 다른 컴퓨터에 꽂는  
그런 장치는 맞지만 실제 물리적 연결은 없으며  
네트워크를 통해 연결되는 거죠  

EBS 볼륨은 특정한 가용 영역에 고정되어 있으므로  
us-east-1a에 생성된 볼륨은  
us-east-1b로 연결이 불가능합니다

## 스냅샷

언제든 원하는 시점에 EBS 볼륨을 가지고 와서  
`백업이라고 불리기도 하는 스냅샷(Snapshot)`을 생성할 수 있습니다

스냅샷을 생성하는 이유는 무엇일까요? 복원 목적도 있으나  
가용 영역(AZ) 또는 리전(Region)에 걸친 스냅샷을 복사할 수 있기 때문입니다

더불어서 AWS의 다른 리전에 데이터를 전송하여 글로벌 인프라를 활용하는 것이죠  
또 다른 가용 영역에 새로운 EBS 볼륨을 복원할 수 있습니다

## AMI

AMI는 Amazon Machine Image의 약자로 사용자 지정 EC2 인스턴스를 나타냅니다  
각자의 소프트웨어 구성에 대해 운영 체제를 정의 및 설정하며 모니터링 도구를 설정할 수도 있는데  
이때 자체적으로 AMI를 생성하면 부팅과 구성에 시간이 단축됩니다

다른 사용자가 만들어서 판매하는 AMI로 자주 찾아볼 수 있습니다 AWS의 공급 업체가  
자체적으로 AMI나 구성이 훌륭한 소프트웨어를 생성하고  
여러분은 시간 절약을 위해 `마켓 플레이스 AMI에서 이들을 구매`할 수 있습니다

`AMI는 특정 AWS 지역용으로 구축되며 각 AWS 지역마다 고유`합니다.   
다른 AWS 지역의 AMI를 사용하여 EC2 인스턴스를 시작할 수는 없지만   
AMI를 대상 AWS 지역에 복사한 다음 이를 사용하여 EC2 인스턴스를 생성할 수 있습니다.

## 인스턴스 스토어

EC2 인스턴스는 가상 머신이지만 `실제로는 하드웨어 서버에 연결`되어 있습니다  
EC2 인스턴스 스토어는 I/O 성능 향상을 위해 활용할 수 있습니다  
이들이 훌륭한 처리량을 갖추고 있어서 매우 향상된 디스크 성능을  
요할 때에 활용할 수 있도록 확보할 필요가 있습니다

이때 주의할 점은 여러분이  
EC2 인스턴스, 즉 인스턴스 스토어를 중지 또는 종료하면  
해당 스토리지 또한 손실된다는 것입니다  
이 같은 이유로 이를 임시 스토리지라고 부르며  
EC2 인스턴스 스토어가  
`장기적으로 데이터를 보관할 만한 장소가 될 수 없음을 보여 줍니다`

그러면 언제 사용하는 것이 좋을까요?  
버퍼나 캐시 스크래치 데이터 또는 임시 콘텐츠를 사용하려는 경우  
이들을 보관할 좋은 장소가 되지만 장기적인 스토리지는 될 수 없습니다  
장기 스토리지의 경우에는 EBS 가 적합합니다

마지막으로 EC2 인스턴스의 기본 서버에 장애가 발생할 시에는  
해당 EC2 인스턴스가 연결된 하드웨어에도  
장애가 발생하므로 데이터 손실에 대한 위험이 존재합니다  
따라서 EC2 인스턴스 스토어를 사용할 때에는  
여러분의 필요에 따라 데이터를 백업해 두거나  
복제해 둬야 합니다

## EBS 볼륨

총 여섯 개의 유형이 있고
이들을 여러 범주로 나눌 수 있습니다

`gp2/gp3`가 있는데 이는 범용 SSD 볼륨으로
`다양한 워크로드에 대해 가격과 성능의 절충안이 되어 줍니다`

`io1과 io2`가 있습니다  
최고 성능을 자랑하는 SSD 볼륨으로  
`미션 크리티컬이자 지연 시간이 낮고 대용량의 워크로드`에 쓰입니다

`st1` 볼륨이 있는데 이는 저비용의 HDD 볼륨으로  
`잦은 접근과 처리량이 많은 워크로드`에 쓰이죠

`sc1` 볼륨은 가장 비용이 적게 드는 HDD 볼륨으로  
`접근 빈도가 낮은 워크로드`를 위해 설계되었습니다

EBS 볼륨은 어떻게 정의하는 걸까요?  
가령 크기, 처리량과 IOPS가 있죠  
`IOPS는 초당 I/O 작업 수`를 뜻합니다

EC2 인스턴스에는  
`gp2/gp3와 io1/io2만이 부팅 볼륨`으로 사용될 수 있습니다  

gp2 는 짧은 지연 시간을 자랑하며 효율적인 비용의 스토리지입니다  
시스템 부팅 볼륨에서 가상 데스크톱, 개발, 테스트 환경에서 사용할 수 있죠   
gp2는 좀 더 오래된 버전으로 볼륨이 더 작습니다 최대 3,000 IOPS에  
볼륨과 IOPS가 연결되어 있어서 IOPS가 증가할 때면  
즉 볼륨의 GB 수를 늘릴 때에 세 배 더 증가한 16,000 IOPS가 된다는 의미입니다

gp2와 gp3에는 차이가 있는데 gp3는 최신 세대의 볼륨으로  
기본 성능으로 3,000 IOPS와 초당 125MB의 처리량을 제공합니다  
각각 IOPS는 최대 16,000 처리량은 1,000MB/s까지 증가시킬 수 있습니다

gp2/gp3가 비용 효과적인 스토리지이며  
`gp3에서는 IOPS와 처리량을 독자적으로 설정`할 수 있는 반면  
`gp2에서는 그 둘이 연결되어 있다는 점`입니다

### IOPS

이는 IOPS 성능을 유지할 필요가 있는 주요 비즈니스 애플리케이션이나  
16,000 IOPS 이상을 요하는 애플리케이션에 적합합니다  
일반적으로 `데이터베이스 워크로드`에 알맞죠

### 스토리지

io1/io2에 중에서는 최신 세대를 고르는 것이 좋습니다 4에서 16TB에 달하며  
Nitro EC2 인스턴스에서는 최대 64,000 IOPS까지 가능합니다  
Nitro EC2 인스턴스의 경우 이를 통해 더 높은 IOPS까지 이용할 수 있습니다  
Nitro EC2 인스턴스가 아닌 경우에는  
최대 32,000 IOPS까지 지원됩니다  
또한 io1/io2를 이용하면 gp3 볼륨처럼 프로비저닝된 IOPS를  
스토리지 크기와 독자적으로 증가시킬 수 있습니다  
io2 이용 장점은 무엇일까요?
`io1과 동일한 비용으로 내구성과 기가 당 IOPS의 수가 더 높습니다`  
현재까지는 io2를 사용하는 것이 더 합리적인 거죠

### st1과 sc1

부팅 볼륨일 수 없습니다   
최대 16TB까지 확장

두 가지 종류의 볼륨을 제공합니다  
하나는 st1인 처리량 최적화 HDD로  
빅 데이터나 데이터 웨어하우징 로그 처리에 적합합니다

최대 처리량은 초당 500MB 그리고 최대 IOPS는 500에 달합니다  
다음으로는 sc1인 콜드 HDD가 있는데 이는 아카이브 데이터용으로  
접근 빈도가 낮은 데이터에 적합합니다  
최저 비용으로 데이터를 저장할 때에 사용하죠  
최대 처리량은 초당 250MB  
그리고 최대 IOPS도 250입니다

### EBS 다중 연결

앞서 EBS 볼륨은 단일한 EC2 인스턴스에만 연결할 수 있다고 했습니다  
EBS 다중 연결을 제외한 경우에 말이죠   
동일한 가용 영역 내의 여러 EC2 인스턴스에 연결하여 사용할 수 있습니다  
EBS는 `io1이나 io2 제품군일 때만 여러 EC2 인스턴스에 연결이 가능`합니다

## EFS 

Elastic File System 의 약자로  
다양한 가용 영역에 걸쳐 다수의 EC2 인스턴스에  
마운트 할 수 있는 관리형 NFS 혹은 네트워크 파일 시스템입니다  
즉 `다중 AZ에서 동작`하며   
이 점이 EFS와 EBS의 가장 큰 차이를 보여 줍니다  

EBS는 단일 가용 영역에 묶여 있는 반면  
`EFS는 다중 가용 영역에 걸쳐서 마운트가 가능`합니다

따라서 가용성이 매우 높죠   
확장성도 높으며 비용도 많이 듭니다

EBS는 한 번에 하나의 EC2 인스턴스에만 연결되어 있어서  
데이터가 다중의 EC2 인스턴스 간 공유되지 않지만

EFS 에서는 네트워크 파일 시스템으로  
`EFS 드라이브의 모든 EC2 인스턴스가 동일한 파일에 대한 접근 권한`을 갖습니다

EFS는 콘텐츠 관리, 웹 서비스  
데이터 공유 또는 WordPress 웹사이트에서 쓰입니다  
`표준 NFSv4.1 프로토콜`이 사용되며  
이는 네트워크 드라이브 마운트 시 기본적인 방법입니다  

EFS는 Windows가 아닌 `Linux 기반 AMI에서만 작동`합니다

EFS 옵션에 대해서는 어떤 사항을 알고 있어야 할까요? 먼저 그 규모를 알아야 합니다
EFS는 수천 명의 동시 클라이언트와  
초당 최대 10GB의 처리량을 자랑합니다   
성능이 아주 우수한 거죠  
또한 파일 시스템 자체가 페타바이트(PB) 정도까지  
확장될 수 있으므로 용량을 따로 관리할 필요가 없습니다  
자동으로 수행되죠

### 성능 모드

웹 서버 운영이나 지연 시간에 민감한 파일이 있는 경우  
범용 성능 모드를 사용합니다

EFS에서 대규모 데이터 워크로드를 처리하는 경우  
Max I/O 성능 모드가 적합합니다    
지연 시간은 더 길겠지만 처리량은 더 향상됩니다

### 처리량 모드
 
기본적으로는 버스팅(Bursting) 처리량 모드로 설정되어 있습니다
1TB의 스토리지에 대해  
초당 50MB를 저장할 수 있으며  
여기에 초당 100MB까지 확장이 가능하다는 겁니다  
일반적으로는 파일 시스템의 크기에 따라  
처리량이 증가하므로 EFS 파일 시스템의  
크지는 줄이면서 처리량을 높이기 위해서는  
프로비저닝 된 처리량 모드로 설정을 바꿀 수 있습니다  
이 모드에서는 스토리지 크기와 상관없이 처리량을 설정할 수 있죠  
1TB의 스토리지에 불과하더라도  
초당 1GB의 처리량을 요청할 수 있습니다

접근 빈도가 높은 파일에 대해 표준으로 설정되어 있으며  
EFS-IA라고 부르며 저비용의 빈도가 낮은 접근에 대한 티어가 있습니다  
이와 같은 파일을 저장할 저비용의 장소인 거죠

## EBS vs EFS

### EBS

EBS 볼륨은 한 번에 하나의 인스턴스에만 연결이 가능하고  
특정 가용 영역에 한정됩니다  

gp2에서는 디스크 크기가 늘어나면 IO도 함께 증가하죠  

io 1은 IO를 볼륨 크기와 관계 없이 독립적으로 증가시킬 수 있죠  
중요한 데이터베이스를 실행할 때 좋은 방법입니다

EBS를 다른 가용 영역으로 옮기고자 할 때는  
가장 먼저 스냅샷을 찍어야 합니다  
스냅샷을 찍었다면  
다른 AZ에서 그 스냅샷을 복원시키죠

EBS의 스냅샷이나 백업을 만들 때에는  
EBS 볼륨 내의 IO를 전부 사용하게 되니  
`인스턴스가 EBS를 사용 중이 아닐 때에만 실행`하셔야 합니다

EBS의 경우에는 실제 사용한 양이 아니라  
EBS 드라이브의 크기에 따라  
실제 사용량이 아니라 정해진 사용량을 지불하는 식이었죠

반면 EBS는 네트워크 볼륨을 한 번에 하나의 인스턴스에  
연결할 수 있고 특정 AZ 내로 한정이 되죠  
인스턴스 스토어는 EC2 인스턴스에 IO를 최대로 사용하게끔 해주지만,
`인스턴스가 망가지면 함께 망가지는`  
임시 드라이브인 거죠

### EFS

EFS는 여러 개의 가용 영역에 걸쳐  
`무수히 많은 인스턴스들에 연결`될 수 있습니다

이는 Linux 인스턴스에서만, 가능한데 POSIX 파일 시스템이라  
Windows에서 구동되지 않기 때문입니다

`EFS 는 EBS 보다 훨씬 비쌉니다`  
거의 세 배 정도 더 비싸죠
































































