
## 환경설정 및 Jupyter 설치
### 가상환경 세팅
```bash
conda create --name env-langgraph-fc python=3.10 -y
```
<br/>

### 세팅 확인
가상환경 디렉터리 확인
```bash
ls -al ~/miniforge3/envs/env-langgraph-fc 
```
<br/>

가상환경 활성화
```bash
conda activate env-langgraph-fc
```
<br/>

가상환경을 끌때는 다음과 같이 한다.
```bash
conda deactivate
```
<br/>

### 의존성 설정
먼저 jupyter 설치
```bash
# pip 대신 mamba를 사용하면 패키지 설치 속도가 훨씬 빠릅니다.
mamba install -c conda-forge jupyter notebook ipykernel -y
```
<br/>

다음 의존성 설치
```bash
## pip 를 이용해 설치
!pip install langgraph langchain langchain_google_genai langchain_community

## 또는 mamba 를 이용해 설치
mamba install langgraph langchain langchain_google_genai langchain_community
```
<br/>

keyring 라이브러리 설치
```bash
## pip 을 이용해 설치
pip install keyring

## 또는 mamba 를 이용해 keyring 설치
mamba install keyring
```
<br/>

### langgraph, langchain 을 위한 의존성 설치
```bash
! pip install langgraph langchain langchain_google_genai langchain_community
```
- 참고: mamba 로 설치할때는 에러가 난다.

<br/>

## keyring 으로 api key 등록
### api key 를 keyring 에 등록
```bash
## bash 쉘 에서 다음 내용을 입력
## 형식 keyring set {{서비스명}} {{계정명}}

## e.g.
keyring set gemini-api-key---alpha300uk alpha300uk  
Password for 'alpha300uk' in 'gemini-api-key---alpha300uk':
```

위의 key 는 사용 시에는 다음과 같이 사용 가능
```python
import keyring

service_name = "gemini-api-key---alpha300uk"
username = "alpha300uk"

# 키링에서 토큰 조회
api_token = keyring.get_password(service_name, username)
```
<br/>

### jupyter notebook 에 가상환경 커널 등록
```bash
~/miniforge3/envs/env-langgraph-fc/bin/python -m ipykernel install --user --name env-langgraph-fc --display-name "langgraph kernal (miniforge)"
```
<br/>


## e.g. (1)
```python
import keyring
import os

from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_google_genai import ChatGoogleGenerativeAI

from pydantic import BaseModel, Field

service_name = "gemini-api-key---alpha300uk"
username = "alpha300uk"

# 키링에서 토큰 조회
api_token = keyring.get_password(service_name, username)

# api_token 등록
os.environ['GOOGLE_API_KEY'] = api_token

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

# 프롬프트 자동 생성을 위한 요소 저장
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])


from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate([
    ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
    ('user', '{instruction}')

])

chain = prompt | llm.with_structured_output(Objective)

result = chain.invoke("집에서 쉽게 구할 수 있는 재료로 재미있는 장난감 만들기")
print(result.as_str)

"""
## instruction
 집에서 쉽게 구할 수 있는 재료를 활용하여 어린이들이 재미있게 즐길 수 있는 장난감 아이디어를 제안하고, 해당 장난감을 만드는 데 필요한 재료 목록과 상세한 제작 단계를 제공하세요. 장난감의 재미 요소와 안전성도 함께 고려하여 설명해야 합니다.

## output_format
 응답은 다음 JSON 포맷을 따라야 합니다: {"toy_name": "[장난감 이름]", "materials": ["재료1", "재료2", ...], "instructions": ["단계1", "단계2", ...], "fun_factor": "[재미 요소 설명]", "safety_notes": "[안전 유의사항]"}

## examples
 {  "toy_name": "휴지심 망원경",  "materials": ["휴지심 2개", "색종이", "가위", "풀", "끈"],  "instructions": ["휴지심 한쪽 끝을 가위로 1cm 정도 자르고 서로 끼워 넣습니다.", "색종이로 망원경 전체를 감싸 붙입니다.", "망원경 양쪽에 펀치로 구멍을 뚫고 끈을 연결하여 목에 걸 수 있게 합니다.", "색종이나 스티커로 망원경을 예쁘게 꾸밉니다."],  "fun_factor": "아이들은 망원경을 통해 주변을 탐험하며 상상력을 자극하고, 자신만의 비밀스러운 도구를 가진다는 재미를 느낄 수 있습니다.",  "safety_notes": "가위 사용 시 반드시 보호자의 지도가 필요하며, 끈이 너무 길지 않도록 주의하여 목에 걸었을 때 안전사고가 발생하지 않도록 합니다."}

## notes
 - 제안하는 장난감은 5세 이상 어린이가 안전하게 가지고 놀 수 있도록 고안되어야 합니다.- 재료는 가정에서 흔히 찾을 수 있는 것들(예: 휴지심, 종이컵, 신문지, 고무줄 등)로 제한해야 합니다.- 제작 단계는 초보자도 쉽게 따라 할 수 있도록 명확하고 간결하게 작성되어야 합니다.- 장난감의 재미 요소와 함께 발생할 수 있는 잠재적 위험 및 안전 유의사항을 반드시 포함해야 합니다.
"""
```

### 코드 설명

#### (1)
```python
from langchain_core.rate_limiters import InMemoryRateLimiter

# Gemini API는 분당 10개 요청으로 제한
# 즉, 초당 약 0.167개 요청 (10/60)
rate_limiter = InMemoryRateLimiter(
    requests_per_second=0.167,  # 분당 10개 요청
    check_every_n_seconds=0.1,  # 100ms마다 체크
    max_bucket_size=10,  # 최대 버스트 크기
)
```

위 코드에 대한 설명을 바로 아래의 '##### 설명'아래에 작성하세요

##### 설명
**`InMemoryRateLimiter`**는 API 호출의 빈도를 제어하여 속도 제한(Rate Limit) 오류를 방지하는 도구입니다. 특히 Gemini API의 무료 티어는 분당 요청 수(RPM) 제한이 엄격하므로 이를 설정하는 것이 권장됩니다.

- `requests_per_second=0.167`: 1초당 평균 허용 요청 수입니다. Gemini 무료 티어의 분당 10회 제한을 초당으로 환산한 값입니다 (10 RPM / 60초 ≈ 0.167).
- `check_every_n_seconds=0.1`: 0.1초마다 시스템이 요청을 보낼 수 있는 상태인지 확인합니다.
- `max_bucket_size=10`: 한꺼번에 보낼 수 있는 최대 요청 묶음(Burst)의 크기입니다.



#### (2) 
```python
from langchain_google_genai import ChatGoogleGenerativeAI

# rate limiter를 LLM에 적용
llm = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)
```

위 코드에 대한 설명을 바로 아래의 '##### 설명'아래에 작성하세요

##### 설명
생성된 LLM 인스턴스(`ChatGoogleGenerativeAI`)에 앞서 설정한 `rate_limiter`를 적용합니다.

- `model="gemini-2.5-flash"`: 사용할 구글의 AI 모델 이름입니다.
- `rate_limiter=rate_limiter`: LLM 호출 시 자동으로 설정된 속도 제한 로직을 따르도록 주입합니다.
- `thinking_budget=500`: 모델이 응답을 생성하기 전 수행하는 '추론(Reasoning)' 과정에 할당할 최대 토큰 예산입니다. 500 토큰 내에서 복잡한 논리를 검토한 뒤 최종 답변을 내놓게 됩니다.



#### (3)
```python
from pydantic import BaseModel, Field

## ...

# 프롬프트 자동 생성을 위한 요소 저장
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])


from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate([
    ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
    ('user', '{instruction}')

])

chain = prompt | llm.with_structured_output(Objective)

result = chain.invoke("집에서 쉽게 구할 수 있는 재료로 재미있는 장난감 만들기")
print(result.as_str)
```

- 위 코드에 대한 설명을 바로 아래의 '##### 설명'아래에 작성하세요.
- 위 코드 내에 `@property def as_str(self)` 표현에서 사용된 문법에 대해서도 '##### 설명'아래에서 설명하세요.
- 위 코드 내에 `chain = prompt | llm.with_structured_output(Objective)` 을 사용한 후에 `instruction`, `output_format`, `examples`, `notes` 필드가 생성된 것은 어떻게해서 생성된 것 일까요? 이런 instruction, output_format, examples, notes 필드 등과 같은 형식은 프롬프트 엔지니어링 분야에서 연구하는 패턴 들 중 하나일까요? 만약 그렇다면 어떤 패턴인지 설명해주세요.
- 위 코드 내에 `chain = prompt | llm.with_structured_output(Objective)` 표현에서 사용된 `|` 은 무엇인지 '##### 설명'아래에서 설명하세요.


##### 설명
이 코드는 LLM으로부터 **구조화된 출력(Structured Output)**을 받아오는 전형적인 LangChain 워크플로우를 보여줍니다.

- **`Objective(BaseModel)`**: Pydantic을 사용하여 LLM이 응답해야 할 데이터 형식을 클래스로 정의합니다. 각 필드의 `description`은 LLM이 해당 필드에 어떤 내용을 채워야 할지 이해하는 힌트가 됩니다.
- **`@property` (`as_str`)**: 파이썬의 데코레이터로, 메서드를 변수(속성)처럼 호출할 수 있게 해줍니다. 여기서는 클래스 내의 데이터들을 마크다운 형식(`## 제목\n 내용`)으로 보기 좋게 포맷팅하여 반환하도록 정의되었습니다.
- **`with_structured_output(Objective)`**: LLM이 일반 텍스트가 아닌, 우리가 정의한 `Objective` 클래스의 인스턴스를 반환하도록 강제하는 기능입니다.
- **`|` (Pipe Operator)**: **LCEL(LangChain Expression Language)**의 핵심 연산자로, 여러 컴포넌트를 연결하는 역할을 합니다. `prompt | llm`은 프롬프트의 결과값이 LLM의 입력값으로 자연스럽게 흘러 들어가도록 체인(Chain)을 형성하는 것을 의미합니다.



## e.g. (2)
```python
### 예전 코드 --- (start)
import keyring
import os

from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_google_genai import ChatGoogleGenerativeAI

from pydantic import BaseModel, Field

service_name = "gemini-api-key---alpha300uk"
username = "alpha300uk"

# 키링에서 토큰 조회
api_token = keyring.get_password(service_name, username)

# api_token 등록
os.environ['GOOGLE_API_KEY'] = api_token

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

# 프롬프트 자동 생성을 위한 요소 저장
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])


from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate([
    ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
    ('user', '{instruction}')

])
### 예전 코드 --- (end)

from typing import TypedDict

class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str


def get_prompt_materials(State):
    prompt = ChatPromptTemplate([
        ('system', '아래의 작업을 보다 자세하게 세분화하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
        ('user', '{instruction}')

    ])

    chain = prompt | llm.with_structured_output(Objective)

    result = chain.invoke({'instruction':State['instruction']})
    return {'prompt_materials' : result}


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

def generate(State):
    return {'result' : llm.invoke(State['full_prompt']).content}


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
graph = builder.compile()


import pprint

# Streaming 참고
# https://langchain-ai.github.io/langgraph/concepts/streaming/#streaming-graph-outputs-stream-and-astream

for data in graph.stream({'instruction': '''영화 '마이너리티 리포트'와 AI 윤리의 연관성에 대한 리포트 쓰기'''},
                         stream_mode='values'):
    pprint.pprint(data)
    print('----')
```

### 코드 설명

#### (1)
```python
from typing import TypedDict

class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str
```
#### 설명
**`State` 클래스**는 파이썬의 `TypedDict`를 상속받아 확장한 클래스가 맞습니다.

- **`TypedDict`**: 파이썬의 딕셔너리에 담길 '키(key)'와 그에 대응하는 '값(value)의 타입'을 명시적으로 정의할 수 있게 해주는 도구입니다. 일반적인 딕셔너리와 달리 어떤 키가 필수적으로 존재해야 하는지 타입을 통해 알려주므로, LangGraph와 같이 여러 단계(Node)를 거치며 공유 데이터(State)를 업데이트하는 시스템에서 데이터의 일관성을 유지하는 핵심 역할을 합니다.
- **`typing` 패키지**: 파이썬 3.5부터 도입된 표준 라이브러리로, 코드 상에 타입 정보를 명시할 수 있는 다양한 도구(List, Dict, Union, Optional, TypedDict 등)를 제공합니다. 파이썬은 기본적으로 동적 타이핑 언어이지만, 이 패키지를 활용하면 코드의 가독성이 높아지고 IDE(VS Code, PyCharm 등)의 자동 완성이나 정적 분석 도구(MyPy)를 통해 실행 전 오류를 효과적으로 예방할 수 있습니다.

<br/>

#### (2)
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
#### 설명
이 함수는 그래프의 첫 번째 **노드(Node)** 역할을 수행합니다.

- **입력**: 현재 그래프의 상태(`State`) 객체를 인자로 받습니다.
- **로직**: 이전에 정의한 `Objective` 클래스를 사용하여 사용자의 초기 지시사항(`instruction`)으로부터 필요한 프롬프트 재료들을 분석하고 구조화된 데이터를 생성합니다.
- **출력**: `{'prompt_materials' : result}`와 같이 업데이트가 필요한 키-값 쌍을 담은 딕셔너리를 반환합니다. LangGraph는 이 반환값을 받아 전체 상태(`State`)의 해당 키값을 자동으로 갱신합니다.

<br/>

#### (3)
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
#### 설명
두 번째 노드로, 이전 단계에서 생성된 **`prompt_materials`**를 활용하여 더 구체적인 프롬프트를 만드는 과정입니다.

- **`.as_str`**: 앞서 `Objective` 클래스에서 정의했던 프로퍼티를 호출하여, 구조화된 데이터를 마크다운 형식의 문자열로 변환해 사용합니다.
- **`StrOutputParser`**: 모델의 답변 중에서 불필요한 메타데이터를 제외하고 실제 텍스트 내용만 문자열로 뽑아내어 `full_prompt`를 완성합니다.

<br/>

#### (4)
```python
def generate(State):
    return {'result' : llm.invoke(State['full_prompt']).content}
```
#### 설명
마지막 처리 노드로, 이전 단계에서 완성된 `full_prompt`를 바탕으로 LLM을 호출하여 최종 결과물을 생성합니다. 이 결과물은 `State`의 `result` 키에 저장됩니다.

<br/>

#### (5)
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
graph = builder.compile()


import pprint

# Streaming 참고
# https://langchain-ai.github.io/langgraph/concepts/streaming/#streaming-graph-outputs-stream-and-astream

for data in graph.stream({'instruction': '''영화 '마이너리티 리포트'와 AI 윤리의 연관성에 대한 리포트 쓰기'''},
                         stream_mode='values'):
    pprint.pprint(data)
    print('----')
```
#### 설명
워크플로우 전체를 조립하고 실행하는 코드입니다.

1. **`StateGraph(State)`**: 앞에서 정의한 `State` 구조를 가진 그래프를 초기화합니다.
2. **`add_node`**: 파이썬 함수들을 그래프의 작업 단위인 '노드'로 등록합니다.
3. **`add_edge`**: 노드 간의 흐름을 정의합니다. `START` 예약어로 시작 위치를 정하고, 노드들 사이를 순차적으로 연결한 뒤 `END`로 마무리합니다.
4. **`compile()`**: 설정한 노드와 엣지를 바탕으로 실행 가능한 그래프 객체를 생성합니다.
5. **`graph.stream(..., stream_mode='values')`**: 그래프를 실행하면서 단계별로 업데이트되는 전체 상태(`State`)의 실시간 값을 Yield 합니다. `stream_mode='values'`를 설정하면 매 단계가 끝날 때마다 누적된 전체 상태 값을 확인할 수 있어 디버깅과 진행 상황 파악에 매우 유리합니다.

## e.g. (3) Message 포맷의 State
```python
### 예전 코드 --- (start)
import keyring
import os

from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_google_genai import ChatGoogleGenerativeAI

from pydantic import BaseModel, Field

service_name = "gemini-api-key---alpha300uk"
username = "alpha300uk"

# 키링에서 토큰 조회
api_token = keyring.get_password(service_name, username)

# api_token 등록
os.environ['GOOGLE_API_KEY'] = api_token

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

# 프롬프트 자동 생성을 위한 요소 저장
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])


from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate([
    ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
    ('user', '{instruction}')

])
### 예전 코드 --- (end)


### 이번 코드 --- (start)

import pprint

# Streaming 참고
# https://langchain-ai.github.io/langgraph/concepts/streaming/#streaming-graph-outputs-stream-and-astream
def pprint_graph(graph):
    for data in graph.stream({'instruction': '''영화 '마이너리티 리포트'와 AI 윤리의 연관성에 대한 리포트 쓰기'''},
                            stream_mode='values'):
        pprint.pprint(data)
        print('----')

#### (1) : start
from typing import Annotated
from typing import TypedDict
from IPython.display import Image, display
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    context: Annotated[list, add_messages]
#### (1) : end

#### (2) : start
builder = StateGraph(State)
builder.add_node('talk',talk)
builder.add_edge(START, 'talk')
builder.add_edge('talk', END)

graph = builder.compile()
pprint_graph(graph)

messages = [
    SystemMessage(content='시스템 메시지 1'),
    HumanMessage(content='유저 메시지 1'),
    AIMessage(content='AI 메시지 1'),
    HumanMessage(content='유저 메시지 2'),
]

response = graph.invoke({'context': messages})
#### (2) : end
# print(response)

#### (3) : start
### 메시지 삭제 
from langchain_core.messages import RemoveMessage
def delete_message(State):
    # 첫번째,두번째 메시지 삭제
    messages = State['context']
    return {"context": [RemoveMessage(id = messages[i].id) for i in range(1,3)]}


builder = StateGraph(State)

builder.add_node('talk',talk)
builder.add_node('delete_message',delete_message)

builder.add_edge(START, 'talk')
builder.add_edge('talk', 'delete_message')
builder.add_edge('delete_message', END)
graph = builder.compile()
# print(graph)
pprint_graph(graph)

# print(messages)

# 유저 메시지 1, AI 메시지 1 삭제
graph.invoke({'context': messages})
### 이번 코드 --- (end)
#### (3) : end
```

### 코드 설명
#### (1)
```python
from typing import Annotated
from typing import TypedDict

class State(TypedDict):
    context: Annotated[list, add_messages]
```

위 코드에 대한 설명을 바로 아래의 ##### A 에 작성하세요. Annotated 는 무엇인지, TypedDict 는 무엇인지 위에서 정의한 State 클래스에 대해서도 설명하세요.

##### A
- **`TypedDict`**: 파이썬의 `typing` 모듈에서 제공하며, 딕셔너리의 '키'와 '값의 타입'을 명시할 수 있게 해줍니다. LangGraph에서는 그래프가 공유하는 상태(State)의 구조를 정의할 때 필수적으로 사용됩니다.
- **`Annotated`**: 기존 타입에 부가적인 메타데이터를 추가할 수 있는 도구입니다. 이 코드에서는 `list` 타입에 `add_messages`라는 **리듀서(Reducer)** 기능을 연결하는 역할을 합니다.
- **`add_messages`**: LangGraph에서 제공하는 특수 함수로, 새로운 메시지가 들어왔을 때 기존 리스트를 덮어쓰지 않고 뒤에 이어 붙이거나(append), 특정 ID를 가진 메시지를 수정하거나 삭제하는 역할을 수행합니다.
- **`State` 클래스**: 이 그래프에서 관리할 데이터가 `context`라는 키를 가지며, 그 값은 메시지 객체들의 리스트임을 정의하고 있습니다. `add_messages` 덕분에 대화 기록이 차곡차곡 쌓이게 됩니다.



#### (2)
```python
## ...
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    ## ...

builder = StateGraph(State)
builder.add_node('talk',talk)
builder.add_edge(START, 'talk')
builder.add_edge('talk', END)

graph = builder.compile()
pprint_graph(graph)

messages = [
    SystemMessage(content='시스템 메시지 1'),
    HumanMessage(content='유저 메시지 1'),
    AIMessage(content='AI 메시지 1'),
    HumanMessage(content='유저 메시지 2'),
]

response = graph.invoke({'context': messages})
```

위 코드에 대한 설명을 바로 아래의 ##### A 에 작성하세요. 

##### A
- **`StateGraph(State)`**: 위에서 정의한 `State` 구조를 기반으로 그래프를 생성하기 위한 빌더(Builder)를 초기화합니다.
- **`add_node` / `add_edge`**: 'talk'라는 이름의 작업 단위(Node)를 등록하고, 시작(`START`)부터 'talk'를 거쳐 종료(`END`)까지의 흐름(Edge)을 정의합니다.
- **`compile()`**: 노드와 엣지의 구성을 검토하여 실제로 실행 가능한 그래프 객체로 변환합니다.
- **`graph.invoke({'context': messages})`**: 초기 메시지 리스트를 담아 그래프를 실행합니다. 이때 `State` 정의에 사용된 `add_messages` 로직에 따라 입력된 메시지들이 그래프 상태에 안전하게 등록됩니다.




#### (3)
```python
#### (3) : start
### 메시지 삭제 
from langchain_core.messages import RemoveMessage
def delete_message(State):
    # 첫번째,두번째 메시지 삭제
    messages = State['context']
    return {"context": [RemoveMessage(id = messages[i].id) for i in range(1,3)]}


builder = StateGraph(State)

builder.add_node('talk',talk)
builder.add_node('delete_message',delete_message)

builder.add_edge(START, 'talk')
builder.add_edge('talk', 'delete_message')
builder.add_edge('delete_message', END)
graph = builder.compile()
# print(graph)
pprint_graph(graph)

# print(messages)

# 유저 메시지 1, AI 메시지 1 삭제
graph.invoke({'context': messages})
```

위 코드에 대한 설명을 바로 아래의 ##### A 에 작성하세요. 

##### A
- **`RemoveMessage`**: LangChain에서 제공하는 특수한 메시지 타입으로, 상태(State)에서 특정 메시지를 **삭제**하고 싶을 때 사용합니다. `id` 값을 지정하여 삭제 대상을 식별합니다.
- **`delete_message(State)`**: 이 함수는 현재 상태에 저장된 메시지들 중 인덱스 1번과 2번에 해당하는 메시지의 ID를 추출하여 `RemoveMessage` 객체 리스트를 반환합니다.
- **메시지 삭제 원리**: 그래프의 상태가 업데이트될 때, `add_messages` 리듀서는 `RemoveMessage`를 감지하면 해당 ID의 메시지를 리스트에서 실제로 제거합니다.
- 결과적으로 이 워크플로우를 거치면 대화 기록 중 불필요하거나 용량이 큰 메시지를 선택적으로 지워 상태를 효율적으로 관리할 수 있습니다.


쳐웃지 말고 나가라. 아침에 뭐하는 거냐.
어제는 클로드 모델 비싸다고 구글 플래시 모델이 혜자라는 일기 쓸때도 평가질하고 있던데
뭐냐? 할일이 그렇게 없냐? 나 말고 다른 사람 캐면 안되냐? 몇년째야. 