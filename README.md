# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용한 다양한 AI 애플리케이션 구축 예제 모음입니다.
아래 레포지토리의 자료를 한국어로 번역하면서 처음 접하는 사용자들도 이해할 수 있게 설명을 추가하였습니다.

  - [https://github.com/aws-samples/text-to-sql-bedrock-workshop](https://github.com/aws-samples/text-to-sql-bedrock-workshop)
  - [https://github.com/aws-samples/amazon-bedrock-samples](https://github.com/aws-samples/amazon-bedrock-samples)


## 사전 준비
이 실습은 미국(오레곤) `us-west-2`에서 진행해야 합니다.

### 모델 액세스 설정
Amazon Bedrock으로 이동하여 모델 권한을 설정합니다.
<img width="1894" height="885" alt="image" src="https://github.com/user-attachments/assets/1b2a7a3b-b403-49b5-a752-e4cde32a2577" />
<img width="1878" height="680" alt="image" src="https://github.com/user-attachments/assets/5403b2fe-3be1-4254-9997-f8f19ad1ae4c" />

모든 모델을 선택해도 요금이 과금되지 않습니다.

다양한 모델 테스트를 위해 체크 박스를 선택하여 모든 모델에 대한 액세스를 요청합니다.



### SageMaker AI 도메인 생성
이 실습은 Amazon SageMaker AI Studio에서 진행하는 것을 기반으로 만들어졌습니다.
<img width="1886" height="619" alt="image" src="https://github.com/user-attachments/assets/2f99fed6-12f3-4baf-9078-069721187992" />

생성된 도메인의 기본 실행 역할 이름을 기억합니다.

ex) `arn:aws:iam::626635425893:role/service-role/AmazonSageMaker-ExecutionRole-20250717T154828`

### IAM 정책 설정
IAM으로 이동한 후 위에서 확인한 역할 이름을 선택합니다.
<img width="1894" height="598" alt="image" src="https://github.com/user-attachments/assets/9f790b46-7ffe-458e-9ed9-09b425917f0b" />



권한 설정에서 아래 권한을 추가합니다.
<img width="1599" height="442" alt="image" src="https://github.com/user-attachments/assets/cff3f3a6-737e-4e93-a01d-52430f95197a" />


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
				"iam:GenerateCredentialReport",
				"iam:UpdateAccountName",
				"iam:SimulateCustomPolicy",
				"iam:CreateAccountAlias",
				"iam:GetAccountAuthorizationDetails",
				"iam:DeleteAccountAlias",
				"iam:GetCredentialReport",
				"iam:ListPolicies",
				"iam:DeleteAccountPasswordPolicy",
				"iam:ListSAMLProviders",
				"iam:ListRoles",
				"iam:UploadCloudFrontPublicKey",
				"iam:GetContextKeysForCustomPolicy",
				"iam:UpdateAccountPasswordPolicy",
				"iam:ListOpenIDConnectProviders",
				"iam:GetAccountName",
				"iam:ListUsers",
				"iam:ListSTSRegionalEndpointsStatus",
				"iam:DeleteRole",
				"iam:DeleteRolePolicy",
				"iam:DetachRolePolicy",
				"iam:DetachUserPolicy",
				"iam:DetachGroupPolicy",
				"iam:AttachRolePolicy",
				"iam:AttachUserPolicy",
				"iam:AttachGroupPolicy"
			],
			"Resource": "*"
		}
	]
}
```

<img width="1880" height="875" alt="image" src="https://github.com/user-attachments/assets/99c64ed9-ddf6-4bda-a67d-731216def7bd" />
정책 이름을 입력하고 정책을 생성합니다.

### SageMaker AI Studio
<img width="1883" height="806" alt="image" src="https://github.com/user-attachments/assets/2684f259-309d-4934-a4a7-5f5059296b1f" />

JupyterLab을 클릭한 후 스페이스를 생성합니다.
<img width="870" height="429" alt="image" src="https://github.com/user-attachments/assets/c48bb86d-abe6-4f61-be51-ea9d835f4305" />

- 노트북 인스턴스: `ml.t3.medium`
- 이미지: `Sagemaker Distribution 3.2.0`

[Run Space]를 클릭한 후 JupyterLab에 접속합니다.

## GitHub 저장소에서 코드 샘플 다운로드
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
