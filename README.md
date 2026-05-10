# One-Agent (AI Agent 통합 앱)

AI Agent 가 4개의 Tool(시간, 일반 챗봇, 인터넷 검색, RAG)을 자동으로 선택해 실행하는 Streamlit 데모 앱입니다.

- 메인 파일: `one-agent.py`
- 기본 모델: `gpt-4o`
- RAG 백엔드: FAISS + BM25 (Ensemble)
- 인터넷 검색: Perplexity (선택, 키 미설정 시 비활성화)

---

## 1. 폴더 구성

```
prompt/5.one-agent/
├── one-agent.py              # Streamlit 앱 본체
├── requirements.txt          # 배포용 의존성
├── runtime.txt               # 배포 Python 버전 고정
├── .streamlit/
│   └── config.toml           # 서버/테마 설정
└── README.md                 # 본 문서
```

---

## 2. 로컬 실행

### 2-1. 가상환경 + 설치

```bash
python -m venv .venv
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
# macOS / Linux
source .venv/bin/activate

pip install -r prompt/5.one-agent/requirements.txt
```

### 2-2. 환경 변수

루트 또는 같은 폴더에 `.env` 를 만들고 키를 넣습니다.

```env
OPENAI_API_KEY=sk-...
# 선택 (Perplexity 도구를 쓸 때만 필요)
PPLX_API_KEY=pplx-...
```

### 2-3. 실행

```bash
streamlit run prompt/5.one-agent/one-agent.py
```

브라우저에서 `http://localhost:8501` 로 접속.

---

## 3. Streamlit Community Cloud 배포

### 3-1. 사전 준비

1. GitHub 저장소에 본 폴더가 푸시돼 있어야 합니다.
2. 같은 폴더(또는 레포 루트)에 아래 파일이 있어야 합니다.
   - `requirements.txt`
   - `runtime.txt`
   - `.streamlit/config.toml` (선택)

### 3-2. 새 앱 만들기

1. <https://share.streamlit.io> 접속 → **New app**.
2. 항목 입력
   - **Repository**: `your-account/your-repo`
   - **Branch**: `main`
   - **Main file path**: `prompt/5.one-agent/one-agent.py`
   - **Python version**: `runtime.txt` 기준으로 자동 인식 (현재 `python-3.12`).
3. **Advanced settings → Secrets** 에 아래를 그대로 붙여넣기.

```toml
OPENAI_API_KEY = "sk-..."
# 선택
PPLX_API_KEY = "pplx-..."
```

> 보안: `.env` 파일은 절대 커밋하지 마세요. `.gitignore` 에 `.env` 가 있는지 꼭 확인.

### 3-3. 배포 후 확인 체크리스트

- 첫 빌드는 5–10분 정도 걸릴 수 있습니다 (faiss-cpu 휠 다운로드).
- "Manage app" → 로그를 보고 `ModuleNotFoundError`/`ImportError` 가 없는지 확인.
- Perplexity 키 없이 띄우면 "인터넷 검색 도구는 비활성화됨" 메시지가 나오는 게 정상입니다.

### 3-4. 자주 만나는 문제

| 증상 | 원인 / 해결 |
| --- | --- |
| `ModuleNotFoundError: langchain.retrievers` | LangChain 1.x 환경. `langchain-classic` 이 `requirements.txt` 에 있는지 확인. |
| `OPENAI_API_KEY 가 .env 파일에서 찾을 수 없습니다` | Streamlit Cloud Secrets 에 키 등록 누락. Settings → Secrets 재확인. |
| `faiss-cpu` 빌드 실패 | `runtime.txt` 가 `python-3.12` 또는 `python-3.11` 인지 확인. 3.13/3.14 는 휠이 없을 수 있음. |
| 첫 응답이 느림 | 콜드 스타트(임베딩 모델 캐시) — 두 번째 호출부터 빨라집니다. |

---

## 4. 다른 호스팅 옵션 (요약)

- **Render / Railway / Fly.io**: `Procfile` 에  
  `web: streamlit run prompt/5.one-agent/one-agent.py --server.port=$PORT --server.address=0.0.0.0`
- **Docker** 기반 배포 시 베이스 이미지 `python:3.12-slim` 권장. 시스템 패키지로 `build-essential`, `libgomp1` 정도가 필요할 수 있습니다.

---

## 5. 빠른 헬스체크 (로컬)

```powershell
.\.venv\Scripts\python.exe -c "
import importlib
mods = [
  'streamlit','dotenv','openai',
  'langchain','langchain_core','langchain_community','langchain_openai','langchain_text_splitters',
  'langchain_classic.retrievers','langchain_classic.agents',
  'faiss','rank_bm25','pypdf'
]
for m in mods:
    try: importlib.import_module(m); print('OK ', m)
    except Exception as e: print('ERR', m, e)
"
```

모두 `OK` 가 나오면 배포 직전 상태가 정상입니다.
