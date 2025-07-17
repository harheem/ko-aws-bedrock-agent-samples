# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용한 다양한 AI 애플리케이션 구축 예제 모음입니다.
아래 레포지토리의 자료를 한국어로 번역하면서 처음 접하는 사용자들도 이해할 수 있게 설명을 추가하였습니다.

  - [https://github.com/aws-samples/text-to-sql-bedrock-workshop](https://github.com/aws-samples/text-to-sql-bedrock-workshop)
  - [https://github.com/aws-samples/amazon-bedrock-samples](https://github.com/aws-samples/amazon-bedrock-samples)

## SageMaker Studio
<img width="1662" height="403" alt="image" src="https://github.com/user-attachments/assets/f7726e16-026d-4506-a8ee-69a0ab17476b" />
이 실습은 Amazon SageMaker AI의 Notebooks에서 실행하는 것을 권장드립니다.

- 노트북 인스턴스 유형: ml.t3.medium
- 플랫폼 식별자: Amazone Linux 2, Jupyter Lab 4


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
