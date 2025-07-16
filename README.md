# Amazon Bedrock 에이전트 샘플 (한국어)

AWS Bedrock 에이전트를 활용한 다양한 AI 애플리케이션 구축 예제 모음입니다.
아래 레포지토리의 자료를 한국어로 번역하였습니다.
- https://github.com/aws-samples/text-to-sql-bedrock-workshop
- https://github.com/aws-samples/amazon-bedrock-samples

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

각 디렉토리의 Jupyter Notebook 파일(.ipynb)을 실행하여 예제를 체험할 수 있습니다.



## 요구사항

- AWS 계정 및 Bedrock 모델 접근 권한

- Python 3.8+

- 필요한 패키지는 각 노트북의 requirements.txt 참조



## 주요 학습 목표
- Bedrock 에이전트 생성 및 구성

- 지식 베이스와 액션 그룹 통합

- 메모리 기능으로 대화형 AI 구현

- Text-to-SQL 시스템 최적화 기법