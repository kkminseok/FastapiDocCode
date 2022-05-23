# FastapiDocCode
fastapi도큐에 있는 예제 코드 다 긁어모음. 내가 보기 편하게

# 20. 📜 File

<https://kkminseok.github.io/posts/fastapi20/>

2가지 방식

```python

@app.post("/files/")
async def create_file(file: bytes = File()):
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}

# 디폴트 값 설정
@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile | None = None):
    if not file:
        ...
    else:
        ...

# 메타정보 추가

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(default=None,description= "upload file read by kms")):
    ...

# list로 여러개 파일 한 번에 받기 + 메타정보입력

@app.post("/files/")
async def create_files(
    files: list[bytes] = File(description="Multiple files as bytes"),
):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(
    files: list[UploadFile] = File(description="Multiple files as UploadFile"),
):
    return {"filenames": [file.filename for file in files]}

#UploadFile list로 받기
@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile]):
    ...

```

# 21. 📜 Request Forms and Files

<https://kkminseok.github.io/posts/fastapi21/>


```python
# File, Form 같이 받기
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(
    file: bytes = File(), fileb: UploadFile = File(), token: str = Form()
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

# 22. 📜 Handling Errors

<https://kkminseok.github.io/posts/fastapi22/>

```python
from fastapi import FastAPI, HTTPException

#기본 예제
@app.get("/items/{item_id}")
async def read_item(item_id : str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found by kms")
    return ...

#헤더에 에러 추가
async def read_item(item_id : str):
    if item_id not in items:
        raise HTTPException(headers={"Kms-error": "There goes KMS error"},)
```

에러 핸들러 제작

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class CustomException(Exception):
    def __init__(self,name: str):
        self.name = name

app = FastAPI()

items = {"kms" : "Kms is Handsome guy"}

@app.exception_handler(CustomException)
async def unvicorn_exception_handelr(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=418,
        content={"message" : f"Oops! {exc.name} did something.. There goes a rainbow"},
    )
@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise CustomException(name=name)
    return {"unicorn_name": name}

```

기본 예외처리 커스텀

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse

app = FastAPI()


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}

```

`HTTPException` 오버라이딩

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

`RequestValidationError`

```python
from fastapi import FastAPI, Request, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({"detail": exc.errors(), "body": exc.body}),
    )


class Item(BaseModel):
    title: str
    size: int


@app.post("/items/")
async def create_item(item: Item):
    return item

```

기본 예외처리기와 커스텀 예외처리 같이 쓰기

```python
from fastapi import FastAPI, HTTPException
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    return await http_exception_handler(request, exc)


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    print(f"OMG! The client sent invalid data!: {exc}")
    return await request_validation_exception_handler(request, exc)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```