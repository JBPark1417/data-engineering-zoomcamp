# 파이프라인을 도커화하기 Dockerizing the Pipeline

**[↑ Up](README.md)** | **[← Previous](02-virtual-environment.md)** | **[Next →](04-postgres-docker.md)**

이제 스크립트를 컨테이너화해 보겠습니다. 다음 Dockerfile파일을 생성하세요.
Now let's containerize the script. Create the following `Dockerfile` file:
<img width="1096" height="554" alt="image" src="https://github.com/user-attachments/assets/4da95f8f-4c12-4743-8611-eaf11cb10a00" />


## pip를 사용한 간단한 Dockerfile Simple Dockerfile with pip
아래 내용 Dockerfile파일안에 저장하고 
```dockerfile
# base Docker image that we will build on
FROM python:3.13.11-slim

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas pyarrow

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination
COPY pipeline.py pipeline.py

# define what to do first when the container runs
# in this example, we will just run the script
ENTRYPOINT ["python", "pipeline.py"]
```
```
cd ./pipeline/
docker run -t test:pandas .
```
<img width="1441" height="586" alt="image" src="https://github.com/user-attachments/assets/e688a0da-7192-4033-b13a-57ef40bbe9e1" />
오류 나므로 
```
docker build -t test:pandas .
```
를 먼저 한다. 

설명:**Explanation:**
- `FROM`: Base image (Python 3.13) 기본 이미지 (파이썬 3.13)
- `RUN`: Execute commands during build 빌드 중에 명령을 실행합니다.
- `WORKDIR`: Set working directory 작업 디렉토리 설정
- `COPY`: Copy files into the image 파일을 이미지 폴더에 복사합니다
- `ENTRYPOINT`: Default command to run 실행할 기본 명령어

### 빌드 및 실행 Build and Run

이미지를 만들어 봅시다: Let's build the image:
<img width="1199" height="261" alt="image" src="https://github.com/user-attachments/assets/b1479f46-bdc4-4967-bc60-1d2fe9cbc4e3" />

<img width="916" height="234" alt="image" src="https://github.com/user-attachments/assets/97a12d9a-9ef6-40b8-942e-64bc0252c387" />

```bash
docker build -t test:pandas .
```

* 이미지 이름은 test이고 태그는 입니다 pandas. 태그가 지정되지 않으면 기본값인 로 사용됩니다 latest
* The image name will be `test` and its tag will be `pandas`. If the tag isn't specified it will default to `latest`.

이제 컨테이너를 실행하고 인수를 전달하여 파이프라인이 해당 인수를 받을 수 있도록 할 수 있습니다.
We can now run the container and pass an argument to it, so that our pipeline will receive it:

```bash
docker run -it test:pandas some_number
```

파이프라인 스크립트를 단독으로 실행했을 때와 동일한 출력이 나와야 합니다.
You should get the same output you did when you ran the pipeline script by itself.

> 참고: 이 지침은 pipeline.py와 가 Dockerfile같은 디렉토리에 있다고 가정합니다. Docker 명령어 또한 이 파일들이 있는 동일한 디렉토리에서 실행해야 합니다.
> Note: these instructions assume that `pipeline.py` and `Dockerfile` are in the same directory. The Docker commands should also be run from the same directory as these files.

## uv를 사용한 Dockerfile Dockerfile with uv

uv는 어때요? pip 대신 uv를 사용해 봅시다. What about uv? Let's use it instead of using pip:

```dockerfile
# Start with slim Python 3.13 image
FROM python:3.13.10-slim

# Copy uv binary from official uv image (multi-stage build pattern)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

# Set working directory
WORKDIR /app

# Add virtual environment to PATH so we can use installed packages
ENV PATH="/app/.venv/bin:$PATH"

# Copy dependency files first (better layer caching)
COPY "pyproject.toml" "uv.lock" ".python-version" ./
# Install dependencies from lock file (ensures reproducible builds)
RUN uv sync --locked

# Copy application code
COPY pipeline.py pipeline.py

# Set entry point
ENTRYPOINT ["uv", "run", "python", "pipeline.py"]
```

**[↑ Up](README.md)** | **[← Previous](02-virtual-environment.md)** | **[Next →](04-postgres-docker.md)**
