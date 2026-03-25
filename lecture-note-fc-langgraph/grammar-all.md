# QnA 노트

## Q1. 함수 인자에 사용된 State 의미
아래 코드에서 `def get_prompt_materials(State):`와 같이 `State`를 인자로 받는데, 파이썬 문법에서 이것은 어떤 의미인가요?

```python
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
```

### A
파이썬 문법 관점에서 이 코드의 `def get_prompt_materials(State):`는 다음과 같은 의미를 가집니다.

1.  **매개변수 이름 (Parameter Name)**:
    - 여기서 `State`는 함수 내부에서 사용할 **매개변수(인자)의 이름** 그 자체입니다.
    - 파이썬의 타입 힌트 문법인 `state: State` 형식이 아니기 때문에, 문법적으로는 위에서 정의한 `class State`와는 상관없이 단순히 전달받은 객체를 `State`라는 변수명으로 부르겠다는 뜻입니다.

2.  **섀도잉 (Shadowing)**:
    - 위에서 `class State(TypedDict)`를 정의했지만, 함수 인자명을 똑같이 `State`로 지었기 때문에 함수 내부에서 `State`라는 단어는 클래스가 아닌 **전달받은 인자값**을 가리키게 됩니다. (이를 변수 섀도잉이라고 합니다.)
    - 관습적으로 파이썬에서는 변수나 인자명은 소문자(`state`)를 사용하고, 타입 힌트를 줄 때 `state: State`와 같이 쓰지만, 이 코드에서는 인자명 자체를 대문자 `State`로 사용한 사례입니다.

3.  **LangGraph에서의 역할**:
    - LangGraph의 노드 함수는 항상 그래프의 **현재 상태(State)**를 첫 번째 인자로 전달받습니다.
    - 따라서 이 함수는 실행될 때 현재의 상태 딕셔너리를 전달받아 내부의 `State['instruction']`과 같이 데이터를 조회하는 용도로 사용하고 있는 것입니다.
