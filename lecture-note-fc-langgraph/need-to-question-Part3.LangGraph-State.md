## Pydantic

pydantic 은 무엇인지

Objective class 는 무엇인지 (BaseModel을 확장한..) 

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

## ChatPromptTemplate

ChatPromptTemplate 은 무엇인지 (저번에 한번 보긴 했지만 개념파악 용으로 )