# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용하여 다양한 AI 애플리케이션을 구축하는 방법을 소개하는 한국어 예제 모음입니다. 이 저장소는 AWS 공식 샘플을 기반으로, 한국 사용자들이 더 쉽게 이해하고 실습할 수 있도록 상세한 설명과 가이드를 추가했습니다.

  - [https://github.com/aws-samples/text-to-sql-bedrock-workshop](https://github.com/aws-samples/text-to-sql-bedrock-workshop)
  - [https://github.com/aws-samples/amazon-bedrock-samples](https://github.com/aws-samples/amazon-bedrock-samples)


## 사전 준비
본격적인 실습에 앞서, 원활한 진행을 위해 아래의 준비 과정을 순서대로 완료해주세요.

### AWS 리전 설정
이 실습은 AWS 미국 오레곤 `us-west-2` 리전에서 진행하는 것을 권장합니다. AWS 콘솔 상단에서 리전이 `us-west-2`로 설정되어 있는지 확인해주세요.

### 모델 액세스 설정
Amazon Bedrock에서 사용할 기반 모델(Foundation Model)에 대한 접근 권한을 설정해야 합니다.

1. AWS 콘솔에서 Amazon Bedrock 서비스로 이동합니다.
2. 왼쪽 메뉴 하단의 [모델 액세스(Model access)] 를 클릭합니다.
<img width="1894" height="885" alt="image" src="https://github.com/user-attachments/assets/1b2a7a3b-b403-49b5-a752-e4cde32a2577" />
3. [모델 액세스 관리(Manage model access)] 버튼을 누르고, 실습에 사용할 모델들, 특히 Anthropic의 Claude 모델과 Amazon의 Titan 모델을 선택하여 접근을 요청합니다.
<img width="1878" height="680" alt="image" src="https://github.com/user-attachments/assets/5403b2fe-3be1-4254-9997-f8f19ad1ae4c" />

> 팁: 모델 접근 권한을 요청하는 것만으로는 비용이 발생하지 않으니, 원활한 실습을 위해 모든 모델의 접근 권한을 요청하는 것을 추천합니다.




### SageMaker AI 노트북 생성
실습은 Amazon SageMaker AI 노트북 환경에서 진행됩니다. 노트북 인스턴스를 생성하고 필요한 권한을 부여하는 과정입니다.

<img width="1882" height="697" alt="image" src="https://github.com/user-attachments/assets/b0163642-4062-43ae-8165-4293f2a60983" />


1. AWS 콘솔에서 Amazon SageMaker AI 서비스로 이동합니다.
2. 왼쪽 메뉴에서 [Notebooks] 를 클릭하고 [노트북 인스턴스 생성] 버튼을 누릅니다.
3. 노트북 인스턴스 생성 페이지에서 다음 정보를 입력합니다.
	1. 노트북 인스턴스 이름에 노트북 인스턴스의 이름을 입력합니다.
	2. IAM 역할에서 SageMaker AI 리소스에 액세스하는 데 필요한 권한이 있는 계정의 기존 IAM 역할 또는 새 역할 생성을 선택합니다. 새 역할 생성을 선택하면 SageMaker AI는 라는 IAM 역할을 생성합니다. `AmazonSageMaker-ExecutionRole-YYYYMMDDTHHmmSS`. AWS 관리형 정책은 역할에 AmazonSageMakerFullAccess 연결됩니다. 역할은 노트북 인스턴스가 SageMaker AI 및 Amazon S3를 호출할 수 있는 권한을 제공합니다.
4. 노트북 인스턴스 생성을 선택합니다. 몇 분 내에 Amazon SageMaker AI는 ML 컴퓨팅 인스턴스, 즉이 경우 노트북 인스턴스를 시작하고 ML 스토리지 볼륨을 여기에 연결합니다.	
5. 2~3분 정도 기다린 후, 새로 고침 버튼을 클릭하여 인스턴스 상태가 InService로 바뀐 것을 확인합니다.

<img width="1611" height="296" alt="image" src="https://github.com/user-attachments/assets/a0a3dede-5b5e-444f-82cd-6abae08800b8" />




### IAM 정책 설정

<img width="1590" height="693" alt="image" src="https://github.com/user-attachments/assets/7a9a6806-3bf2-4fc0-aa4b-ce12af0a868d" />

1. 생성된 노트북 인스턴스 이름을 클릭합니다.
2. 권한 및 암호화 섹션에서 [IAM 역할 ARN]을 클릭합니다.
3. [권한 추가(Add permissions)] > [인라인 정책 생성(Create inline policy)] 을 선택합니다.

<img width="1586" height="579" alt="image" src="https://github.com/user-attachments/assets/e1dadcb9-185e-48b4-bc15-ac164c1c4668" />


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


- 다시 생성한 노트북 인스턴스 설정 화면으로 돌아와 [JupyterLab 열기(Open JupyterLab)] 버튼을 클릭합니다.
  
<img width="1586" height="696" alt="image" src="https://github.com/user-attachments/assets/4d466a55-16b8-42a8-9a34-a88245f94413" />


- Other의 첫 번째 항목인 Terminal을 클릭하여 터미널에 진입합니다.

<img width="1309" height="738" alt="image" src="https://github.com/user-attachments/assets/244b3d9b-80c3-4a21-8a01-fe324287804f" />


- 실습에 사용할 코드를 다운받기 위해 다음 명령어를 순서대로 입력합니다.
```bash
cd SageMaker/
git clone https://github.com/harheem/ko-aws-bedrock-agent-samples.git
```

- 코드가 성공적으로 다운되면 아래 이미지처럼 디렉토리에서 `ko-aws-bedrock-agent-samples` 폴더를 확인할 수 있습니다.

<img width="858" height="307" alt="image" src="https://github.com/user-attachments/assets/2cdab695-df90-4e03-a1c6-be73667de87e" />


- 이제 실습을 시작할 준비가 되었습니다! 노트북 파일(.ipynb)을 열고 conda_python3을 선택하여 실습을 진행해주세요.

<img width="716" height="320" alt="image" src="https://github.com/user-attachments/assets/e4774766-2ef4-4c30-a2db-c83d28869d9c" />



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
  - **Module 02**: DIN-SQL과 Few-shot 학습 (추가 예정)
  - **Module 03**: RAG를 활용한 Text-to-SQL (추가 예정)

## 시작하기

각 디렉토리의 Jupyter Notebook 파일(.ipynb)을 실행하여 예제를 체험할 수 있습니다. 코드 예제를 시작하려면 먼저 **Amazon Bedrock에 대한 접근 권한**이 있는지 확인하세요. 그런 다음, 이 저장소(repo)를 복제(clone)하고 원하는 폴더로 이동합니다. 자세한 안내는 각 폴더의 README에 제공되어 있습니다.

## 주요 학습 목표

  - Bedrock 에이전트 생성 및 구성
  - 지식 베이스와 액션 그룹 통합
  - 메모리 기능으로 대화형 AI 구현
  - Text-to-SQL 시스템 최적화 기법
