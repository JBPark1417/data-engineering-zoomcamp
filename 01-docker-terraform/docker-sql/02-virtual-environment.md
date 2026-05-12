# Virtual Environments and Data Pipelines

**[↑ Up](README.md)** | **[← Previous](01-introduction.md)** | **[Next →](03-dockerizing-pipeline.md)**

https://www.youtube.com/watch?v=lP8xXebHmuE&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=11
28:01 Data engineering overview for building data pipelines
38:17 Python environment management with UV
44:16 Creating a custom Docker image with a Dockerfile
48:08 Setting the ENTRYPOINT for automated container execution
56:30 Running Postgres in Docker for database management
1:01:36 Host access to Postgres via container port mapping
1:17:01 Pandas data processing for CSV schemaless data types
1:24:51 Optimized data ingestion with chunked reading
1:36:50 Building a command line interface with Click
1:48:19 Resolving container communication using Docker networks
1:59:39 Multi container orchestration with Docker Compose

데이터 파이프라인은 데이터를 입력으로 받아 더 많은 데이터를 출력하는 서비스입니다. 예를 들어 CSV 파일을 읽어 데이터를 변환한 후 PostgreSQL 데이터베이스의 테이블 형태로 저장하는 것이 데이터 파이프라인의 한 예입니다.
A **data pipeline** is a service that receives data as input and outputs more data. For example, reading a CSV file, transforming the data somehow and storing it as a table in a PostgreSQL database.

```mermaid
graph LR
    A[CSV File] --> B[Data Pipeline]
    B --> C[Parquet File]
    B --> D[PostgreSQL Database]
    B --> E[Data Warehouse]
    style B fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff
```
이번 워크숍에서는 다음과 같은 파이프라인을 구축합니다.In this workshop, we'll build pipelines that:

웹에서 CSV 데이터를 다운로드하세요- Download CSV data from the web
pandas를 사용하여 데이터를 변환하고 정리합니다.- Transform and clean the data with pandas
쿼리를 위해 PostgreSQL에 로드합니다.- Load it into PostgreSQL for querying
대용량 파일을 처리하기 위해 데이터를 청크 단위로 처리합니다.- Process data in chunks to handle large files

## 간단한 파이프라인 생성 Creating a Simple Pipeline

예제 파이프라인을 만들어 보겠습니다. 먼저 디렉토리를 생성 pipeline하고 그 안에 파일을 생성합니다 pipeline.py.
Let's create an example pipeline. First, create a directory `pipeline` and inside, create a file  `pipeline.py`:

```python
import sys
print("arguments", sys.argv)

day = int(sys.argv[1])
print(f"Running pipeline for day {day}")
```
이제 팬더스를 추가해 보겠습니다.
Now let's add pandas:

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4]})
print(df.head())

df.to_parquet(f"output_day_{sys.argv[1]}.parquet")
```

## 가상 환경이 필요한 이유는 무엇일까요? Why Virtual Environments?

우리는 pandas가 필요하지만, 현재 설치되어 있지 않습니다. 컨테이너에서 실행하기 전에 pandas를 테스트하고 싶습니다.
We need pandas, but we don't have it. We want to test it before we run things in a container.

다음 명령어로 설치할 수 있습니다 We can install it with `pip`:

```bash
pip install pandas pyarrow
```

하지만 이렇게 하면 해당 패키지가 시스템 전체에 설치됩니다. 따라서 서로 다른 프로젝트에서 동일한 패키지의 다른 버전을 필요로 하는 경우 충돌이 발생할 수 있습니다.
But this installs it globally on your system. This can cause conflicts if different projects need different versions of the same package.

대신, 우리는 가상 환경 , 즉 이 프로젝트의 종속성을 다른 프로젝트 및 시스템 Python과 분리하는 격리된 Python 환경을 사용하려고 합니다 .
Instead, we want to use a **virtual environment** - an isolated Python environment that keeps dependencies for this project separate from other projects and from your system Python.

## uv - 최신 파이썬 패키지 관리자를 사용합니다. Using uv - Modern Python Package Manager
uv우리는 Rust로 작성된 최신 고속 Python 패키지 및 프로젝트 관리 도구인 `pip`을 사용할 것입니다 . `pip`은 pip보다 훨씬 빠르고 가상 환경을 자동으로 처리합니다.
We'll use `uv` - a modern, fast Python package and project manager written in Rust. It's much faster than pip and handles virtual environments automatically.

```bash
pip install uv
```

이제 uv를 사용하여 Python 프로젝트를 초기화하세요. Now initialize a Python project with uv:

```bash
uv init --python=3.13
```

pyproject.toml이렇게 하면 종속성 관리를 위한 파일과 또 다른 파일이 생성됩니다 .python-version. 
This creates a `pyproject.toml` file for managing dependencies and a `.python-version` file.

### 파이썬 버전 비교 Comparing Python Versions

```bash
uv run which python  # Python in the virtual environment
uv run python -V

which python        # System Python
python -V
```

보시면 아시겠지만, 두 제품은 uv run격리된 환경을 사용합니다. You'll see they're different - `uv run` uses the isolated environment.

### 종속성 추가 Adding Dependencies

이제 팬더스를 추가해 보겠습니다. Now let's add pandas:

```bash
uv add pandas pyarrow
```

이렇게 하면 pandas가 사용자 환경에 추가되고 pyproject.toml가상 환경에 설치됩니다. This adds pandas to your `pyproject.toml` and installs it in the virtual environment.

### 파이프라인 실행 Running the Pipeline

이제 파일을 실행할 수 있습니다. Now we can execute the file:

```bash
uv run python pipeline.py 10
```

두고 보면 알겠죠: We will see:

* `['pipeline.py', '10']`
* `job finished successfully for day = 10`

## Git 설정 Git Configuration

이 스크립트는 바이너리(parquet) 파일을 생성하므로, 실수로 Git에 커밋하지 않도록 parquet 확장자를 추가해 두겠습니다 .gitignore.
This script produces a binary (parquet) file, so let's make sure we don't accidentally commit it to git by adding parquet extensions to `.gitignore`:

```
*.parquet
```

**[↑ Up](README.md)** | **[← Previous](01-introduction.md)** | **[Next →](03-dockerizing-pipeline.md)**
