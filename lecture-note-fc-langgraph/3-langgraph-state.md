
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
    model="gemini-3-flash-preview",
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
    model="gemini-3-flash-preview",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)
```

위 코드에 대한 설명을 바로 아래의 '##### 설명'아래에 작성하세요

##### 설명
생성된 LLM 인스턴스(`ChatGoogleGenerativeAI`)에 앞서 설정한 `rate_limiter`를 적용합니다.

- `model="gemini-3-flash-preview"`: 사용할 구글의 AI 모델 이름입니다.
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
이 코드는 LLM으로부터 **구조화된 데이터(Structured Output)**를 추출하여 전문적인 프롬프트 구성 요소를 자동으로 생성하는 핵심 로직을 보여줍니다.

- **`@property def as_str(self)` 문법**:
  - 파이썬의 **데코레이터(Decorator)** 중 하나로, 클래스의 메서드를 마치 **일반 속성(변수)**처럼 접근할 수 있게 해줍니다. 
  - 외부에서 `result.as_str()`과 같이 괄호를 쓰는 대신 `result.as_str`만으로 값을 가져올 수 있어 코드가 간결해지며, 주로 내부 데이터를 가공하여 속성처럼 노출할 때 사용합니다.

- **필드 생성 원리 및 프롬프트 엔지니어링 패턴**:
  - **생성 원리**: Pydantic의 `Objective` 모델 정보를 `llm.with_structured_output(Objective)`에 주입하면, LangChain은 내부적으로 이 모델의 스키마(필드명, 타입, 설명)를 LLM에 전달합니다. LLM은 이 규격에 맞게 JSON 등으로 응답을 생성하며, LangChain이 이를 다시 `Objective` 객체로 자동 파싱하여 반환합니다.
  - **관련 패턴**: `instruction`, `output_format`, `examples`, `notes` 구성은 효과적인 프롬프트를 구성하는 **'Prompt Template Components'** 패턴이자, 결과의 일관성을 확보하는 **'Structured Prompting'** 기법입니다. 이는 특히 **CO-STAR**나 **CREATE**와 같은 유명한 프롬프트 프레임워크의 핵심 요소들을 선별한 것으로, 작업(Instruction), 예시(Few-shot), 형식(Format), 제약 사항(Notes)을 명확히 구분함으로써 LLM의 이해도와 결과 품질을 극대화합니다.

- **`|` (Pipe Operator)**:
  - **LCEL(LangChain Expression Language)**의 핵심 연산자로, 앞 단계의 결과를 뒷 단계의 입력으로 전달하는 **체이닝(Chaining)**을 의미합니다.
  - 파이썬의 `__or__` 매직 메서드를 오버라이딩하여 구현되었으며, Unix 파이프와 같이 데이터가 왼쪽에서 오른쪽으로 흐르게 하여 복잡한 로직을 직관적으로 표현할 수 있게 돕습니다.

<br/>
<br/>


## e.g. (2)
```python
### 예전 코드 --- (start)
import keyring
import os

from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_google_genai import ChatGoogleGenerativeAI

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
    model="gemini-3-flash-preview",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)

# 프롬프트 자동 생성을 위한 요소 저장
from pydantic import BaseModel, Field
class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])


# from langchain_core.prompts import ChatPromptTemplate
# prompt = ChatPromptTemplate([
#     ('system', '아래의 작업을 보다 자세하게 요청하는 시스템 프롬프트를 구성하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
#     ('user', '{instruction}')

# ])
### 예전 코드 --- (end)

from typing import TypedDict
class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str


from langchain_core.prompts import ChatPromptTemplate

"""
(1) 'instruction' 속성이 포함된 State 객체를 state 를 받아서 (참고: graph.stream)
(2) chain : llm 을 거친 후 llm 이 Object 타입으로 structured_output(구조화된 출력)으로 변환하는 파이프라이닝 정의
(3) result : llm 을 통해 프롬프트를 생성하며 Objective 타입으로 구조화된 프롬프트 객체 'result'를 생성
    llm.with_structured_output(Objective) 은 llm 이 Objective 타입으로 결과물을 변환해서 반환합니다.
(4) ret_result : 결과값으로는 조금 더 풍부해진 프롬프트인 {'prompt_materials': result} 를 return 하며, prompt_materials 는 Objective 타입입니다.
"""
def get_prompt_materials(State):
    prompt = ChatPromptTemplate([
        ('system', '아래의 작업을 보다 자세하게 세분화하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
        ('user', '{instruction}')

    ])

    chain = prompt | llm.with_structured_output(Objective)

    result = chain.invoke({'instruction':State['instruction']})
    print(f"(get_prompt_materials) 'instruction' = {State['instruction']}, 생성된 프롬프트 생성 질문(생성재료 'prompt_materials') {result}")
    
    ret_result = {'prompt_materials' : result}
    # print(f"💡 ret_result = {ret_result}")
    return ret_result


"""
(1) {'prompt_materials': Objective} 속성이 포함된 State 객체를 state 를 받아서 ChatPromptTemplate 객체 prompt를 만든 후
(2) chain : llm을 거친 후 llm 이 StrOutputParser()를 통해 str 타입으로 변환하는 파이프라이닝 정의
(3) result : 실행 결과는 str 타입 (StrOutputParser 를 거쳤으므로)
(4) ret_result : {'full_prompt': result} 로 감싸서 'full_prompt' 속성에 대해 바인딩
"""
from langchain_core.output_parsers import StrOutputParser
def generate_prompt(State):
    prompt = ChatPromptTemplate([
        ('system', '''당신은 체계적이고 정확한 프롬프트 엔지니어입니다. 아래의 포인트를 바탕으로, LLM에 입력할  시스템 프롬프트를 작성하세요.
{points}'''),
        ('user', '{instruction}')
    ])
    

    chain = prompt | llm | StrOutputParser()

    ## 참고) 
    #### result : result 는 StrOutputParser() 를 통해 str 타입으로 변환된 결과물
    #### StrOutputParser : AI가 준 복잡한 객체에서 다른 군더더기(토큰 정보 등)는 다 버리고, 실제 답변인 '문자열(String)'만 추출합니다.
    result = chain.invoke({'instruction': State['instruction'], 'points': State['prompt_materials'].as_str})
    print(f"(generate_prompt) State = {State}, 생성된 프롬프트 결과물 {result}")
    ret_result = {'full_prompt' : result}
    # print(f"💡 ret_result = {ret_result}")
    return ret_result


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
<br/>
<br/>

### 코드 설명

#### (1)
```python
# 프롬프트 자동 생성을 위한 요소 저장
from pydantic import BaseModel, Field

class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])

## ...

from typing import TypedDict
class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str


from langchain_core.prompts import ChatPromptTemplate

"""
(1) 'instruction' 속성이 포함된 State 객체를 state 를 받아서 (참고: graph.stream)
(2) chain : llm 을 거친 후 llm 이 Object 타입으로 structured_output(구조화된 출력)으로 변환하는 파이프라이닝 정의
(3) result : llm 을 통해 프롬프트를 생성하며 Objective 타입으로 구조화된 프롬프트 객체 'result'를 생성
    llm.with_structured_output(Objective) 은 llm 이 Objective 타입으로 결과물을 변환해서 반환합니다.
(4) ret_result : 결과값으로는 조금 더 풍부해진 프롬프트인 {'prompt_materials': result} 를 return 하며, prompt_materials 는 Objective 타입입니다.
"""
def get_prompt_materials(State):
    prompt = ChatPromptTemplate([
        ('system', '아래의 작업을 보다 자세하게 세분화하고자 합니다. 주어진 포맷에 적절하게 작성하세요.'),
        ('user', '{instruction}')

    ])

    chain = prompt | llm.with_structured_output(Objective)

    result = chain.invoke({'instruction':State['instruction']})
    print(f"(get_prompt_materials) 'instruction' = {State['instruction']}, 생성된 프롬프트 생성 질문(생성재료 'prompt_materials') {result}")
    
    ret_result = {'prompt_materials' : result}
    # print(f"💡 ret_result = {ret_result}")
    return ret_result
```
<br/>

위 코드에 대해 바로 아래의 ##### A 에 설명하세요. 질문 내용은 삭제하거나 수정하지 마세요.

##### A
이 코드는 자동화된 프롬프트 엔지니어링 워크플로우의 첫 번째 단계(Node)와 데이터 구조를 정의합니다.

- **`Objective` 클래스 (Pydantic 모델)**: LLM이 응답해야 할 **구조화된 데이터의 규격**을 정의합니다. `instruction`, `output_format`, `examples`, `notes`라는 4가지 핵심 필드를 가지며, `@property as_str`을 통해 이 필드들을 마크다운 문자열로 간결하게 변환하는 기능을 제공합니다.
- **`State` 클래스 (TypedDict)**: LangGraph 내에서 노드 간에 공유되는 **상태(Shared State)의 구조**를 정의합니다. 사용자의 입력부터 중간 생성물(`prompt_materials`), 그리고 최종 결과물까지 어떤 데이터가 저장될지 명시합니다.
- **`get_prompt_materials(State)` 함수 (Node)**:
    - **역할**: 사용자의 단순한 지시사항(`instruction`)을 바탕으로, 더 좋은 프롬프트를 만들기 위한 **'상세 재료'들을 생성**하는 첫 번째 작업 단위입니다.
    - **구조화된 출력**: `llm.with_structured_output(Objective)`을 사용하여 모델이 일반 텍스트가 아닌 `Objective` 객체의 형태로 결과물을 반환하도록 강제합니다.
    - **상태 업데이트**: 생성된 `Objective` 객체를 `prompt_materials`라는 키로 반환하며, 이는 LangGraph에 의해 전역 상태(`State`)에 자동으로 저장됩니다.

<br/>
<br/>

#### (2)
```python
# 프롬프트 자동 생성을 위한 요소 저장
from pydantic import BaseModel, Field

class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])

## ...

from typing import TypedDict
class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str


from langchain_core.prompts import ChatPromptTemplate

### ...
def get_prompt_materials(State):
    ### ... 
    return ret_result


"""
(1) {'prompt_materials': Objective} 속성이 포함된 State 객체를 state 를 받아서 ChatPromptTemplate 객체 prompt를 만든 후
(2) chain : llm을 거친 후 llm 이 StrOutputParser()를 통해 str 타입으로 변환하는 파이프라이닝 정의
(3) result : 실행 결과는 str 타입 (StrOutputParser 를 거쳤으므로)
(4) ret_result : {'full_prompt': result} 로 감싸서 'full_prompt' 속성에 대해 바인딩
"""
from langchain_core.output_parsers import StrOutputParser
def generate_prompt(State):
    prompt = ChatPromptTemplate([
        ('system', '''당신은 체계적이고 정확한 프롬프트 엔지니어입니다. 아래의 포인트를 바탕으로, LLM에 입력할  시스템 프롬프트를 작성하세요.
{points}'''),
        ('user', '{instruction}')
    ])
    

    chain = prompt | llm | StrOutputParser()

    ## 참고) 
    #### result : result 는 StrOutputParser() 를 통해 str 타입으로 변환된 결과물
    #### StrOutputParser : AI가 준 복잡한 객체에서 다른 군더더기(토큰 정보 등)는 다 버리고, 실제 답변인 '문자열(String)'만 추출합니다.
    result = chain.invoke({'instruction': State['instruction'], 'points': State['prompt_materials'].as_str})
    print(f"(generate_prompt) State = {State}, 생성된 프롬프트 결과물 {result}")
    ret_result = {'full_prompt' : result}
    # print(f"💡 ret_result = {ret_result}")
    return ret_result
```
위 코드에 대해 바로 아래의 ##### 설명 아래에 설명을 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

##### 설명
이 코드는 이전 단계에서 생성된 '상세 재료'들을 조합하여 **최종적인 시스템 프롬프트를 완성**하는 두 번째 노드(Node)입니다.

- **입력 데이터**: 
    - `State['instruction']`: 사용자가 처음에 입력한 기초 지시사항입니다.
    - `State['prompt_materials'].as_str`: 첫 번째 노드에서 생성된 `Objective` 객체의 프로퍼티를 호출하여, 구조화된 재료들을 마크다운 형식의 문자열로 변환해 가져옵니다.
- **파이프라이닝 및 출력 파서**:
    - `chain = prompt | llm | StrOutputParser()`: 이번 체인에서는 `with_structured_output` 대신 **`StrOutputParser()`**를 사용합니다. 이는 AI의 응답 메시지에서 군더더기를 빼고 실제 **텍스트 내용(문자열)**만 깔끔하게 추출하기 위함입니다.
- **상태 업데이트**:
    - 최종 생성된 프롬프트 문자열인 `result`를 `full_prompt`라는 키에 담아 반환합니다. 이를 통해 그래프의 전역 상태(`State`)에 완성된 시스템 프롬프트가 저장되며, 다음 노드에서 이를 바로 사용할 수 있게 됩니다.

<br/>
<br/>

#### (3)
```python
# 프롬프트 자동 생성을 위한 요소 저장
from pydantic import BaseModel, Field

class Objective(BaseModel):
    instruction: str = Field(description='프롬프트의 지시 사항을 명확히 재구성')
    output_format: str = Field(description='출력 포맷에 대한 설명')
    examples: str = Field(description='예시 출력(1개)')
    notes: str = Field(description='작업 과정에서 중요한 내용을 4개의 개조식 문장으로 구성')

    @property #
    def as_str(self) -> str:
        return '\n\n'.join([f'## {key}\n {value}' for key, value in self])

## ...

from typing import TypedDict
class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str


from langchain_core.prompts import ChatPromptTemplate

### ...
def get_prompt_materials(State):
    ### ... 
    return ret_result

from langchain_core.output_parsers import StrOutputParser
def generate_prompt(State):
    ## ...



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
<br/>

위 코드에 대한 설명을 바로 아래의 '##### 설명'섹션에 작성하세요. 질문 내용은 수정하거나 삭제하지 마세요.<br/>
<br/>

##### 설명
이 코드는 최종적인 답변을 생성하기 위한 **마지막 노드 정의와 전체 그래프의 구성 및 실행** 과정을 보여줍니다.

- **`generate(State)` 노드**: 워크플로우의 최종 단계로, 이전 노드(`generate_prompt`)에서 완성된 `full_prompt`를 사용하여 LLM을 호출합니다. 모델로부터 받은 최종 결과물은 `result`라는 키에 담겨 `State`를 업데이트합니다.
- **그래프 구성 (`StateGraph`)**:
    - `StateGraph(State)`를 통해 우리가 정의한 상태 구조를 가진 그래프를 생성합니다.
    - `add_node`를 사용하여 `get_prompt_materials`, `generate_prompt`, `generate`라는 세 가지 작업 단위를 노드로 등록합니다.
    - `add_edge`를 통해 데이터가 흐르는 순서(`START` -> 재료 생성 -> 프롬프트 완성 -> 최종 생성 -> `END`)를 정의합니다.
- **그래프 컴파일 및 실행**:
    - `compile()`을 통해 실행 가능한 그래프 객체를 생성합니다.
    - `graph.stream(...)`은 초기 `instruction`을 입력받아 그래프를 실행하며, `stream_mode='values'` 옵션을 사용하여 각 단계가 끝날 때마다 전체 상태 정보를 출력(Yield)합니다. 이를 통해 사용자는 초기 아이디어가 어떻게 구체화되어 최종 리포트로 탄생하는지 단계별로 확인할 수 있습니다.


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
    # requests_per_second=0.167,  # 분당 10개 요청
    requests_per_second=1,  # 초당 최대 1개, 분당 최대 60개 요청
    check_every_n_seconds=0.1,  # 100ms마다 체크
    max_bucket_size=10,  # 최대 버스트 크기
)

# rate limiter를 LLM에 적용
llm = ChatGoogleGenerativeAI(
    model="gemini-3-flash-preview",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)
### 예전 코드 --- (end)

### 이번 코드 --- (start)
def talk(State):
    return {'context': AIMessage(content='AI 메시지 2')}

from typing import Annotated
from typing import TypedDict
from IPython.display import Image, display
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages

class State(TypedDict):
    context: Annotated[list, add_messages]

builder = StateGraph(State)
builder.add_node('talk',talk)
builder.add_edge(START, 'talk')
builder.add_edge('talk', END)

graph = builder.compile()

import pprint
## 출력문이 여러 곳에서 동시에 쓰이기에 함수로 공통화
def pprint_talk_graph(graph, input_messages):
    """
    그래프의 'context' 상태에 메시지 리스트를 전달하고
    각 단계별(Node) 실행 결과를 출력하는 헬퍼 함수입니다.
    """
    import pprint
    # 'instruction' 대신 현재 State 구조에 맞는 'context' 키를 사용합니다.
    for data in graph.stream({'context': input_messages}, stream_mode='values'):
        pprint.pprint(data)
        print('----' * 10)


from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
messages = [
    SystemMessage(content='시스템 메시지 1'),
    HumanMessage(content='유저 메시지 1'),
    AIMessage(content='AI 메시지 1'),
    HumanMessage(content='유저 메시지 2'),
]

### invoke : 한꺼번에 결과를 낸다.
### stream : 각 노드의 실행이 끝날 때마다 데이터를 순차적으로(Generator) 배출
# response = graph.invoke({'context': messages})
pprint_talk_graph(graph, messages)


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

### invoke : 한꺼번에 결과를 낸다.
### stream : 각 노드의 실행이 끝날 때마다 데이터를 순차적으로(Generator) 배출
graph.invoke({'context': messages})
# 유저 메시지 1, AI 메시지 1 삭제
### 이번 코드 --- (end)
```

### 코드 설명
#### (1)
```python
### 예전 코드 --- (start)
## ... 
rate_limiter = InMemoryRateLimiter(
    # requests_per_second=0.167,  # 분당 10개 요청
    requests_per_second=1,  # 초당 최대 1개, 분당 최대 60개 요청
    check_every_n_seconds=0.1,  # 100ms마다 체크
    max_bucket_size=10,  # 최대 버스트 크기
)

# rate limiter를 LLM에 적용
llm = ChatGoogleGenerativeAI(
    model="gemini-3-flash-preview",
    rate_limiter=rate_limiter,
    # temperature
    # max_tokens

    thinking_budget = 500  # 추론(Reasoning) 토큰 길이 제한
)
### 예전 코드 --- (end)


### 이번 코드 --- (start)
def talk(State):
    return {'context': AIMessage(content='AI 메시지 2')}

import pprint
## 출력문이 여러 곳에서 동시에 쓰이기에 함수로 공통화
def pprint_talk_graph(graph, input_messages):
    """
    그래프의 'context' 상태에 메시지 리스트를 전달하고
    각 단계별(Node) 실행 결과를 출력하는 헬퍼 함수입니다.
    """
    import pprint
    # 'instruction' 대신 현재 State 구조에 맞는 'context' 키를 사용합니다.
    for data in graph.stream({'context': input_messages}, stream_mode='values'):
        pprint.pprint(data)
        print('----' * 10)


from typing import Annotated
from typing import TypedDict
from IPython.display import Image, display
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages

class State(TypedDict):
    context: Annotated[list, add_messages]

builder = StateGraph(State)
builder.add_node('talk',talk)
builder.add_edge(START, 'talk')
builder.add_edge('talk', END)

graph = builder.compile()

from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
messages = [
    SystemMessage(content='시스템 메시지 1'),
    HumanMessage(content='유저 메시지 1'),
    AIMessage(content='AI 메시지 1'),
    HumanMessage(content='유저 메시지 2'),
]

### invoke : 한꺼번에 결과를 낸다.
### stream : 각 노드의 실행이 끝날 때마다 데이터를 순차적으로(Generator) 배출
# response = graph.invoke({'context': messages})
pprint_talk_graph(graph, messages)


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

### invoke : 한꺼번에 결과를 낸다.
### stream : 각 노드의 실행이 끝날 때마다 데이터를 순차적으로(Generator) 배출
graph.invoke({'context': messages})
# 유저 메시지 1, AI 메시지 1 삭제
### 이번 코드 --- (end)
```

위 코드에서 `### 이번 코드 --- (start)` ~ `### 이번 코드 --- (end)` 부분에 대한 설명을 바로 아래의 ##### A 에 작성하세요. 예제에서 무엇을 설명하려고 하는지, 예제의 의도는 무엇인지 설명을 바로 아래의 ##### A 에 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

##### A
이 실습 예제는 LangGraph에서 **대화 기록(Message History)을 어떻게 지능적으로 관리하고 제어하는지**를 보여주는 데 그 의도가 있습니다. 주요 설명과 의도는 다음과 같습니다.

### 예제의 의도 및 핵심 설명
이 예제는 단순히 메시지를 주고받는 것을 넘어, 그래프의 '상태(State)'를 **추가(Append)하거나 삭제(Delete)** 하는 패턴을 익히기 위해 설계되었습니다.

1.  **`add_messages` 리듀서의 활용 (상태 업데이트의 자동화)**:
    - **의도**: 개발자가 직접 리스트를 더하거나 합치는 복잡한 로직을 짤 필요 없이, LangGraph가 제공하는 `add_messages` 함수가 새로운 메시지를 기존 대화 기록에 자동으로 병합해주는 편리함을 설명합니다.
    - **핵심**: `Annotated[list, add_messages]` 설정을 통해 리스트 덮어쓰기가 아닌 '누적 업데이트'가 가능해집니다.

2.  **`RemoveMessage`를 통한 선택적 삭제 (컨텍스트 관리)**:
    - **의도**: 대화가 길어질 때 불필요한 메시지를 삭제하여 컨텍스트 윈도우를 효율적으로 관리하는 방법을 보여줍니다.
    - **핵심**: `delete_message` 노드에서 `RemoveMessage(id=...)`를 반환하면, 리듀서가 해당 ID를 찾아 실제 상태에서 메시지를 제거합니다. 이는 특정 시점의 대화 내용을 비우거나 정리할 때 매우 유용한 패턴입니다.

3.  **실행 방식의 시각화 (`invoke` vs `stream`)**:
    - **의도**: 그래프가 한 번에 결과를 내놓는 방식(`invoke`)과 각 단계(Node)를 거칠 때마다 상태가 어떻게 변하는지 실시간으로 보여주는 방식(`stream`)의 차이를 이해시킵니다.
    - **핵심**: `pprint_talk_graph` 함수를 통해 노드를 거칠 때마다 메시지가 추가되거나 삭제되는 과정을 눈으로 직접 확인하며 디버깅하는 능력을 길러줍니다.

**요약하자면:** 
이 예제는 **"메시지 리스트를 LangGraph의 내장 리듀서(`add_messages`)로 자동 결합하고, 때에 따라 특정 메시지를 ID 기반으로 안전하게 삭제(`RemoveMessage`)하는 프레임워크 수준의 상태 관리 기법"**을 습득하기 위한 목적을 가지고 있습니다.

<br/>


## Q. with_structured_output 과 StrOutputParser 의 차이점은?
### A
`with_structured_output`과 `StrOutputParser`는 모두 LLM의 결과물을 우리가 원하는 형태로 가공하는 도구들이지만, **그 작동 원리와 최종 결과물**에서 큰 차이가 있습니다. 

요약해 드리자면 다음과 같습니다:

| 비교 항목 | `with_structured_output` | `StrOutputParser` |
| :--- | :--- | :--- |
| **핵심 역할** | LLM에게 **"지정한 구조(JSON, Class)"로 답하라고 강제**함 | LLM이 보낸 **답변(AI Message)에서 '텍스트'만 쏙 뽑아냄** |
| **작동 방식** | LLM의 **도구 호출(Tool Calling/Function Calling)** 기능을 주로 활용 | LLM이 답변을 마친 후 **답변 내용(`content`)만 추출하는 사후 처리** |
| **최종 결과 타입** | **Pydantic 객체**(`Objective` 등) 또는 딕셔너리(`dict`) | 일반 **문자열(`str`)** |
| **용도** | **데이터 추출**, 객체 기반의 프로그래밍이 필요할 때 | **자유 형식의 답변**, 단순 텍스트 메시지가 필요할 때 |

---

### 좀 더 자세한 설명

#### 1. `with_structured_output(Objective)`
- **"LLM에게 가이드라인을 줌"**: 실행 전에 LLM에게 "내가 준 `Objective` 클래스의 모양에 맞춰서 JSON 형식으로 답을 해줘"라고 명확히 지시를 내립니다.
- **도구 호출 활용**: 최신 모델(Gemini, GPT-4 등)의 API에 포함된 '도구 호출' 기능을 사용하여, 모델이 애초부터 정해진 구조를 벗어나지 못하도록 유도합니다.
- **장점**: 답변이 항상 같은 구조로 보장되므로, 이후 코드에서 `result.notes`와 같이 객체 필드에 바로 접근할 수 있어 에러 발생 확률이 낮습니다.

#### 2. `StrOutputParser()`
- **"나중에 결과만 솎아냄"**: LLM은 평소처럼 자유롭게 대답합니다. 하지만 LLM의 응답에는 텍스트 외에 '사용한 토큰 수', '모델명' 같은 메타데이터가 포함된 `AIMessage` 객체가 전달됩니다.
- **사후 처리**: 이 복잡한 꾸러미에서 우리가 실제로 보려는 **알맹이인 '텍스트(content)'**만 문자열 타입으로 변환해주는 아주 단순한 청소기 역할을 합니다.
- **장점**: 별도의 구조를 설계할 필요 없이, 단순히 모델의 답변 메시지만 문자열로 깔끔하게 받아보고 싶을 때 가장 편리합니다.


## Q. `chain = prompt | llm | StrOutputParser()` 를 통해 prompt, llm, StrOutputParser 를 파이프라이닝한 것은 무슨 의미인가요?
### A
`prompt | llm | StrOutputParser()`와 같이 구성하는 것을 LangChain에서는 **LCEL(LangChain Expression Language)**이라고 부르며, 데이터가 왼쪽에서 오른쪽으로 흘러가며 변환되는 **파이프라인(Pipeline) 혹은 체인(Chain)**을 의미합니다.

각 단계별로 데이터가 어떻게 흘러가는지 그 의미를 풀어서 설명해 드릴게요:

#### 1. 데이터의 흐름 (Step-by-Step)
이 파이프라인은 우리가 `chain.invoke(입력값)`을 호출하는 순간 다음과 같이 순차적으로 작동합니다.
1. **`prompt` 단계**: 사용자가 준 딕셔너리(예: `{'instruction': '...'}`)를 받아서, 미리 정의된 템플릿에 끼워 넣어 **최종 프롬프트 문장(메시지 묶음)**을 만듭니다.
2. **`llm` 단계**: 앞단계에서 만든 프롬프트 문장을 **AI 모델(Gemini 등)에게 전달**합니다. AI는 이를 읽고 답변이 담긴 **`AIMessage`라는 복잡한 객체**를 돌려줍니다.
3. **`StrOutputParser()` 단계**: AI가 준 복잡한 객체에서 다른 군더더기(토큰 정보 등)는 다 버리고, **실제 답변인 '문자열(String)'만 추출**합니다.

#### 2. 파이프라이닝의 장점
단순히 코드를 연결하는 것 이상의 중요한 의미가 있습니다:
- **가독성**: `parser(llm(prompt(input)))`와 같이 복잡하게 중첩해서 쓰는 대신, `A | B | C` 형태로 써서 데이터가 흘러가는 방향을 한눈에 볼 수 있게 해줍니다.
- **유연성 (Runnable)**: 이 체인에 연결된 모든 요소는 'Runnable'이라는 표준 규격을 따릅니다. 덕분에 중간에 새로운 필터를 끼워 넣거나(`A | B | Filter | C`), 동기/비동기 처리를 아주 쉽게 바꿀 수 있습니다.
- **표준화된 인터페이스**: 이렇게 체인을 만들면 `invoke`(실행), `stream`(한 글자씩 출력), `batch`(한꺼번에 여러 개 처리) 등의 기능을 공통적으로 사용할 수 있게 됩니다.

**요약하자면:** 
"원재료(사용자 입력)를 받아 -> 요리 도구(Prompt)로 다듬고 -> 셰프(LLM)가 요리한 뒤 -> 접시(Parser)에 예쁘게 담아 내놓는" **일련의 과정을 하나의 자동화된 공정(파이프라인)으로 묶었다**는 뜻입니다!



## Q. `for data in graph.stream(...)` 에 대해
```python
import pprint
from langgraph.graph import StateGraph, START, END

# Streaming 참고
# https://langchain-ai.github.io/langgraph/concepts/streaming/#streaming-graph-outputs-stream-and-astream
def pprint_graph(graph):
    for data in graph.stream({'instruction': '''영화 '마이너리티 리포트'와 AI 윤리의 연관성에 대한 리포트 쓰기'''}, stream_mode='values'):
        pprint.pprint(data)
```
위 코드에 대해 바로 아래의 ### A 에 설명하세요. 질문 내용을 수정하거나 삭제하지 마세요.

### A
이 코드는 LangGraph의 **스트리밍(Streaming)** 기능을 활용하여 그래프가 실행되는 과정을 실시간으로 모니터링하는 방법을 보여줍니다.

- **`graph.stream(...)`**: 그래프를 실행할 때 `invoke`와 달리 한꺼번에 결과를 내는 것이 아니라, **각 노드의 실행이 끝날 때마다 데이터를 순차적으로(Generator) 배출**합니다.
- **`stream_mode='values'` (핵심)**:
    - 이 옵션은 노드가 완료될 때마다 **그래프의 전역 상태(State) 전체**를 출력하도록 설정합니다.
    - 기본값인 `updates` 모드는 해당 노드에서 '변경된 부분'만 보여주는 반면, `values` 모드는 누적된 전체 데이터를 보여주므로 진행 상황을 한눈에 파악하기 좋습니다.
- **`for data in ...` 루프**: 스트림이 종료(END 노드 도달)될 때까지 생성되는 데이터를 하나씩 꺼내어 처리합니다.
- **`pprint.pprint(data)`**: 파이썬의 `pprint` 라이브러리를 사용해 복잡한 딕셔너리 구조의 상태 데이터를 읽기 좋은 형식으로 출력합니다.

**요약하자면:** 
전체 작업이 끝나기까지 기다리지 않고, **"A 노드가 끝나면 이 상태, B 노드가 끝나면 이 상태"** 식으로 그래프 내부의 **데이터 변화 과정을 실시간으로 추적 및 디버깅**하기 위해 사용하는 코드입니다.


## Q. `pydantic` 은 무엇인지, BaseModel, Field 는 무엇인지 설명하세요.
e.g.
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
<br/>

`pydantic` 은 무엇인지, BaseModel, Field 는 무엇인지 설명하세요. 바로 아래의 ### A 에 설명을 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

### A
- **`pydantic`**: 파이썬의 타입 힌트를 기반으로 데이터의 **유효성 검사(Validation)**와 설정 관리를 수행하는 라이브러리입니다. 런타임에 데이터 타입을 강제하고 오류를 방지하는 역할을 합니다.
- **`BaseModel`**: Pydantic 모델을 만들 때 상속받는 기본 클래스입니다. 이를 통해 데이터를 자동으로 객체로 변환하거나 유효성을 검사하며, JSON 등으로 쉽게 직렬화할 수 있습니다.
- **`Field`**: 각 필드에 대한 **메타데이터(설명, 기본값 등)**를 추가할 때 사용합니다. LangChain의 `with_structured_output`과 결합하면, `description`에 적힌 내용이 LLM에게 필드의 의미를 알려주는 가이드 역할을 합니다.


## Q. python 의 typing 패키지, TypedDict, Annotated 에 대해 설명하세요. 
e.g.

```python
from typing import TypedDict, Annotated

### (1)
class State(TypedDict):
    instruction : str
    prompt_materials : Objective # Objective Class를 하나의 값에 저장
    full_prompt : str
    result : str

## ...

### (2)
class State(TypedDict):
    context: Annotated[list, add_messages]
```

python 의 typing 패키지, TypedDict, Annotated 에 대해 설명하세요. 바로 아래의 ### A 에 설명을 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

### A
- **`typing` 패키지**: 파이썬 코드 상에 타입 정보를 명시할 수 있게 돕는 표준 라이브러리입니다. 코드 가독성을 높이고 정적 분석 도구를 통한 오류 예방을 가능하게 합니다.
- **`TypedDict`**: 딕셔너리의 '키'와 '값의 타입'을 고정한 특수 타입입니다. LangGraph에서는 그래프 노드 간에 공유되는 **상태(State)의 규격**을 정의하기 위해 필수적으로 사용합니다.
- **`Annotated`**: 기존 타입에 추가적인 정보(메타데이터)를 덧붙이는 도구입니다. LangGraph에서는 특정 상태 필드에 **리듀서(Reducer, 예: add_messages)** 기능을 연결하여, 새로운 데이터가 들어올 때 기존 상태와 어떻게 합칠지(덮어쓰기 vs 추가하기 등)를 정의할 때 사용합니다.



## Q. LangGraph 의 Streaming 은 무엇인지, chain.stream() 과 graph.stream() 의 차이점은 무엇인지 설명하세요. 
LangGraph 의 Streaming 은 무엇인지, chain.stream() 과 graph.stream() 의 차이점은 무엇인지 설명하세요. 
바로 아래의 ### A 에 설명을 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

### A
- **LangGraph Streaming**: 그래프의 전체 실행이 끝날 때까지 기다리지 않고, 실행 과정 중에 발생하는 **중간 결과물들을 실시간으로 받아보는 기능**입니다.
- **`chain.stream()` vs `graph.stream()`**:
    - **`chain.stream()` (LangChain)**: 일반적으로 LLM의 답변 내용을 **글자 단위(Token by Token)**로 실시간으로 쏟아냅니다.
    - **`graph.stream()` (LangGraph)**: 그래프의 실행 흐름을 기반으로 하며, 각 **노드(Node)의 작업이 완료될 때마다** 현재의 상태나 업데이트 내용을 순차적으로 반환합니다. 전체 워크플로우의 진행 단계를 모니터링하기에 적합합니다.





## Q. graph.stream() 과 graph.invoke() 의 차이점은 무엇인가요?

graph.stream() 과 graph.invoke() 의 차이점은 무엇인가요? graph.invoke() 는 무엇이고, graph.stream() 은 무엇인가요? 바로 아래의 ### A 에 설명을 작성하세요. 질문 내용은 삭제하거나 수정하지 마세요.

### A
- **`graph.invoke()`**: 그래프를 시작부터 끝까지 **동기적으로 한 번에 실행**하고, 모든 처리가 완료된 후의 **'최종 상태' 하나만**을 응답으로 돌려줍니다.
- **`graph.stream()`**: 그래프를 실행하면서 발생되는 데이터를 **제너레이터(Generator) 형태로 하나씩 응답**합니다. 호출 시점에 결과가 한꺼번에 오지 않고 실행 도중에 계속 들어오기 때문에, 중간 디버깅이나 긴 작업의 진행 상황을 사용자에게 보여줄 때 사용합니다.




