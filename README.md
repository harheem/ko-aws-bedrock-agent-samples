# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용하여 다양한 AI 애플리케이션을 구축하는 방법을 소개하는 한국어 예제 모음입니다. 이 저장소는 AWS 공식 샘플을 기반으로, 한국 사용자들이 더 쉽게 이해하고 실습할 수 있도록 상세한 설명과 가이드를 추가했습니다.

  - [https://github.com/aws-samples/text-to-sql-bedrock-workshop](https://github.com/aws-samples/text-to-sql-bedrock-workshop)
  - [https://github.com/aws-samples/amazon-bedrock-samples](https://github.com/aws-samples/amazon-bedrock-samples)


## 사전 준비
본격적인 실습에 앞서, 원활한 진행을 위해 아래의 준비 과정을 순서대로 완료해주세요.

### AWS 리전 설정
이 실습은 AWS 미국 버지니아 북부 `us-east-1` 리전에서 진행하는 것을 권장합니다. AWS 콘솔 상단에서 리전이 `us-east-1`로 설정되어 있는지 확인해주세요.

### 모델 액세스 설정
Amazon Bedrock에서 사용할 기반 모델(Foundation Model)에 대한 접근 권한을 설정해야 합니다.

1. AWS 콘솔에서 Amazon Bedrock 서비스로 이동합니다.
2. 왼쪽 메뉴 하단의 [모델 액세스(Model access)] 를 클릭합니다.
<img width="1894" height="885" alt="image" src="https://github.com/user-attachments/assets/1b2a7a3b-b403-49b5-a752-e4cde32a2577" />
3. [모델 액세스 관리(Manage model access)] 버튼을 누르고, 실습에 사용할 모델들, 특히 Anthropic의 Claude 모델과 Amazon의 Titan 모델을 선택하여 접근을 요청합니다.
<img width="1878" height="680" alt="image" src="https://github.com/user-attachments/assets/5403b2fe-3be1-4254-9997-f8f19ad1ae4c" />

> 팁: 모델 접근 권한을 요청하는 것만으로는 비용이 발생하지 않으니, 원활한 실습을 위해 모든 모델의 접근 권한을 요청하는 것을 추천합니다.




### SageMaker AI 도메인 생성
실습은 Amazon SageMaker AI 노트북 환경에서 진행됩니다. 노트북 인스턴스를 생성하고 필요한 권한을 부여하는 과정입니다.

1번 이미지

1. AWS 콘솔에서 Amazon SageMaker AI 서비스로 이동합니다.
2. 왼쪽 메뉴에서 [Notebooks] 를 클릭하고 [노트북 인스턴스 생성] 버튼을 누릅니다.
3. 인스턴스 이름을 입력한 후 인스턴스를 생성합니다.
4. 2~3분 정도 기다린 후, 새로 고침 버튼을 클릭하여 인스턴스 상태가 InService로 바뀐 것을 확인합니다.

2번 이미지


### IAM 정책 설정

3번 이미지
1. 생성된 노트북 인스턴스 이름을 클릭합니다.
2. 권한 및 암호화 섹션에서 [IAM 역할 ARN]을 클릭합니다.
3. [권한 추가(Add permissions)] > [인라인 정책 생성(Create inline policy)] 을 선택합니다.

4번 이미지

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
  
5번 이미지

- Other의 첫 번째 항목인 Terminal을 클릭하여 터미널에 진입합니다.

6번 이미지

- 실습에 사용할 코드를 다운받기 위해 다음 명령어를 순서대로 입력합니다.
```bash
cd SageMaker/
git clone https://github.com/harheem/ko-aws-bedrock-agent-samples.git
```

- 코드가 성공적으로 다운되면 아래 이미지처럼 디렉토리에서 `ko-aws-bedrock-agent-samples` 폴더를 확인할 수 있습니다.

7번 이미지

- 이제 실습을 시작할 준비가 되었습니다!

8번 이미지


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

## 주요 학습 목표

  - Bedrock 에이전트 생성 및 구성
  - 지식 베이스와 액션 그룹 통합
  - 메모리 기능으로 대화형 AI 구현
  - Text-to-SQL 시스템 최적화 기법
