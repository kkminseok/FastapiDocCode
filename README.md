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



