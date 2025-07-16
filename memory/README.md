# 메모리 기능을 갖춘 에이전트 생성

이 폴더에서는 Amazon Bedrock 에이전트의 새로운 [메모리](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-memory.html) 기능을 활용한 여행 에이전트 예제를 제공합니다.

에이전트 생성 시 `memoryConfiguration` 매개변수를 통해 메모리 기능을 활성화할 수 있습니다. 이후 에이전트 호출 시 `memoryId`를 사용하여 세션을 추적하고 메모리에 요약 정보를 저장할 수 있습니다.

이 예제에서는 다음과 같은 구조의 테스트 에이전트를 생성합니다:

![에이전트 아키텍처](images/architecture.png)


아래 코드는 boto3 SDK의 `create_agent` 함수를 사용하여 에이전트 생성 시 메모리 기능을 구성하는 방법을 보여줍니다. `enabledMemoryTypes`(현재 `SESSION_SUMMARY`만 지원)와 `storageDays`를 정의해야 합니다.

```python
    response = bedrock_agent_client.create_agent(
        agentName=agent_name,
        agentResourceRoleArn=agent_role['Role']['Arn'],
        description=agent_description,
        idleSessionTTLInSeconds=1800,
        foundationModel=agent_foundation_model,
        instruction=agent_instruction,
        memoryConfiguration={
            "enabledMemoryTypes": ["SESSION_SUMMARY"],
            "storageDays": 30
        }
    )
```
에이전트 호출 시에는 `memoryId`와 `sessionId` 매개변수를 함께 전달해야 합니다.
```python
    # 에이전트 API 호출
    agent_response = bedrock_agent_runtime_client.invoke_agent(
        inputText="사용자 질문",
        agentId="<AGENT_ID>",
        agentAliasId="<AGENT_ALIAS_ID>",
        sessionId="<SESSION_ID>",
        enableTrace=True or False,
        endSession=True or False,
        memoryId="메모리 ID",
        ...
    )
```

`endSession` 플래그를 `True`로 설정하여 에이전트를 호출하면, 해당 `sessionId`와 관련된 대화가 요약되어 에이전트 메모리에 저장됩니다. `bedrock-agent-runtime` boto3 SDK 클라이언트의 `get_agent_memory`를 사용하여 에이전트의 메모리 정보를 조회할 수 있습니다.

```python
    memory_content=bedrock_agent_runtime_client.get_agent_memory(
        agentAliasId="<AGENT_ALIAS_ID>",
        agentId="<AGENT_ID>",
        memoryId="<호출 시 사용한 메모리 ID>",
        memoryType='SESSION_SUMMARY'
    )
```

`delete_agent_memory`를 사용하여 특정 메모리 ID의 **전체** 에이전트 메모리를 삭제할 수 있습니다.

```python
    response = bedrock_agent_runtime_client.delete_agent_memory(
        agentAliasId="<AGENT_ALIAS_ID>", 
        agentId="<AGENT_ID>",
        memoryId="<호출 시 사용한 메모리 ID>",
    )
```