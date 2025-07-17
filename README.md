# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용하여 다양한 AI 애플리케이션을 구축하는 방법을 소개하는 한국어 예제 모음입니다. 이 저장소는 AWS 공식 샘플을 기반으로, 한국 사용자들이 더 쉽게 이해하고 실습할 수 있도록 상세한 설명과 가이드를 추가했습니다.

  - [https://github.com/aws-samples/text-to-sql-bedrock-workshop](https://github.com/aws-samples/text-to-sql-bedrock-workshop)
  - [https://github.com/aws-samples/amazon-bedrock-samples](https://github.com/aws-samples/amazon-bedrock-samples)


## 사전 준비
본격적인 실습에 앞서, 원활한 진행을 위해 아래의 준비 과정을 순서대로 완료해주세요.

### AWS 리전 설정
이 실습은 AWS 미국 서부(오레곤) us-west-2 리전에서 진행하는 것을 권장합니다. AWS 콘솔 상단에서 리전이 us-west-2로 설정되어 있는지 확인해주세요.

### 모델 액세스 설정
Amazon Bedrock에서 사용할 기반 모델(Foundation Model)에 대한 접근 권한을 설정해야 합니다.

1. AWS 콘솔에서 Amazon Bedrock 서비스로 이동합니다.
2. 왼쪽 메뉴 하단의 [모델 액세스(Model access)] 를 클릭합니다.
<img width="1894" height="885" alt="image" src="https://github.com/user-attachments/assets/1b2a7a3b-b403-49b5-a752-e4cde32a2577" />
3. [모델 액세스 관리(Manage model access)] 버튼을 누르고, 실습에 사용할 모델들, 특히 Anthropic의 Claude 모델과 Amazon의 Titan 모델을 선택하여 접근을 요청합니다.
<img width="1878" height="680" alt="image" src="https://github.com/user-attachments/assets/5403b2fe-3be1-4254-9997-f8f19ad1ae4c" />

> 팁: 모델 접근 권한을 요청하는 것만으로는 비용이 발생하지 않으니, 원활한 실습을 위해 모든 모델의 접근 권한을 요청하는 것을 추천합니다.




### SageMaker AI 도메인 생성
실습은 Amazon SageMaker Studio 환경에서 진행됩니다. Studio를 설정하고 필요한 권한을 부여하는 과정입니다.

<img width="1886" height="619" alt="image" src="https://github.com/user-attachments/assets/2f99fed6-12f3-4baf-9078-069721187992" />

1. AWS 콘솔에서 Amazon SageMaker 서비스로 이동합니다.
2. 왼쪽 메뉴에서 [도메인(Domain)] 을 클릭하고 [도메인 생성(Create domain)] 버튼을 누릅니다.
3. 빠른 설정을 위해 [빠른 설치(Quick setup)] 를 선택하여 도메인을 생성합니다.
4. 생성이 완료되면, 사용자 프로필(User profiles) 목록에서 기본으로 생성된 default-user의 실행 역할(Execution Role) ARN을 복사하여 기록해두세요. 다음 단계에서 권한을 추가할 때 필요합니다.

   
ex) `arn:aws:iam::626635425893:role/service-role/AmazonSageMaker-ExecutionRole-20250717T154828`

### IAM 정책 설정
1. AWS 콘솔에서 IAM 서비스로 이동합니다.
2. 왼쪽 메뉴에서 [역할(Roles)] 을 클릭하고, 위 단계에서 기록해 둔 SageMaker 실행 역할을 검색하여 선택합니다.
3. [권한 추가(Add permissions)] > [인라인 정책 생성(Create inline policy)] 을 선택합니다.
4. JSON 탭을 선택하고, 아래의 정책을 붙여넣은 후 정책 이름을 지정하여 생성합니다. 이 정책은 Bedrock, Lambda, S3 등 실습에 필요한 서비스에 접근할 권한을 부여합니다.
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "OtherServices",
			"Effect": "Allow",
			"Action": [
				"dynamodb:*",
				"bedrock:*",
				"aoss:*",
				"lambda:*",
				"s3:*",
				"glue:*",
				"athena:*",
				"cloudformation:*",
				"iam:*",
				"ec2:*",
				"rds:*",
				"sagemaker:*"
			],
			"Resource": "*"
		}
	]
}
```

<img width="1894" height="598" alt="image" src="https://github.com/user-attachments/assets/9f790b46-7ffe-458e-9ed9-09b425917f0b" />


<img width="1880" height="875" alt="image" src="https://github.com/user-attachments/assets/99c64ed9-ddf6-4bda-a67d-731216def7bd" />


### SageMaker AI Studio
<img width="1883" height="806" alt="image" src="https://github.com/user-attachments/assets/2684f259-309d-4934-a4a7-5f5059296b1f" />

- 다시 SageMaker 도메인 화면으로 돌아와 [실행(Launch)] 드롭다운 메뉴에서 [Studio] 를 선택합니다.
  
- default-user 프로필에서 [JupyterLab 실행(Run JupyterLab)] 버튼을 클릭하여 스페이스를 생성하고 실행합니다.
  
	- 인스턴스 타입: ml.t3.medium (실습에 충분합니다)
   
	- 이미지: SageMaker Distribution (최신 버전을 권장합니다)


<img width="870" height="429" alt="image" src="https://github.com/user-attachments/assets/c48bb86d-abe6-4f61-be51-ea9d835f4305" />


### GitHub 저장소에서 코드 샘플 다운로드
터미널을 클릭한 후, 다음 코드를 사용하여 실습 코드를 다운로드 받으세요.
<img width="1247" height="700" alt="image" src="https://github.com/user-attachments/assets/fa2a53d5-806b-4853-9025-d6d225e7fe0d" />


```bash
git clone https://github.com/harheem/ko-aws-bedrock-agent-samples.git
```

## 프로젝트 구조

### action-group/

지식 베이스와 액션 그룹이 통합된 레스토랑 예약 에이전트

  - **기능**: 메뉴 조회, 테이블 예약/수정/삭제
  - **구성요소**: Knowledge Base, DynamoDB, Lambda 함수
  - **데이터**: 레스토랑 메뉴 PDF 파일들

### memory/

메모리 기능을 갖춘 여행 예약 에이전트

  - **기능**: 여행 예약 생성/수정/취소, 대화 맥락 기억
  - **특징**: 세션 요약 메모리로 이전 대화 내용 활용
  - **구성요소**: Lambda 함수, 메모리 설정

### text-to-sql/

자연어를 SQL 쿼리로 변환하는 시스템

  - **Module 01**: 단일 테이블 최적화 (의료 데이터)
  - **Module 02**: DIN-SQL과 Few-shot 학습
  - **Module 03**: RAG를 활용한 Text-to-SQL

## 시작하기

각 디렉토리의 Jupyter Notebook 파일(.ipynb)을 실행하여 예제를 체험할 수 있습니다. 코드 예제를 시작하려면 먼저 **Amazon Bedrock에 대한 접근 권한**이 있는지 확인하세요. 그런 다음, 이 저장소(repo)를 복제(clone)하고 원하는 폴더로 이동합니다. 자세한 안내는 각 폴더의 README에 제공되어 있습니다.

## 요구사항

  - AWS 계정 및 Bedrock 모델 접근 권한
  - Python 3.8+
  - 필요한 패키지는 각 노트북의 `requirements.txt` 참조

### Bedrock을 위한 AWS IAM 권한 활성화하기

현재 환경에서 사용하는 AWS 자격증명(SageMaker의 경우 **Studio/노트북 실행 역할**, 자체 관리 노트북이나 다른 사용 사례의 경우 IAM 역할 또는 사용자)은 Amazon Bedrock 서비스를 호출할 수 있는 충분한 **AWS IAM 권한**을 가지고 있어야 합니다.

Bedrock 접근 권한을 부여하는 방법은 다음과 같습니다:

1.  **AWS IAM 콘솔**을 엽니다.
2.  사용 중인 **역할(Role)**(SageMaker 또는 다른 IAM 역할을 사용하는 경우) 또는 \*\*사용자(User)\*\*를 찾습니다.
3.  \*\*권한 추가(Add Permissions) \> 인라인 정책 생성(Create Inline Policy)\*\*을 선택하여 새로운 인라인 권한을 연결합니다.
4.  **JSON** 편집기를 열고 아래 예시 정책을 붙여넣습니다:

<!-- end list -->

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockFullAccess",
            "Effect": "Allow",
            "Action": ["bedrock:*"],
            "Resource": "*"
        }
    ]
}
```

⚠️ **참고 1**: Amazon SageMaker에서 노트북 실행 역할은 일반적으로 AWS 콘솔에 로그인하는 사용자와는 **별개**입니다. Amazon Bedrock용 AWS 콘솔을 탐색하려면 콘솔 사용자/역할에도 권한을 부여해야 합니다.

⚠️ **참고 2**: 최상위 폴더 변경에 대해서는 GitHub 관리자에게 문의해주세요.

Bedrock의 세분화된 작업 및 리소스 권한에 대한 더 자세한 정보는 [Bedrock 개발자 안내서](https://docs.aws.amazon.com/bedrock/latest/userguide/security_iam_id-based-policy-examples.html)를 확인하세요.

## 주요 학습 목표

  - Bedrock 에이전트 생성 및 구성
  - 지식 베이스와 액션 그룹 통합
  - 메모리 기능으로 대화형 AI 구현
  - Text-to-SQL 시스템 최적화 기법
