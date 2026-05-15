https://www.youtube.com/watch?v=lP8xXebHmuE&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=12
56:30 데이터베이스 관리를 위해 Docker에서 Postgres 실행

# Running PostgreSQL with Docker

**[↑ Up](README.md)** | **[← Previous](03-dockerizing-pipeline.md)** | **[Next →](05-data-ingestion.md)**

이제 본격적인 데이터 엔지니어링을 해봅시다. 이를 위해 포스트그레스 데이터베이스를 사용해 보겠습니다. Now we want to do real data engineering. Let's use a Postgres database for that.

별도의 설치 과정 없이 컨테이너화된 PostgreSQL 버전을 실행할 수 있습니다. 몇 가지 환경 변수 와 데이터 저장을 위한 볼륨 만 제공하면 됩니다 .
You can run a containerized version of Postgres that doesn't require any installation steps. You only need to provide a few _environment variables_ to it as well as a _volume_ for storing data.

## 컨테이너에서 PostgreSQL 실행하기Running PostgreSQL in a Container

PostgreSQL이 데이터를 저장할 폴더를 원하는 위치에 생성하세요. 여기서는 example 폴더를 사용하겠습니다 ny_taxi_postgres_data. 컨테이너를 실행하는 방법은 다음과 같습니다.  Create a folder anywhere you'd like for Postgres to store data in. We will use the example folder `ny_taxi_postgres_data`. Here's how to run the container:

```bash
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```
### 매개변수에 대한 설명 Explanation of Parameters

* `-e` 환경 변수(사용자 이름, 비밀번호, 데이터베이스 이름)를 설정합니다  sets environment variables (user, password, database name)
    *   -e POSTGRES_USER="root" \ # 아이디
    *   -e POSTGRES_PASSWORD="root" \ # 비번
    *   -e POSTGRES_DB="ny_taxi" \   #DB이름
* `-v ny_taxi_postgres_data:/var/lib/postgresql` creates a **명명된 볼륨을 생성합니다.named volume**
  * Docker는 이 볼륨을 자동으로 관리합니다. Docker manages this volume automatically
  * 컨테이너가 제거된 후에도 데이터는 유지됩니다 Data persists even after container is removed
  * 볼륨은 Docker의 내부 저장소에 저장됩니다. Volume is stored in Docker's internal storage
  * 우리가 이전에 볼륨매핑을 $ docker run -it --rm -v $(pwd)/test:/app/test --entrypoint=bash python:3.9.16-slim
 코드로 pwd라는 명령어를 사용해서 절대 경로를 지정했습니다. 이름은 없고요. 슬래시(/)도 없고, 그냥 이름만 있습니다. 이렇게 해서 이 Docker 볼륨은 Docker의 내부 볼륨이 됩니다 따라서 내부 내용을 볼 수 없게 되는데, 이는 좋은 점입니다.파일 내용에 간섭할 수 없게 되니까요. 이것들은 진행 과정 내부에 있지만, 이 라인은 실제로 꽤 중요합니다. 왜냐하면 이것으로 저희는 데이터를 보호할 수 있기 때문입니다. 따라서 우리는 원하는 데이터를 진행 상황에 넣을 수 있으며 다음에 실행할 때도 그 데이터는 그대로 남아 있을 것입니다. 
* `-p 5432:5432` 컨테이너에서 호스트로 5432번 포트를 매핑합니다. maps port 5432 from container to host
  * <img width="893" height="478" alt="image" src="https://github.com/user-attachments/assets/7aa69bba-5ba3-4728-a990-d100d6b22eb8" />

* `postgres:18` PostgreSQL 버전 18(2025년 12월 기준 최신 버전)을 사용합니다. uses PostgreSQL version 18 (latest as of Dec 2025)

<img width="940" height="589" alt="image" src="https://github.com/user-attachments/assets/27681bd8-4d7a-4210-a33c-3e8dc42fd748" />
이제 database는 ready to connect
new terminal을 열고 
$ docker build -t test:pandas .
$ docker run -it --entrypoint=bash --rm test:pandas 
$ ls
root@8c5c891442f6:/app# uv add --dev pgcli
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi


<img width="1262" height="534" alt="image" src="https://github.com/user-attachments/assets/fa39e8c8-38b6-4ca9-b548-adc5ff9d4d9d" />
<img width="909" height="478" alt="image" src="https://github.com/user-attachments/assets/9771035e-7f9c-4326-ae70-b8ce4de1177f" />


### 대안적 접근 방식 - 바인드 마운트 Alternative Approach - Bind Mount

먼저 디렉토리를 생성한 다음 매핑하십시오. First create the directory, then map it:

```bash
mkdir ny_taxi_postgres_data

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```

### 명명된 볼륨과 바인드 마운트의 차이점 Named Volume vs Bind Mount

* **명명된 볼륨 Named volume** (`name:/path`): Docker에서 관리되므로 사용이 더 간편합니다. Managed by Docker, easier
* **바인드 마운트 Bind mount** (`/host/path:/container/path`): 호스트 파일 시스템에 직접 매핑하여 더 많은 제어가 가능합니다. Direct mapping to host filesystem, more control

## PostgreSQL에 연결 중 Connecting to PostgreSQL

컨테이너가 실행되면 pgcli를 사용하여 데이터베이스에 로그인할 수 있습니다  Once the container is running, we can log into our database with [pgcli](https://www.pgcli.com/).

pgcli를 설치하세요 Install pgcli:

```bash
uv add --dev pgcli
```

이 플래그는 해당 항목이 개발 종속성(프로덕션 환경에서는 필요하지 않음)임을 나타냅니다. 이 항목은 메인 섹션 대신 다른 섹션 --dev에 추가됩니다 The `--dev` flag marks this as a development dependency (not needed in production). It will be added to the `[dependency-groups]` section of `pyproject.toml` instead of the main `dependencies` section.

이제 이 정보를 사용하여 Postgres에 연결하세요 Now use it to connect to Postgres:

```bash
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi
```

* `uv run` 가상 환경의 컨텍스트에서 명령을 실행합니다. executes a command in the context of the virtual environment
* `-h` 호스트입니다. 로컬에서 실행 중이므로 를 사용할 수 있습니다  is the host. Since we're running locally we can use `localhost`.
* `-p` 그곳이 항구입니다. is the port.
* `-u` 사용자 이름입니다. is the username.
* `-d` 데이터베이스 이름입니다.is the database name.
* 비밀번호는 제공되지 않으며, 명령 실행 후 입력하라는 메시지가 나타납니다. The password is not provided; it will be requested after running the command.

암호를 묻는 메시지가 나타나면 암호를 입력하십시오 When prompted, enter the password: `root`

## 기본 SQL 명령어 Basic SQL Commands

SQL 명령어를 몇 개 시도해 보세요. Try some SQL commands:

```sql
-- List tables
\dt

-- Create a test table
CREATE TABLE test (id INTEGER, name VARCHAR(50));

-- Insert data
INSERT INTO test VALUES (1, 'Hello Docker');

-- Query data
SELECT * FROM test;

-- Exit
\q
```

**[↑ Up](README.md)** | **[← Previous](03-dockerizing-pipeline.md)** | **[Next →](05-data-ingestion.md)**
