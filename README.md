# LangGraph 기반 운동 추천 멀티에이전트 실습

## 1. 프로젝트 소개

본 프로젝트는 LangGraph를 활용하여 사용자의 질문을 단계적으로 처리하는 멀티에이전트 워크플로우를 구현한 실습 프로젝트입니다.

사용자의 질문에서 운동 추천에 필요한 증상 또는 고민 상태를 추출하고, 해당 상태를 해결하는 데 도움이 될 수 있는 운동 후보를 생성한 뒤, 최종적으로 사용자가 이해하기 쉬운 개조식 답변을 생성합니다.

예시 질문은 다음과 같습니다.

```text
체력이 안 좋고, 살이 계속 찌는데 어떤 운동을 할까?
```

본 프로젝트는 단순히 LLM에 한 번 질문하고 답변을 받는 구조가 아니라, 여러 개의 역할 기반 에이전트를 LangGraph의 노드로 구성하여 순차적으로 실행하는 구조입니다.

---

## 2. 프로젝트 목적

본 프로젝트의 목적은 다음과 같습니다.

- LangGraph의 기본 구조 이해
- StateGraph를 활용한 상태 기반 워크플로우 구현
- 역할이 분리된 멀티에이전트 구조 실습
- LangChain의 PromptTemplate과 LLM Chain 활용
- Ollama 기반 로컬 LLM 실행
- 그래프 구조 시각화 및 실행 결과 확인

---

## 3. 멀티에이전트 구조 소개

본 프로젝트는 총 3개의 에이전트로 구성되어 있습니다.

| 에이전트 | 역할 | 입력값 | 출력값 |
|---|---|---|---|
| 추출 에이전트 | 사용자 질문에서 증상 또는 고민 상태 추출 | 사용자 질문 | 체력 저하, 체중 증가 등 |
| 후보 에이전트 | 추출된 증상을 바탕으로 운동 후보 생성 | 추출된 증상 | 걷기, 실내 자전거, 근력 운동 등 |
| 답변 생성 에이전트 | 증상과 운동 후보를 바탕으로 최종 답변 생성 | 증상, 운동 후보 | 개조식 운동 추천 답변 |

각 에이전트는 독립적인 프롬프트를 사용하며, LangGraph의 StateGraph를 통해 순차적으로 연결됩니다.

---

## 4. 전체 처리 흐름

```text
사용자 질문
   ↓
[Extractor Agent]
사용자의 증상 또는 고민 상태 추출
   ↓
[Candidate Agent]
증상 해결에 적합한 운동 후보 생성
   ↓
[Answer Agent]
증상과 운동 후보를 바탕으로 최종 답변 생성
   ↓
최종 응답 출력
```

본 프로젝트의 핵심은 `AgentState`라는 상태값이 각 노드를 지나면서 점진적으로 채워진다는 점입니다.

---

## 5. 기술 스택

| 기술 | 사용 목적 |
|---|---|
| Python | 전체 프로젝트 구현 언어 |
| LangGraph | 에이전트 실행 흐름을 그래프 구조로 구성 |
| LangChain | PromptTemplate 및 LLM Chain 구성 |
| Ollama | 로컬 환경에서 LLM 실행 |
| EXAONE 3.5 | Ollama에서 사용하는 언어 모델 |
| IPython | 그래프 구조 이미지 출력 |

---

## 6. 사용 모델

본 프로젝트에서는 Ollama를 통해 로컬 LLM 모델을 실행합니다.

```python
llm = Ollama(model="exaone3.5:7.8b")
```

Ollama를 사용하면 외부 API Key 없이 로컬 환경에서 LLM 기반 에이전트 실습을 진행할 수 있습니다.

---

## 7. 상태 정의

LangGraph에서는 각 노드가 공유하는 상태값을 정의해야 합니다.

본 프로젝트에서는 `TypedDict`를 사용하여 상태 구조를 정의했습니다.

```python
class AgentState(TypedDict, total=False):
    query: str
    symptoms: str
    exercise_candidates: str
    result: str
```

각 상태값의 의미는 다음과 같습니다.

| 상태값 | 설명 |
|---|---|
| query | 사용자가 입력한 원본 질문 |
| symptoms | 추출 에이전트가 추출한 증상 또는 고민 |
| exercise_candidates | 후보 에이전트가 생성한 운동 후보 리스트 |
| result | 답변 생성 에이전트가 생성한 최종 응답 |

---

## 8. 그래프 구조

아래 이미지는 LangGraph로 구성한 에이전트 실행 구조입니다.

![LangGraph 구조](./assets/graph_structure.png)

그래프는 다음 순서로 실행됩니다.

```text
extractor → candidate → answer → END
```

각 노드는 하나의 에이전트 역할을 수행하며, 마지막 `answer` 노드에서 최종 응답을 생성한 뒤 종료됩니다.

---

## 9. 실행 결과 예시

입력 질문은 다음과 같습니다.

```text
체력이 안 좋고, 살이 계속 찌는데 어떤 운동을 할까?
```

실행 결과는 아래와 같습니다.

![실행 결과](./assets/result_capture.png)

예상 출력 예시는 다음과 같습니다.

```text
============================== 추출된 증상:
체력 저하, 체중 증가

============================== 운동 후보:
걷기, 실내 자전거, 가벼운 조깅, 근력 운동, 스트레칭

============================== 최종 응답:
- 체력 저하가 있는 경우 걷기처럼 부담이 적은 운동부터 시작하는 것이 좋습니다.
- 체중 증가가 고민이라면 유산소 운동과 근력 운동을 함께 병행하는 것이 도움이 될 수 있습니다.
- 처음부터 고강도 운동을 하기보다는 낮은 강도로 시작하여 점진적으로 운동 시간을 늘리는 것이 좋습니다.
```

---

## 10. 실행 방법

### 1. Ollama 모델 다운로드

먼저 Ollama에서 사용할 모델을 다운로드합니다.

```bash
ollama pull exaone3.5:7.8b
```

### 2. 필요한 라이브러리 설치

```bash
pip install -r requirements.txt
```

### 3. Python 파일 실행

```bash
python main.py
```

또는 Jupyter Notebook 환경에서 셀 단위로 실행할 수 있습니다.

---

## 11. 주요 코드 설명

### 11.1 추출 에이전트

```python
def extractor_agent(state: AgentState):
    chain = extractor_prompt | llm

    symptoms = chain.invoke({
        "query": state["query"]
    })

    return {
        **state,
        "symptoms": symptoms.strip()
    }
```

추출 에이전트는 사용자의 질문에서 운동 추천에 필요한 핵심 증상 또는 고민 상태를 추출합니다.

예를 들어 다음과 같은 질문이 입력되면,

```text
체력이 안 좋고, 살이 계속 찌는데 어떤 운동을 할까?
```

아래와 같은 핵심 상태를 추출합니다.

```text
체력 저하, 체중 증가
```

---

### 11.2 후보 에이전트

```python
def candidate_agent(state: AgentState):
    chain = candidate_prompt | llm

    exercise_candidates = chain.invoke({
        "symptoms": state["symptoms"]
    })

    return {
        **state,
        "exercise_candidates": exercise_candidates.strip()
    }
```

후보 에이전트는 추출된 증상 또는 고민 상태를 바탕으로 도움이 될 수 있는 운동 후보를 생성합니다.

예상 결과는 다음과 같습니다.

```text
걷기, 실내 자전거, 가벼운 조깅, 근력 운동, 스트레칭
```

---

### 11.3 답변 생성 에이전트

```python
def answer_agent(state: AgentState):
    chain = answer_prompt | llm

    answer = chain.invoke({
        "symptoms": state["symptoms"],
        "exercise_candidates": state["exercise_candidates"]
    })

    return {
        **state,
        "result": answer.strip()
    }
```

답변 생성 에이전트는 추출된 증상과 추천 운동 리스트를 종합하여 사용자가 이해하기 쉬운 개조식 답변을 생성합니다.

---

## 12. LangGraph 구성 코드

```python
graph = StateGraph(AgentState)

graph.add_node("extractor", extractor_agent)
graph.add_node("candidate", candidate_agent)
graph.add_node("answer", answer_agent)

graph.set_entry_point("extractor")

graph.add_edge("extractor", "candidate")
graph.add_edge("candidate", "answer")
graph.add_edge("answer", END)

app = graph.compile()
```

위 코드는 LangGraph의 핵심 구조입니다.

각 에이전트를 노드로 추가하고, `add_edge()`를 통해 실행 순서를 정의합니다.

---

## 13. 프로젝트 특징

본 프로젝트의 특징은 다음과 같습니다.

- LangGraph를 활용한 노드 기반 워크플로우 구성
- 역할이 분리된 멀티에이전트 구조 구현
- `TypedDict`를 활용한 상태 관리
- LCEL 방식의 PromptTemplate과 LLM 연결
- Ollama 기반 로컬 LLM 실행
- 그래프 구조 시각화를 통한 실행 흐름 확인
- 단일 LLM 호출이 아닌 단계별 처리 구조 구현

---

## 14. 프로젝트 의의

본 프로젝트는 LangGraph를 활용하여 LLM 기반 멀티에이전트 시스템을 직접 구성한 실습입니다.

특히 사용자의 질문을 한 번에 처리하지 않고, 다음과 같이 단계별로 나누어 처리했습니다.

```text
질문 이해 → 핵심 정보 추출 → 후보 생성 → 최종 답변 생성
```

이를 통해 LangGraph의 상태 기반 실행 흐름, 노드 연결 구조, 에이전트 역할 분리 방식을 실습할 수 있었습니다.

---

## 15. 개선 가능 방향

향후 다음과 같은 방식으로 프로젝트를 확장할 수 있습니다.

- 사용자의 나이, 운동 경험, 질환 여부 등을 추가 입력값으로 반영
- 운동 강도를 초급, 중급, 고급으로 분류
- 조건문을 활용하여 유산소 중심 / 근력 중심 루트 분기
- LangGraph의 conditional edge를 활용한 분기형 에이전트 구조 구현
- 운동 추천 결과를 표 형태로 정리
- Streamlit 또는 Gradio를 활용한 웹 인터페이스 구현
