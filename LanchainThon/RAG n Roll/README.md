# 자소서낋여오거라 🎯
> **JD 기준으로 코칭하는 RAG 솔루션**  
> '그냥 써주는 AI'가 아닌, 채용공고(JD) 기반으로 자소서를 점수화하고 합격 패턴에 근거해 첨삭하는 서비스입니다.

<br>

## 📌 프로젝트 개요

많은 지원자들이 AI를 활용해 자소서를 작성하지만, 결과물은 **예쁜 문장**일 뿐 **합격의 근거**가 부족합니다.

| 기존 AI 자소서 서비스 | 자소서낋여오거라 |
|---|---|
| Fluency 중심 생성 | JD 적합도 기반 피드백 |
| 왜 고쳐졌는지 설명 없음 | 구체적 이유 + 점수 제시 |
| 환각(Hallucination) 위험 | 합격 사례 DB 기반 근거 제시 |
| 정량적 평가 기준 없음 | LangSmith 3대 지표 검증 |

<br>

## 🛠 기술 스택

| 분류 | 기술 |
|---|---|
| LLM | OpenAI GPT-4o |
| Orchestration | LangChain |
| Vector Store | Chroma DB |
| Data Validation | Pydantic |
| Evaluation | LangSmith |
| UI | Streamlit |
| Language | Python 3.10+ |

<br>

## 📁 프로젝트 구조

```
.
├── chroma_db/          # 벡터 저장소 (합격 자소서 임베딩)
├── data/               # 합격 자소서 PDF 원본 (46개)
├── rag/                # RAG 파이프라인 핵심 로직
├── ui/                 # Streamlit 프론트엔드
├── utils/              # 공통 유틸리티 함수
├── app.py              # 서비스 진입점 (Streamlit 실행)
├── build_index.py      # Chroma DB 인덱싱 스크립트 (최초 1회 실행)
├── config.py           # 환경 설정 (모델명, 경로 등)
├── eval_langsmith.py   # LangSmith 평가 실행 스크립트
├── prompts.py          # 프롬프트 템플릿 모음
├── run_ls_eval.py      # LangSmith 실험 실행 래퍼
├── schemas.py          # Pydantic 데이터 스키마 정의
├── requirements.txt    # 패키지 목록
└── .env                # API Key 환경변수 (git 제외)
```

<br>

## ⚙️ 코칭 파이프라인

```
[오프라인 인덱싱]
합격 자소서 PDF → PyPDFLoader 파싱 → 청킹 (800/150) → text-embedding-3 임베딩 → Chroma DB 저장

[실시간 서비스]
JD + 자소서 입력 → JD 구조화 (LLM/JSON) → Query 생성 → MMR 검색 → GPT-4o 코칭 → 리포트 출력
```

**5단계 코칭 프로세스:**
1. **JD 분석·구조화** - 필수 역량, 기술 스택, 암묵적 요구사항을 JSON으로 추출
2. **구조 라벨링** - 자소서 문단을 [도입/문제/해결/성과/피고]로 자동 분류
3. **정량 점수화** - JD 적합도 및 스토리텔링 품질을 객관적 지표로 산출
4. **수정안 + 근거** - 문장 단위 수정 제안과 함께 합격 사례/JD 근거 제시
5. **리포트 생성** - 클릭 시 수정본 자동 생성 및 상세 분석 리포트 다운로드

<br>

## 🚀 시작하기

### 1. 환경 설정

```bash
git clone https://github.com/your-repo/coverletter-rag.git
cd coverletter-rag
pip install -r requirements.txt
```

### 2. 환경변수 설정

`.env` 파일을 생성하고 아래 내용을 입력하세요:

```
OPENAI_API_KEY=your_openai_api_key
LANGCHAIN_API_KEY=your_langsmith_api_key
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=coverletter-rag
```

### 3. Chroma DB 인덱싱 (최초 1회)

```bash
python build_index.py
```

`data/` 폴더에 합격 자소서 PDF를 넣은 뒤 실행하세요.

### 4. 서비스 실행

```bash
streamlit run app.py
```

브라우저에서 `http://localhost:8501` 접속

<br>

## 📊 평가 결과 (LangSmith)

| 지표 | 결과 |
|---|---|
| JD Match | 0.70 |
| Overall Gap | 0.60 |
| Clarity | 0.50 (개선 필요) |
| 응답 지연 (P50) | 5.58s (기존 6.94s 대비 **-19.6%** 개선) |
| 건당 비용 | $0.01 미만 |

> **발견된 이슈**: Clarity 지표의 '50점 수렴 현상'  
> 원인: 평가자(Evaluator) 모델의 보수적 채점 로직으로 인한 변별력 부족  
> → 향후 평가자 모델 프롬프트 튜닝으로 개선 예정

<br>

## 🔮 향후 계획

**Phase 1 - 기술 내실화**
- Re-ranking 기법 도입 및 BM25 앙상블 검색으로 키워드 매칭 보완
- Fuzzy-Match로 원문 유지율 향상

**Phase 2 - 서비스 확장**
- 직무별 특화 합격 패턴 DB 구축 (마케팅, 영업 등)
- Multi-turn 대화형 피드백 지원

**Phase 3 - 생태계 구축**
- 링커리어 등 채용 플랫폼 API 제휴 (B2B2C)
- 대학 취업지원센터 관리자 대시보드 제공 (B2G)

<br>


## 📄 참고

- 합격 자소서 데이터 출처: 링커리어
- 평가 프레임워크: LangSmith (LLM-as-Judge)
