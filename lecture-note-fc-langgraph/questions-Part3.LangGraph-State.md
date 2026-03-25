참고
- 현재 질문들 모두는 [실습]_3_LangGraph의_다양한_State.ipynb 의 내용에 대한 질문임.

## Q1
```python
!pip install langgraph langchain langchain_google_genai langchain_community
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
이 코드는 LangGraph 실습에 필요한 주요 패키지들을 설치하는 명령어입니다.
- `langgraph`: LangChain 기반의 상태 기반 워크플로우를 구축하기 위한 라이브러리입니다.
- `langchain`: LLM 어플리케이션 개발을 위한 프레임워크입니다.
- `langchain_google_genai`: Google Gemini 모델을 LangChain에서 사용하기 위한 통합 패키지입니다.
- `langchain_community`: 커뮤니티에서 유지보수하는 다양한 외부 도구 및 통합 기능을 포함합니다.

<br/>

## Q2
```python
import os
os.environ['GOOGLE_API_KEY'] = 'AIxxx'

from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini API는 분당 10개 요청으로 제한
# 즉, 초당 약 0.167개 요청 (10/60)
rate_limiter = InMemoryRateLimiter(
    requests_per_second=0.167,  # 분당 10개 요청
    check_every_n_seconds=0.1,  # 100ms마다 체크
    max_bucket_size=10,  # 최대 버스트 크기
)

# rate limiter를 LLM에 적용
llm = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
이 코드는 Google Gemini API를 사용하기 위한 초기 설정 및 속도 제한(Rate Limiting)을 구현합니다.
1. `os.environ`: 환경 변수에 Google API 키를 설정합니다.
2. `InMemoryRateLimiter`: Gemini 무료 티어의 제한(분당 10회)을 준수하기 위해 초당 약 0.167회로 요청 속도를 제한하는 객체를 생성합니다.
3. `ChatGoogleGenerativeAI`: `gemini-2.5-flash` 모델을 초기화하며, 앞서 만든 `rate_limiter`를 적용하여 API 호출 안정성을 확보합니다. `thinking_budget`은 모델의 추론 프로세스에 할당할 토큰 한도를 설정합니다.

<br/>

## Q3
```python
from pydantic import BaseModel, Field

# 프롬프트 자동 생성을 위한 요소 저장
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
Pydantic을 사용하여 구조화된 출력(Structured Output)을 위한 데이터 모델을 정의합니다.
- `Objective` 클래스는 프롬프트 엔지니어링에 필요한 4가지 핵심 요소(`instruction`, `output_format`, `examples`, `notes`)를 필드로 가집니다.
- `@property`로 정의된 `as_str` 메서드는 각 필드의 키와 값을 "## Key \n Value" 형식의 마크다운 문자열로 변환하여, 이후 프롬프트의 배경 정보로 쉽게 주입할 수 있도록 돕습니다.

<br/>

## Q4
```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate([
    ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
    ('user', '{instruction}')

])

chain = prompt | llm.with_structured_output(Objective)

result = chain.invoke("집에서 쉽게 구할 수 있는 재료로 재미있는 장난감 만들기")

result
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
사용자의 단순한 지시 사항을 입력받아 `Objective` 스키마에 맞는 구조화된 데이터를 생성하는 체인입니다.
- `ChatPromptTemplate`: 시스템 메시지를 통해 모델의 역할을 부여합니다.
- `llm.with_structured_output(Objective)`: 모델이 일반 텍스트가 아닌, 앞서 정의한 `Objective` 클래스 구조의 JSON 형식 데이터를 반환하도록 강제합니다.
- `chain.invoke`: "장난감 만들기"라는 입력을 주면 모델은 이를 분석하여 세부 지침, 출력 형식, 예시, 주의사항이 포함된 객체를 생성합니다.

<br/>

## Q5
```python
print(result.as_str)
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
`Objective` 객체의 `as_str` 속성을 사용하여 생성된 결과를 마크다운 형식의 문자열로 출력합니다. 이를 통해 모델이 생성한 구조화된 데이터가 실제 프롬프트 템플릿에 어떻게 들어갈지 확인할 수 있습니다.

<br/>

## Q6
```python
from typing import TypedDict

class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
LangGraph에서 사용할 전체 상태(State)의 구조를 정의합니다.
- `TypedDict`를 상속받은 `State` 클래스는 그래프 노드 간에 전달될 데이터 바구니 역할을 합니다.
- 특징적인 점은 `prompt_materials` 필드에 단순한 문자열이 아닌, 앞서 정의한 Pydantic 모델인 `Objective` 객체를 통째로 저장하여 풍부한 문맥 정보를 유지한다는 것입니다.

<br/>

## Q7
```python
def get_prompt_materials(State):
    prompt = ChatPromptTemplate([
        ('system', '아래의 작업을 보다 자세하게 세분화하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
        ('user', '{instruction}')

    ])

    chain = prompt | llm.with_structured_output(Objective)

    result = chain.invoke({'instruction':State['instruction']})
    return {'prompt_materials' : result}
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
그래프의 첫 번째 노드 역할을 하는 함수입니다.
- 입력받은 `instruction`을 기반으로 `Objective` 구조의 데이터를 생성합니다.
- 생성된 결과는 `State`의 `prompt_materials` 키에 업데이트됩니다.

<br/>

## Q8
```python
from langchain_core.output_parsers import StrOutputParser

def generate_prompt(State):
    prompt = ChatPromptTemplate([
        ('system', '''당신은 체계적이고 정확한 프롬프트 엔지니어입니다. 아래의 포인트를 바탕으로, LLM에 입력할  시스템 프롬프트를 작성하세요.
{points}'''),
        ('user', '{instruction}')
    ])

    chain = prompt | llm | StrOutputParser()

    result = chain.invoke({'instruction': State['instruction'], 'points': State['prompt_materials'].as_str})
    return {'full_prompt' : result}
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
두 번째 노드 함수로, 정교한 시스템 프롬프트를 작성하는 역할을 합니다.
- 이전 단계에서 생성된 `State['prompt_materials'].as_str`를 `points` 변수로 전달받습니다.
- 전문 프롬프트 엔지니어로서의 지침과 세부 포인트들을 조합하여, 최종적으로 LLM이 수행할 '완성된 프롬프트'를 생성하고 이를 `full_prompt` 키에 저장합니다.

<br/>

## Q9
```python
def generate(State):
    return {'result' : llm.invoke(State['full_prompt']).content}
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
세 번째 노드 함수로, 이전 단계에서 완성된 `full_prompt`를 사용하여 최종 결과를 생성합니다.
- 모델에게 정교하게 짜여진 프롬프트를 입력으로 주고, 그 응답 내용을 `result` 키에 저장하여 반환합니다.

<br/>

## Q10
```python
from IPython.display import Image, display
from langgraph.graph import StateGraph, START, END


# 그래프 구성
builder = StateGraph(State)

builder.add_node("get_prompt_materials", get_prompt_materials)
builder.add_node("generate_prompt", generate_prompt)
builder.add_node("generate", generate)

builder.add_edge(START, "get_prompt_materials")
builder.add_edge("get_prompt_materials", "generate_prompt")
builder.add_edge("generate_prompt", "generate")

builder.add_edge("generate", END)
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
LangGraph를 사용하여 노드와 엣지를 연결하고 워크플로우를 정의합니다.
1. `StateGraph(State)`: 정의한 상태 구조를 기반으로 그래프 빌더를 생성합니다.
2. `add_node`: 위에서 정의한 3개의 함수를 그래프의 노드로 등록합니다.
3. `add_edge`: 데이터의 흐름을 정의합니다. `START` -> `get_prompt_materials` -> `generate_prompt` -> `generate` -> `END` 순서대로 실행되도록 선형적인 구조를 만듭니다.

<br/>

## Q11
```python
graph = builder.compile()
graph
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
정의된 그래프 설정을 컴파일하여 실행 가능한 `graph` 객체로 만듭니다. 컴파일 과정에서 그래프의 구조적 무결성(모든 노드와 엣지가 유효한지 등)을 검증합니다.

<br/>

## Q12
```python
import pprint

# Streaming 참고
# https://langchain-ai.github.io/langgraph/concepts/streaming/#streaming-graph-outputs-stream-and-astream

for data in graph.stream({'instruction': '''영화 '마이너리티 리포트'와 AI 윤리의 연관성에 대한 리포트 쓰기'''},
                         stream_mode='values'):
    pprint.pprint(data)
    print('----')
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
그래프를 실행하고 그 과정을 스트리밍 방식으로 확인합니다.
- `graph.stream`: 입력값과 함께 스트리밍 모드를 `values`로 설정하면, 각 노드가 실행될 때마다 변화하는 `State`의 전체 값을 반환합니다.
- 이를 통해 '리포트 쓰기'라는 요청이 각 노드를 거치며 어떻게 구체화되고(materials), 프롬프트가 만들어지며(full_prompt), 최종 결과물(result)이 도출되는지 단계별로 추적할 수 있습니다.

<br/>

## Q13
```python
data
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
스트리밍 반복문이 끝난 후의 마지막 `data` 변수 내용을 확인합니다. 여기에는 모든 노드의 실행이 완료된 시점의 최종 상태값이 담겨 있습니다.

<br/>

## Q14
```python
from IPython.display import display
from IPython.display import Markdown
import textwrap

def to_markdown(text):
  text = text.replace('•', '  *')
  return Markdown(textwrap.indent(text, '> ', predicate=lambda _: True))

to_markdown(data['result'])
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
모델이 생성한 최종 결과물(`data['result']`)을 마크다운 형식으로 예쁘게 출력하기 위한 헬퍼 함수입니다. `IPython.display.Markdown`을 사용하여 주피터 노트북 환경에서 가독성 높은 형태로 렌더링합니다.

<br/>

## Q15
```python
from typing import Annotated
from langgraph.graph.message import add_messages
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

# add_messages: 메시지를 계속 뒤에 추가하는 방식
# 기존 메시지를 수정하거나, 삭제하는 것도 가능합니다.

class State(TypedDict):
    context : Annotated[list, add_messages]
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
메시지 리스트를 관리하기 위한 특수한 상태 구조를 정의합니다.
- `Annotated[list, add_messages]`: 이것은 LangGraph의 핵심 기능 중 하나로, `context` 필드에 새로운 메시지가 들어오면 기존 리스트를 덮어쓰는 것이 아니라 `add_messages` 함수를 통해 **누적(append)**하거나 특정 ID를 가진 메시지를 수정/삭제하도록 지시합니다. 대화 이력 관리에 필수적인 설정입니다.

<br/>

## Q16
```python
def talk(State):
    return {'context': AIMessage(content='AI 메시지 2')}


builder = StateGraph(State)
builder.add_node('talk',talk)
builder.add_edge(START, 'talk')
builder.add_edge('talk', END)
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
메시지 기반의 간단한 그래프를 구성합니다.
- `talk` 노드는 단순히 새로운 `AIMessage`를 반환합니다.
- `State`에서 `add_messages` 리듀서를 설정했기 때문에, 이 노드가 실행되면 기존의 대화 내역 뒤에 이 메시지가 자동으로 추가됩니다.

<br/>

## Q17
```python
graph = builder.compile()
graph
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
메시지 기반 그래프를 컴파일합니다.

<br/>

## Q18
```python
messages = [
    SystemMessage(content='시스템 메시지 1'),
    HumanMessage(content='유저 메시지 1'),
    AIMessage(content='AI 메시지 1'),
    HumanMessage(content='유저 메시지 2'),
]

response = graph.invoke({'context': messages})
response
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
그래프를 실행하며 초기 메시지 리스트를 전달합니다.
- `messages`에는 이미 4개의 대화 내역이 들어 있습니다.
- `graph.invoke`를 통해 이를 입력하면, `talk` 노드가 반환한 'AI 메시지 2'가 더해져 총 5개의 메시지가 포함된 `context` 결과를 받게 됩니다.

<br/>

## Q19
```python
from langchain_core.messages import RemoveMessage
def delete_message(State):
    # 첫번째,두번째 메시지 삭제
    messages = State['context']
    return {"context": [RemoveMessage(id = messages[i].id) for i in range(1,3)]}
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
`RemoveMessage`를 사용하여 상태에서 특정 메시지를 삭제하는 노드 함수입니다.
- `add_messages` 리듀서는 `RemoveMessage` 객체를 받으면 해당 `id`를 가진 메시지를 상태에서 제거합니다.
- 이 코드는 리스트의 1번, 2번 인덱스에 해당하는 메시지들을 삭제 대상으로 지정하여 반환합니다.

<br/>

## Q20
```python
builder = StateGraph(State)

builder.add_node('talk',talk)
builder.add_node('delete_message',delete_message)

builder.add_edge(START, 'talk')
builder.add_edge('talk', 'delete_message')
builder.add_edge('delete_message', END)
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
메시지 생성(`talk`) 후 특정 메시지를 삭제(`delete_message`)하는 흐름을 가진 그래프를 재구성합니다.

<br/>

## Q21
```python
graph = builder.compile()
graph
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
삭제 로직이 포함된 그래프를 컴파일합니다.

<br/>

## Q22
```python
messages
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
원래의 `messages` 변수 내용을 다시 확인합니다.

<br/>

## Q23
```python
# 유저 메시지 1, AI 메시지 1 삭제

graph.invoke({'context': messages})
```
위 코드를 설명하세요. 바로 아래의 ### A 작성하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
새로운 그래프를 실행합니다.
- 초기 4개의 메시지가 입력되고, `talk` 노드에서 1개가 추가됩니다.
- 그 후 `delete_message` 노드가 실행되면서 지정된 인덱스의 메시지 2개가 삭제됩니다.
- 최종 결과에는 추가된 메시지는 포함되고, 삭제된 메시지는 제외된 상태의 컨텍스트가 남게 됩니다.
