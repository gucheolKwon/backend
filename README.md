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


ddd