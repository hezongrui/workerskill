# Python 后端 TDD 指南（FastAPI 为主）

适用于 Python Web 后端开发，以 FastAPI 为例，覆盖异步、依赖注入、SSE 流式响应的测试方式。Flask/Django 可类比参考。

---

## 核心原则

1. **API 层测行为，不测实现**：断言状态码和响应体，不检查内部路由注册
2. **依赖注入是测试的抓手**：通过 `dependency_overrides` 替换数据库/AI 客户端为 fake
3. **异步是常态**：测试用 `pytest-asyncio`，HTTP 调用用 `AsyncClient`
4. **SSE 测事件流**：断言事件序列和数据格式，不模拟底层连接

---

## 项目结构参考

```
myproject/
├── src/
│   ├── main.py              # FastAPI app 入口
│   ├── routers/
│   │   └── chat.py          # API 路由
│   ├── services/
│   │   └── rag_service.py   # 业务逻辑
│   └── dependencies.py      # 可注入的依赖（如 get_db, get_llm）
├── tests/
│   ├── conftest.py          # 共享 fixtures
│   ├── unit/
│   │   └── test_rag_service.py
│   └── integration/
│       └── test_chat_api.py
└── pyproject.toml
```

---

## 1. FastAPI 集成测试（依赖注入替换）

### 红

```python
# tests/integration/test_chat_api.py
import pytest
from httpx import AsyncClient
from src.main import app
from src.dependencies import get_rag_service

class FakeRagService:
    async def query(self, question: str):
        return {"answer": f"fake answer to: {question}"}

@pytest.fixture
async def client():
    app.dependency_overrides[get_rag_service] = FakeRagService
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    app.dependency_overrides.clear()

@pytest.mark.asyncio
async def test_chat_returns_answer(client):
    response = await client.post("/chat", json={"question": "hello"})
    assert response.status_code == 200
    assert response.json()["answer"] == "fake answer to: hello"
```

运行：
```bash
pytest tests/integration/test_chat_api.py -v
# 预期：FAIL - 404 Not Found（路由还没写）
```

### 绿

```python
# src/routers/chat.py
from fastapi import APIRouter, Depends
from src.dependencies import get_rag_service
from src.services.rag_service import RagService

router = APIRouter()

@router.post("/chat")
async def chat(payload: dict, service: RagService = Depends(get_rag_service)):
    result = await service.query(payload["question"])
    return result
```

```python
# src/dependencies.py
from src.services.rag_service import RagService

def get_rag_service() -> RagService:
    return RagService()
```

---

## 2. 异步业务逻辑 TDD

### 红

```python
# tests/unit/test_rag_service.py
import pytest
from src.services.rag_service import RagService

class FakeRetriever:
    async def search(self, query: str):
        return [{"text": "doc1"}]

class FakeLLM:
    async def generate(self, context: str, question: str):
        return "generated answer"

@pytest.mark.asyncio
async def test_rag_service_combines_retriever_and_llm():
    service = RagService(retriever=FakeRetriever(), llm=FakeLLM())
    result = await service.query("what is ai?")
    assert result["answer"] == "generated answer"
    assert len(result["sources"]) == 1
```

### 绿

```python
# src/services/rag_service.py
class RagService:
    def __init__(self, retriever=None, llm=None):
        self.retriever = retriever
        self.llm = llm

    async def query(self, question: str):
        docs = await self.retriever.search(question)
        context = "\n".join([d["text"] for d in docs])
        answer = await self.llm.generate(context, question)
        return {"answer": answer, "sources": docs}
```

---

## 3. SSE 流式响应测试

### 红

```python
# tests/integration/test_stream.py
import pytest
from httpx import AsyncClient
from src.main import app

@pytest.mark.asyncio
async def test_stream_returns_events():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        async with ac.stream("POST", "/stream", json={"prompt": "hi"}) as response:
            assert response.status_code == 200
            events = []
            async for line in response.aiter_lines():
                if line.startswith("data: "):
                    events.append(line[6:])
            assert events[0] == '"hello"'
            assert events[-1] == '"[DONE]"'
```

### 绿

```python
# src/routers/stream.py
from fastapi import APIRouter
from fastapi.responses import StreamingResponse
from sse_starlette.sse import EventSourceResponse

router = APIRouter()

async def event_generator(prompt: str):
    for chunk in ["hello", "world", "[DONE]"]:
        yield chunk

@router.post("/stream")
async def stream(payload: dict):
    return EventSourceResponse(event_generator(payload["prompt"]))
```

**原则：**
- 测事件序列和数据内容
- 不 mock `EventSourceResponse` 内部，测端到端行为
- 如果生成器依赖外部 LLM，通过依赖注入传 fake generator

---

## 4. 依赖注入与数据库测试

### 用内存/测试数据库，避免 mock ORM

```python
# tests/conftest.py
import pytest
import pytest_asyncio
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from src.dependencies import get_db
from src.main import app

TEST_DATABASE_URL = "sqlite+aiosqlite:///:memory:"

@pytest_asyncio.fixture
async def db_session():
    engine = create_async_engine(TEST_DATABASE_URL)
    async_session_maker = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
    # 建表
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    async with async_session_maker() as session:
        yield session
    await engine.dispose()

@pytest_asyncio.fixture
async def client_with_db(db_session):
    async def override_get_db():
        yield db_session
    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    app.dependency_overrides.clear()
```

**原则：**
- 单元测试里 mock 外部慢依赖（LLM API、邮件服务）
- 集成测试里优先用内存数据库，不 mock ORM
- 所有可替换依赖都通过 `Depends()` 暴露出来

---

## 常用命令

```bash
# 运行全部测试
pytest

# 只跑集成测试
pytest tests/integration -v

# 只跑单元测试
pytest tests/unit -v

# 异步测试失败即停
pytest -x

# 带覆盖率
pytest --cov=src --cov-report=term-missing
```

---

## 关键 pytest 插件

| 插件 | 用途 |
|------|------|
| `pytest-asyncio` | 支持 `async def test_xxx()` |
| `httpx` | `AsyncClient` 测 FastAPI |
| `pytest-cov` | 覆盖率 |
| `aiosqlite` | 异步 SQLite 内存数据库 |

---

## 测试原则速查

1. **API 测试用 `AsyncClient`**，不要用 `requests`
2. **依赖用 `dependency_overrides` 替换**，不要改全局状态
3. **SSE 测事件序列**，不钻底层协议细节
4. **数据库优先用内存版**，比 mock ORM 更真实
5. **每个测试结束后清理 `dependency_overrides`**，避免状态泄漏
