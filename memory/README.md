# 메모리 기능을 갖춘 에이전트 생성

이 폴더는 Amazon Bedrock 에이전트의 [메모리](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-memory.html) 기능을 활용한 여행 에이전트 예제를 제공합니다. 이 README는 Amazon Bedrock 에이전트나 관련 개념에 익숙하지 않은 분들을 위해 주요 개념과 코드의 역할을 상세히 설명합니다.


## 핵심 개념: 에이전트와 메모리

### Amazon Bedrock 에이전트란?

Amazon Bedrock 에이전트는 단순한 챗봇을 넘어, 사용자의 요청을 이해하고 여러 단계의 작업을 자율적으로 수행할 수 있는 프로그램입니다. 예를 들어, "다음 주 제주도 여행 계획을 세워줘"라는 요청을 받으면, 항공편을 조회하고, 숙소를 검색하며, 날씨를 확인하는 등의 복합적인 작업을 알아서 처리할 수 있습니다.

### 에이전트에게 메모리가 왜 필요한가?

기본적으로 에이전트는 방금 나눈 대화가 끝나면 모든 것을 잊어버립니다. 하지만 **메모리** 기능을 활성화하면, 에이전트는 특정 사용자와의 이전 대화 내용을 기억할 수 있습니다.

예를 들어, 사용자가 어제 "저는 해산물을 좋아해요"라고 말했다면, 오늘 "저녁 식사 추천해줘"라고 물었을 때 메모리 기능이 있는 에이전트는 이 정보를 기억하고 해산물 식당을 추천해 줄 수 있습니다. 이처럼 메모리는 대화의 연속성을 부여하여 훨씬 더 개인화되고 지능적인 상호작용을 가능하게 합니다.



## 예제 아키텍처

이 예제에서는 다음과 같은 구조의 여행 상담 테스트 에이전트를 생성합니다. 사용자의 요청은 Bedrock 에이전트로 전달되고, 에이전트는 필요에 따라 외부 도구(API)를 호출하여 답변을 생성합니다. 대화 내용은 메모리에 요약되어 저장됩니다.



## 1\. 메모리 기능을 활성화하여 에이전트 생성하기

에이전트를 처음 만들 때, 메모리 기능을 사용하겠다고 설정해야 합니다. 이는 `create_agent` 함수 호출 시 `memoryConfiguration` 매개변수를 추가하여 이루어집니다.

  * `enabledMemoryTypes`: 어떤 종류의 메모리를 사용할지 지정합니다. 현재는 대화가 끝났을 때 그 내용을 요약해서 저장하는 **`SESSION_SUMMARY`** 방식만 지원됩니다.
  * `storageDays`: 요약된 대화 내용을 며칠 동안 보관할지 설정합니다.
  * `idleSessionTTLInSeconds`: **세션의 유휴 시간 제한**을 초 단위로 설정합니다. 예를 들어 `1800`으로 설정하면, 사용자와의 대화가 30분 동안 아무런 활동 없이 멈춰 있을 경우 해당 대화 세션이 자동으로 종료됩니다. 이는 불필요한 리소스 낭비를 방지합니다.

아래 코드는 boto3 SDK의 `create_agent` 함수를 사용하여 에이전트 생성 시 메모리 기능을 구성하는 방법을 보여줍니다.

```python
    response = bedrock_agent_client.create_agent(
        agentName=agent_name,
        agentResourceRoleArn=agent_role['Role']['Arn'],
        description=agent_description,
        # 30분(1800초)간 활동이 없으면 세션을 자동 종료
        idleSessionTTLInSeconds=1800,
        foundationModel=agent_foundation_model,
        instruction=agent_instruction,
        # 메모리 설정을 추가하는 부분
        memoryConfiguration={
            "enabledMemoryTypes": ["SESSION_SUMMARY"], # 세션 요약 메모리 사용
            "storageDays": 30 # 30일간 메모리 보관
        }
    )
```



## 2\. 에이전트 호출 시 메모리 사용하기

메모리 기능이 활성화된 에이전트를 호출할 때는 두 가지 중요한 ID를 함께 전달해야 합니다.

  * **`memoryId`**: **사용자**를 식별하는 고유 ID입니다. 한 명의 사용자는 여러 번의 대화 세션을 가질 수 있으며, 이 모든 대화 기록은 하나의 `memoryId`에 연결됩니다. A라는 사용자와의 모든 대화 기억을 모아두는 '기억 보관함'이라고 생각할 수 있습니다.
  * **`sessionId`**: **개별 대화**를 식별하는 고유 ID입니다. 사용자가 에이전트와 나누는 하나의 대화 흐름을 나타냅니다. A라는 사용자가 오전에 나눈 대화와 오후에 나눈 대화는 각각 다른 `sessionId`를 갖습니다.

<!-- end list -->

```python
    # 에이전트 API 호출
    agent_response = bedrock_agent_runtime_client.invoke_agent(
        inputText="사용자 질문",
        agentId="<AGENT_ID>",
        agentAliasId="<AGENT_ALIAS_ID>",
        sessionId="<현재 대화를 위한 고유 ID>",
        enableTrace=True,
        # 대화를 종료하고 내용을 메모리에 요약 저장하려면 True로 설정
        endSession=False, 
        memoryId="<사용자를 식별하는 고유 ID>"
    )
```

### 메모리 저장 시점 (`endSession` 플래그)

에이전트는 대화가 진행되는 동안 모든 내용을 실시간으로 기억하는 것이 아니라, 대화가 **종료**되는 시점에 그 내용을 요약하여 `memoryId`에 해당하는 메모리에 저장합니다.

대화를 끝내고 내용을 요약하여 저장하도록 하려면, `invoke_agent` 함수를 호출할 때 **`endSession` 플래그를 `True`로 설정**해야 합니다. 이 플래그가 `True`로 설정된 호출이 이루어지면, 해당 `sessionId`의 대화 내용 전체가 요약되어 에이전트의 장기 기억(`memoryId`)에 저장됩니다.



## 3\. 저장된 메모리 내용 확인 및 삭제

### 메모리 확인하기

특정 `memoryId`에 어떤 내용이 요약되어 저장되었는지 확인하고 싶을 때는 `get_agent_memory` 함수를 사용합니다. 이를 통해 에이전트가 특정 사용자에 대해 무엇을 기억하고 있는지 조회할 수 있습니다.

```python
    memory_content = bedrock_agent_runtime_client.get_agent_memory(
        agentAliasId="<AGENT_ALIAS_ID>",
        agentId="<AGENT_ID>",
        memoryId="<조회하고 싶은 사용자의 메모리 ID>",
        memoryType='SESSION_SUMMARY' # 세션 요약 타입의 메모리를 조회
    )
```

### 메모리 삭제하기

개인정보 보호 등의 이유로 특정 사용자의 모든 대화 기록을 삭제해야 할 경우, `delete_agent_memory` 함수를 사용할 수 있습니다. 이 함수는 특정 `memoryId`와 연결된 **모든** 요약 정보를 영구적으로 삭제합니다. 개별 대화(`sessionId`)만 삭제하는 것이 아니라, 해당 사용자의 전체 기억을 삭제하는 점에 유의해야 합니다.

```python
    response = bedrock_agent_runtime_client.delete_agent_memory(
        agentAliasId="<AGENT_ALIAS_ID>",
        agentId="<AGENT_ID>",
        memoryId="<삭제할 사용자의 메모리 ID>"
    )
```



## 4\. 메모리 크기 제한

Amazon Bedrock 에이전트의 메모리(`SESSION_SUMMARY`) 크기에는 명시적인 바이트(MB/GB) 단위의 제한보다는 **토큰 수**에 기반한 실질적인 제한이 있습니다.

메모리는 대화 전체를 그대로 저장하는 것이 아니라, 대화가 종료될 때 기반 모델(Foundation Model)이 **대화 내용을 요약**하여 저장하는 방식입니다. 따라서 메모리의 크기는 이 '요약' 과정과 관련된 제한의 영향을 받습니다.

1.  **요약 모델의 토큰 제한**: 대화 내용을 요약하는 데 사용되는 기반 모델(예: Claude)은 한 번에 처리할 수 있는 최대 토큰 수가 정해져 있습니다. 매우 긴 대화 내용은 요약 모델의 컨텍스트 창(Context Window)을 초과할 수 있으며, 이 경우 요약이 완전하게 이루어지지 않을 수 있습니다.
2.  **메모리 조회 시 토큰 제한**: 저장된 요약본을 다음 대화에서 참조할 때, 이 요약 정보 역시 프롬프트의 일부로 입력 토큰을 소모합니다. 요약된 내용이 너무 길면 새로운 질문과 함께 처리될 때 모델의 전체 입력 토큰 제한을 초과할 수 있습니다.

**결론적으로,** 명확하게 "최대 OO MB"라고 정해진 하드 리밋은 없지만, 대화 내용을 요약하고 다시 참조하는 과정에서 기반 모델의 **토큰 제한**이 실질적인 메모리 크기 제한으로 작용합니다. 따라서 매우 긴 대화의 모든 세부 사항을 완벽하게 기억하는 데는 한계가 있을 수 있습니다.
