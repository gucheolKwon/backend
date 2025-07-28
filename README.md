# Python Backend (FastAPI)
```bash
uvicorn main:app --reload
```

```python
from fastapi import FastAPI
app = FastAPI()

# FastAPI 인스턴스 생성 -> 데코레이터(@app.get 등)로 엔드포인트(라우트) 선언
@app.get("/")
async def root():
	return {"message": "Hello World"}
```

```python
from pydantic import BaseModel
from typing import Optional

class Item(BaseModel):
	name: str
	description: Optional[str] = None

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: Optional[str] = None):
	return {"item_id": item_id, "q": q}

@app.post("/items/")
async def create_item(item: Item):
	return item
```

의존성 주입 (Dependency Injection) : DB 연결, 인증 등 반복되는 로직을 함수 단위로 의존성 주입해 관리
```python
from fastapi import Depends

def get_db():
    db = ... # db 객체 할당
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
async def read_users(db=Depends(get_db)):
    # db 객체 사용
    ...
```

미들웨어, 이벤트, 예외처리
- 미들웨어 : 요청과 응답 사이에 실행되는 코드로, 공통처리 (로깅, 보안, 인증, CORS 세팅 등) 요청을 가로채는 로직 주입. FastAPI가 Starlette 기반이므로 Starlette 방식의 미들웨어를 지원함
```python
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi.middleware.cors import CORSMiddleware
import time

app = FastAPI()
class LogMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.time()
        response = await call_next(request)
        duration = time.time() start
        print(f"Request: {request.url.path}, Time: {duration}")
        return response

app.add_middleware(LogMiddleware)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```
- 이벤트 핸들러 : 앱 시작/종료 시 실행할 함수 정의(@app.on_event("startup") / ("shutdown"))
- 예외처리 : HTTPException으로 사용자 지정 에러 응답

APIRouter로 대규모 서비스 구조화 : 여러 파일/모듈 분할, 대규모 서비스에 필수적인 구조
```python
from fastapi import APIRouter
router = APIRouter()

@router.get("/users/")
def list_users(): ...
app.include_router(router)
```

BackGroundTask : 긴 작업, 알림 발송, 로그 저장, 파일 처리 등 메인 응답 후 따로 처리해야할 업무를 BackgroundTask로 숳ㅐㅇ할 수 있고, 동기/비동기 구분없이 간단하게 사용 가능
```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()
def write_log(message: str):
    with open("log.txt", mode="a") as file:
        file.write(message + "\n")

@app.post("/send/")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"Send email to: {email}")
    return {"message": f"Email sent to {email}"}
# 웹 요청이 바로 끝나고, 해당 작업은 백그라운드(스레드)에서 처리됨, 비동기 함수도 등록 가능
```

Pydantic : 데이터 유효성 검증과 직렬화/파싱을 위해 사용하는 라이브러리
- 타입 힌트 기반 자동 검증
- .json(), .dict() 등 직렬화/파싱 매우 쉬움
- 중첩 모델, 커스텀 검증, 디폴트값, Enum, 데이터 변환 등 지원
```python
from pydantic import BaseModel, Field
from typing import List, Optional
from enum import Enum

class StatusEnum(str, Enum):
    active = "active"
    inactive = "inactive"

class User(BaseModel):
    id: int
    name: str
    age: Optional[int] = Field(None, gt=0)
    status: StatusEnum = StatusEnum.active

# 사용 예시
user = User(id=1, name="Alice")
print(user.dict())
```
















