
## langchain 의 tool
```python
from langchain_core.tools import tool

@tool
def search_blogs(query: str, display : int = 10, sort : Literal['sim', 'date'] = 'sim') -> list:
    """네이버 블로그 검색을 수행하여 검색 결과를 리스트로 반환합니다.
    query: 검색어
    display: 검색 결과 개수
    sort: sim(관련도순), date(시간순)
    """

    client_id = headers['X-Naver-Client-Id']
    client_secret = headers['X-Naver-Client-Secret']

    encText = urllib.parse.quote(query)
    url = f"https://openapi.naver.com/v1/search/blog?query={encText}&display={display}"
    ## ...
```

LangChain 의 tools 에 대해 설명과 위 코드에 대해 설명하세요. 바로 아래의 ### A 에 작성하세요. 질문 내용은 수정하거나 삭제하지 마세요.

### A

LangChain의 **Tools**는 LLM이 외부 세계와 소통할 수 있게 해주는 인터페이스입니다. 단순히 텍스트를 생성하는 모델에서 벗어나 실질적인 동작(웹 검색, 데이터베이스 조회, 계산 등)을 수행하는 '에이전시'를 부여합니다.

#### 1. `@tool` 데코레이터의 역할
위 코드에서 사용된 `@tool` 데코레이터는 일반 파이썬 함수를 LangChain의 도구(BaseTool) 객체로 변환합니다. 이때 LangChain은 함수의 **이름(name)**, **타입 힌트(args_schema)**, 그리고 **독스트링(description)**을 바탕으로 LLM에게 제공할 메타데이터를 자동으로 생성합니다.

#### 2. 코드 상세 분석 (`search_blogs`)
제공된 코드는 네이버 오픈 API를 호출하여 블로그를 검색하는 함수입니다.

*   **메타데이터 추출**: LLM은 하단의 독스트링(`"""네이버 블로그 검색을 수행하여..."""`)을 읽고, 언제 이 도구를 호출해야 할 지 스스로 판단합니다.
*   **파라미터 정의**: `query`, `display`, `sort` 매개변수에 지정된 타입 힌트와 기본값은 LLM이 도구를 실행할 때 생성해야 할 데이터 형식을 규정합니다.
*   **실제 로직**: 내부적으로 `headers`와 API URL을 구성하여 검색을 수행하며, 그 결과값(list)은 다시 LLM에게 전달되어 최종 응답을 완성하는 데 사용됩니다.

#### 3. 에이전트 워크플로우에서의 의미
LangGraph와 같은 프레임워크에서 이 도구가 등록되면, LLM은 사용자 질문의 맥락에 따라 이 함수를 호출할지 결정합니다. 예를 들어 "아이폰 16 후기 블로그에서 찾아줘"라는 질문이 들어오면 LLM은 `search_blogs(query='아이폰 16 후기')`를 호출하도록 실행 계획을 세우게 됩니다.

#### 4. 독스트링(Docstring)이 중요한 이유
가장 중요한 질문입니다. 만약 독스트링이 없다면 **도구(Tool)의 설명(description)이 부실해지거나 비어 있게 됩니다.**

*   **판단 근거 상실**: LLM은 "이 도구가 구체적으로 어떤 일을 하는가?"를 알 방법이 없습니다. 사용자의 요청(예: 블로그 검색)이 들어왔을 때 이 도구를 호출해야 할지 말지 스스로 판단할 근거가 사라집니다.
*   **오작동 유발**: 설명이 없으면 모델이 엉뚱한 상황에서 도구를 실행하거나, 반대로 도구가 꼭 필요한 상황에서도 이를 무시하게 됩니다. 
*   **인자 생성 오류**: `query`, `display` 등의 인자가 무엇을 의미하는지 독스트링에 상세히 작성하지 않으면 LLM이 인잣값을 채워 넣을 때 기대와 다른 엉뚱한 데이터를 전달할 가능성이 매우 커집니다.

결론적으로, LangChain 에이전트 개발 시 **명확한 독스트링을 작성하는 것**은 모델이 도구의 정체성을 파악하도록 도와주는 '눈'과 같은 필수 요소입니다.


## typing_extensions 패키지에 대해

```python
from typing_extensions import TypedDict, Literal, Annotated

headers = {
    'X-Naver-Client-Id': naver_username,
    'X-Naver-Client-Secret': naver_api_token
}

@tool
def search_blogs(query: str, display : int = 10, sort : Literal['sim', 'date'] = 'sim') -> list:
    """네이버 블로그 검색을 수행하여 검색 결과를 리스트로 반환합니다.
    query: 검색어
    display: 검색 결과 개수
    sort: sim(관련도순), date(시간순)
    """

    client_id = headers['X-Naver-Client-Id']
    client_secret = headers['X-Naver-Client-Secret']

    ## ...

```

python 의 `typing_extensions` 패키지에 대한 설명과 `TypedDict`, `Liternal`, `Annotated` 에 대한 설명을 작성하세요. 바로 아래의 ### A 에 작성하세요. 질문 내용은 수정하거나 삭제하지 마세요.

### A

`typing_extensions`는 파이썬 표준 라이브러리인 `typing` 모듈에 새롭게 추가된 최신 타입 시스템 기능들을 이전 버전의 파이썬에서도 사용할 수 있도록 제공하는 **백포트(Backport)** 패키지입니다. LangGraph와 같이 최신 파이썬의 타입 시스템을 적극 활용하는 프레임워크에서 필수적으로 사용됩니다.

#### 1. `TypedDict`
딕셔너리의 **키(Key)**와 해당 키에 할당될 **값(Value)의 타입**을 명확하게 정의할 때 사용합니다.
- 일반 `dict`는 데이터 구조를 강제할 수 없지만, `TypedDict`를 사용하면 특정 키가 반드시 존재해야 함을 명시할 수 있습니다.
- LangGraph에서는 그래프의 **상태(State)** 객체가 어떤 필드를 가져야 하는지 정의하는 용도로 쓰입니다.

#### 2. `Literal`
변수나 인자가 가질 수 있는 값을 **특정한 상수 값들**로 엄격히 제한합니다.
- 예시 코드의 `sort: Literal['sim', 'date']`는 해당 인자에 `'sim'` 또는 `'date'`라는 두 가지 문자열만 허용됨을 의미합니다.
- LLM에게 도구를 설명할 때 선택 가능한 옵션을 명확히 규정하여 오류를 줄이는 데 매우 효과적입니다.

#### 3. `Annotated`
기존의 타입 힌트에 **추가적인 메타데이터**를 결합할 수 있는 기능입니다.
- 기본 형식: `Annotated[기본 타입, 메타데이터]`
- LangGraph에서 `Annotated[list[BaseMessage], add_messages]`와 같이 사용하여, 특정 상태 값이 업데이트될 때 어떻게 처리할지(예: 리스트에 추가할지 여부) 결정하는 **Reducer 함수**를 연결하는 용도로 핵심적인 역할을 수행합니다.



## Literal
참고: Human In The Loop 챕터 예제
```python
from typing_extensions import TypedDict, Literal, Annotated

## ...

def route_after_llm(state) -> Literal[END, "get_user_input", "human_review"]:

    last_message = state['messages'][-1]
    # 마지막 메시지: tool calling
    # 2025.10.26 업데이트: Gemini의 Thinking 모델(2.5 이후)에는
    # Tool Calling 이후의 messages에 signature가 포함되어 형식이 달라집니다.
    last_message_content = last_message.content

    if not last_message.tool_calls:
        # if '감사합니다! 챗봇을 종료합니다!' in last_message_content:
        if '챗봇을 종료' in last_message_content: ## 어쩌다 가끔은 llm 이 '감사합니다! 챗봇을 종료합니다!!' 같은 메시지가 아니라 유사한 메시지로 응답하는 경우가 있기에 '챗봇을 종료'라는 단어가 있는지 검사해야 할 경우도 있다.
            return END
        elif isinstance(last_message_content[0], dict) and 'text' in last_message_content[0] and '감사합니다! 챗봇을 종료합니다!' in last_message_content[0]['text']:
            return END
        return 'get_user_input'
    else:
        return "human_review"
```

위 코드에서 `Literal[END, "get_user_input", "human_review"]` 는 무엇을 의미하나요? 위 코드의 전반적인 내용 역시 설명하세요. 바로 아래의 ### A 에 작성하세요. 질문 내용은 수정하거나 삭제하지 마세요.

### A

`Literal[END, "get_user_input", "human_review"]`는 이 함수가 **반드시 정의된 특정 값들 중 하나만 반환함**을 명시하는 타입 힌트입니다. LangGraph의 **조건부 에지(Conditional Edge)** 설정에서 그래프가 다음에 이동할 수 있는 경로를 사전에 정의하고 검증하는 중요한 역할을 합니다.

#### 1. `Literal[...]`의 의미와 역할
- **경로 정의**: 함수가 임의의 값이 아닌, 오직 `END`(그래프 종료), `"get_user_input"`, `"human_review"` 중 하나만 반환하도록 강제하여 워크플로우를 안전하게 보호합니다.
- **가시성 확보**: LangGraph 프레임워크는 이 정보를 바탕으로 그래프의 전반적인 흐름을 파악하며 시각화에도 활용합니다.

#### 2. `route_after_llm` 함수 로직 설명
이 함수는 LLM이 응답한 메시지의 성격에 따라 다음에 수행할 작업을 결정하는 '교통 관제' 역할을 담당합니다.

1.  **메시지 분석**: 상태(state)의 최신 메시지를 확인하여 LLM이 도구 호출(`tool_calls`)을 요청했는지, 아니면 일반 답변을 했는지 파악합니다.
2.  **도구 호출 시 (Human-in-the-loop)**: 만약 LLM이 도구 사용을 요청했다면, 즉시 실행하는 대신 사용자의 검토를 거치기 위해 `"human_review"` 노드로 경로를 보냅니다.
3.  **일반 응답 시 (대화 지속 또는 종료)**:
    - **종료 판단**: 응답 내용에 "챗봇을 종료"라는 특정 단어가 있거나, 최신 Gemini 모델(2.5 이후)의 특수한 구조 내에 종료 의사가 있을 경우 `END`를 반환하여 그래프를 안전하게 마무리합니다.
    - **대화 지속**: 종료 조건이 아니라면 사용자 답변을 더 받기 위해 `"get_user_input"` 노드로 연결합니다.

결론적으로 이 코드는 **LLM의 의도를 판별하여 적절한 워크플로우 노드로 안전하게 연결해주는 관제 탑**의 역할을 수행합니다.


## `route_after_llm` 관련
예제에서는 왜 `route_after_llm`은 graph 내에 노드로 표시되지 않나요?
```python
builder = StateGraph(State)
builder.add_node(get_user_input)
builder.add_node(agent)
builder.add_node(run_tool)
builder.add_node(human_review)

builder.add_edge(START, "get_user_input")
builder.add_edge('get_user_input', "agent")
builder.add_conditional_edges("agent", route_after_llm) 
## graph 를 보면 route_after_llm 은 노드로 나타나지 않는다.
## route_after_llm 은 node 로 등록하지 않고, 
builder.add_edge("run_tool", "agent")

memory = MemorySaver()

graph = builder.compile(checkpointer=memory)
# 중간 상태 저장을 위해 체크포인터가 필요합니다!
graph
```
<br/>

![alt text](<CleanShot 2026-03-28 at 12.52.15@2x.png>)

위 코드에서 보듯 우리 예제에서 `route_after_llm`은 노드로 등록하지 않았었다. 따라서 그래프에서도 `route_after_llm` 은 노드로 보이지 않는다.<br/>

그리고 `route_after_llm` 은 "human_review", "get_user_input", "END" 중 하나를 return 하는 함수다. 이 값은 graph 내에서 노드의 이름과 일치하기에 builder 객체의 add_conditional_edges 메서드에 의해 해당 노드 명으로 분기가 되게 된다.<br/>

결론을 내려보면 노드 명을 명시하지 않고 동적으로 분기가 되도록 `route_after_llm`함수를 정의한 것이다.<br/>
<br/>

참고) Gemini 의 답변이 기발해서 발췌해옴<br/>
**`route_after_llm` 함수**가 바로 이 **세 갈래길 앞에 서 있는 교통 경찰**과 같습니다.<br/>
*   "에이전트님, 작업 끝나셨나요? 상태를 보니 도구를 써야겠네요. **`human_review` 화살표**로 가세요!"라고 안내하는 것입니다.<br/>
*   따라서 시각화에서는 `route_after_llm`이라는 박스가 생기는 게 아니라, **`agent` 노드에서 뻗어 나가는 여러 개의 화살표** 그 자체가 `route_after_llm`의 존재를 나타냅니다.<br/>
