공식 명칭과 구체적인 모델 ID는 **Google AI for Developers** (구 Google AI Studio)의 공식 문서 페이지에서 가장 정확하게 확인할 수 있습니다.

## 1. 공식 확인 처 (URL)
아래 페이지들은 실시간으로 업데이트되는 모델 리스트와 성능 지표를 제공합니다.

* **모델 개요 및 리스트:** [ai.google.dev/gemini-api/docs/models](https://ai.google.dev/gemini-api/docs/models)
    * 현재 사용 가능한 모든 Gemini 모델(Pro, Flash, Flash-Lite 등)의 공식 명칭과 특징이 나열되어 있습니다.
* **모델 버전 및 라이프사이클:** [ai.google.dev/gemini-api/docs/models/gemini](https://ai.google.dev/gemini-api/docs/models/gemini)
    * `gemini-2.5-flash-001`과 같은 구체적인 버전 번호와 업데이트 주기를 확인할 수 있습니다.

---

## 2. API 호출 시 사용하는 공식 ID 규칙
코드에서 직접 호출할 때 사용하는 ID는 대개 다음과 같은 형식을 따릅니다.

| 구분 | 형식 (Example) | 설명 |
| :--- | :--- | :--- |
| **최신 안정화 버전** | `gemini-3.1-pro` | 해당 시리즈의 가장 최신 Stable 모델로 자동 연결됩니다. |
| **특정 버전 고정** | `gemini-3.1-pro-001` | 모델 업데이트에 영향받지 않도록 특정 버전을 고정하여 호출합니다. |
| **프리뷰 버전** | `gemini-3.1-flash-preview` | 정식 출시 전 최신 기능을 먼저 테스트할 때 사용합니다. |

---

## 3. 코드에서 실시간으로 확인하기 (Python 예시)
공식 문서를 찾는 대신, 현재 내 API Key로 접근 가능한 모델 목록을 코드로 직접 출력해 볼 수도 있습니다.

```python
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

for m in genai.list_models():
    if 'generateContent' in m.supported_generation_methods:
        print(f"모델명: {m.display_name} / ID: {m.name}")
```

> **참고:** 2026년 현재 **Gemini 3.1** 시리즈가 최신이며, 이전 세대인 **Gemini 2.0 / 2.5** 모델 중 일부는 곧 지원 중단(Deprecation)될 예정이므로 위 문서의 'Retirement date'를 꼭 확인하시는 것이 좋습니다.

혹시 현재 개발 중인 환경(Python, Node.js 등)에 맞는 구체적인 연결 설정 코드가 필요하신가요?


---
## 참고
- 바이브 코딩을 하다보면 AI 모델들이 최신 모델명을 제대로 맞추지 못하기에 직접 찾아서 맞춰줘야 한다. 따라서 직접 어디에 해당 내용이 있는지 정도는 알아야 한다.
<br/>

## 스크린샷
https://ai.google.dev/gemini-api/docs/models 에 방문한다. 나는 Gemini 3 Flash 를 선택했다.
![alt text](<CleanShot 2026-03-28 at 16.34.01.png>)
<br/>

다음과 같이 모델 코드와 입출력 토큰한도, 기타 다른 기능들을 모두 확인할 수 있다.<br/>
![alt text](<CleanShot 2026-03-28 at 16.36.09.png>)

<br/>
