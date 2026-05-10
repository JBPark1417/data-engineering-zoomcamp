강의 : https://www.youtube.com/watch?v=lP8xXebHmuE&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=11


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


# Introduction to Docker

**[↑ Up](README.md)** | **[← Previous](README.md)** | **[Next →](02-virtual-environment.md)**

Docker is a _containerization software_ that allows us to isolate software in a similar way to virtual machines but in a much leaner way.

A Docker image is a _snapshot_ of a container that we can define to run our software, or in this case our data pipelines. By exporting our Docker images to Cloud providers such as Amazon Web Services or Google Cloud Platform we can run our containers there.

## Why Docker?

Docker provides the following advantages:

- Reproducibility: Same environment everywhere
- Isolation: Applications run independently
- Portability: Run anywhere Docker is installed

They are used in many situations:

- Integration tests: CI/CD pipelines
- Running pipelines on the cloud: AWS Batch, Kubernetes jobs
- Spark: Analytics engine for large-scale data processing
- Serverless: AWS Lambda, Google Functions

## repo 만들기
### 00:00 Docker and Python prerequisites for the workshop
도커와 Python을 설치해야 하고 github account를 만들어야 한다. 
- git hub에서 repository 만들기 
<img width="1207" height="437" alt="image" src="https://github.com/user-attachments/assets/5c478b62-eeed-45ce-b292-33a92fad9ae2" />
<img width="1207" height="508" alt="image" src="https://github.com/user-attachments/assets/2be30010-2670-4bfd-97aa-02fa0eb7435f" />
    - 난 data-engineering-learning repo만들었음
    - open in VS code를 권장함
- Visual code 설치 하고 github와 연결
- 명령어실행
```
@JBPark1417 ➜ /workspaces/data-engineering-learning (main) $ python -V
```
<img width="758" height="52" alt="image" src="https://github.com/user-attachments/assets/ce21db94-41a5-497a-ae97-e8fefe26ef77" />

    
```
 @JBPark1417 ➜ /workspaces/data-engineering-learning (main) $ docker
```
<img width="758" height="463" alt="image" src="https://github.com/user-attachments/assets/57787d15-3cc2-46ed-a746-a374fcdfc482" />

    
```
@JBPark1417 ➜ /workspaces/data-engineering-learning (main) $ PS1="> " #앞에 파일경로가 짧아짐, 근데 다른 터미널 열면 안바뀜
```
<img width="777" height="48" alt="image" src="https://github.com/user-attachments/assets/471bd42f-262d-46c7-a37c-4112766453c8" />

    >

```
> echo 'PS1="> "' > ~/.bashrc #이렇게 하면 앞으로 모든 작업 다 >만 뜸
```

## Basic Docker Commands
### 강의 09:33 Core definition of Docker containerization
### 강의 11:42 Docker container isolation and stateless behavior
docker는 host머신에서 분리된 환경을 제공한다 


먼저 docker가 제대로 설치되어 있는지 확인 
Check Docker version:

```bash
docker --version
```

### Run a simple container:
강의 12:12 First Docker command testing Docker run hello world
```bash
docker run hello-world #docker instance가 올바르게 구성되었는지 알수 있다.
```
<img width="860" height="342" alt="image" src="https://github.com/user-attachments/assets/14c57812-0147-476f-962f-fac60d7c7cea" />

Run something more complex:

```bash
docker run ubuntu
```
<img width="860" height="174" alt="image" src="https://github.com/user-attachments/assets/e7403589-b35b-4ffc-85f5-bc7578d30386" />
ubuntu가 다운로드만 되고 아무일도 일어나지 않는다. 
Nothing happens. Need to run it in `-it` mode. `-it` 하면 docker안으로 들어가서 앞의 경로가 바뀐다

### 14:45 Understanding stateless containers and Docker image snapshots 
```bash
docker run -it ubuntu 
```
root@8ad045b5d8cd:/#
<img width="848" height="50" alt="image" src="https://github.com/user-attachments/assets/c42efe43-b0fa-4df1-a6bc-2f7b3158e006" />


우리가 하려는건 docker안에 들어가서 별도로 작업하려는거임
We don't have `python` there so let's install it:
python을 확인하면 없어서 apt update로 설치

```bash
root@8ad045b5d8cd:/# apt update
root@8ad045b5d8cd:/# apt install python3
python3 -V
```
root@45531c9a111e:/# python3 -V
Python 3.14.4
<img width="653" height="50" alt="image" src="https://github.com/user-attachments/assets/dd22e795-c11b-4027-9c65-c8287ea133b4" />

ctr+D하면 원래 host machine으로 돌아감
host에서 python3 -V 하면 다른 버전이 나옴. 왜냐면 host와 docker에 깔린게 다르기 때문임
@JBPark1417 ➜ /workspaces/data-engineering-learning (main) $ python3 -V            
Python 3.12.1
<img width="727" height="49" alt="image" src="https://github.com/user-attachments/assets/2e909f01-6ad2-44ea-87ae-fe88dfb1eb1d" />

Docker Image: 앱 실행에 필요한 모든 파일과 설정을 묶은 읽기 전용 패키지, 레시피
Container: Image를 실제로 실행한 프로세스, 만들어진 음식

```
> docker run -it python:3.13.11
> docker run -it python:3.13.11-slim 더 슬림한 버전으로 해당 이미지 열면 빨리 작동
```
<img width="923" height="364" alt="image" src="https://github.com/user-attachments/assets/b62b2b70-ab32-45be-bf97-eead24bc3c96" />
설치함

## python대신 bash session을 사용하고 싶다면 어떻게 해야하나? → 진입점을 덮어쓸 수 있음
### 20:46 Data persistence using Docker volume mapping

``` > docker run -it --entrypoint=bash python:3.13.11-slim
```
이제 python:3.13.11-slim를 bash로 access할수 있게 된 것임. 직접 python docker container로 들어가면 python만 실행할 수 있는데 bash로 들어가면하나의 bash 방에 들어가서 파이썬을 쓰는 거라 그 안에서 다른 linux code도 쓸 수 있어 패키지 확인이나 디버깅이나 환경변수 확인이 편함 

```
root@0e24939a56fa:/# python -V
``` 하면 version이 3.13.11로 나오는데 원래는 다른 버전이었음
```entrypoint=bash
``` 하면 원래는 python:3.13.11이 자동 실행인데 그걸 bash를 자동실행 하고 해놓고 그 안에서 python을 실행하거나 폴더들 상황보거나 할 수 있게 하는게 entrypoint=bash안하면 ls 명령어 안먹음

  
## Stateless Containers

중요: Docker 컨테이너는 상태를 저장하지 않습니다. 컨테이너 내부에서 변경된 내용은 컨테이너가 종료되었다가 다시 시작될 때 저장되지 않습니다.
컨테이너를 종료하고 다시 사용하면 변경 사항이 사라집니다.
Important: Docker containers are stateless - any changes done inside a container will NOT be saved when the container is killed and started again.
When you exit the container and use it again, the changes are gone:

```bash
docker run -it ubuntu
python3 -V
```
이건 호스트 시스템에 영향을 주지 않기 때문에 좋은 점입니다. 예를 들어 다음과 같은 극단적인 작업을 한다고 가정해 봅시다.
This is good, because it doesn't affect your host system. Let's say you do something crazy like this:

```bash
docker run -it ubuntu
rm -rf / # don't run it on your computer!
```
다음에 실행하면 모든 파일이 복원됩니다.
Next time we run it, all the files are back.

## Managing Containers

하지만 이는 완전히 정확한 설명은 아닙니다 . 상태는 어딘가에 저장됩니다. 중지된 컨테이너를 확인할 수 있습니다. But, this is not _completely_ correct. The state is saved somewhere. We can see stopped containers:

```bash
docker ps -a # 그동안 사용한 컨테이너 리스트 보여줌
```
그중 하나를 다시 시작할 수도 있지만, 좋은 방법이 아니므로 그렇게 하지 않겠습니다. 공간을 차지하므로 삭제하겠습니다.
We can restart one of them, but we won't do it, because it's not a good practice. They take space, so let's delete them:

```bash
docker rm $(docker ps -aq) #현재 열려있는 컨테이너 삭제
docker rm `docker ps -aq` #같은 코드
```
다음에 무언가를 실행할 때 --rm을 추가합니다 .
Next time we run something, we add `--rm`:

```bash
docker run -it --rm ubuntu
```
-rm👉 컨테이너 종료하면 자동 삭제

## Different Base Images
hello-world와 외에도 다른 기본 이미지가 있습니다 ubuntu. 예를 들어, Python은 다음과 같습니다.
There are other base images besides `hello-world` and `ubuntu`. For example, Python:

```bash
docker run -it --rm python:3.9.16
# add -slim to get a smaller version
```
이것이 시작 부분입니다 python. bash를 사용하려면 덮어쓰기를 해야 합니다 entrypoint.
This one starts `python`. If we want bash, we need to overwrite `entrypoint`:

```bash
docker run -it \
    --rm \
    --entrypoint=bash \
    python:3.9.16-slim
```

## Volumes: 호스트에 있는 파일 docker container에 공유

So, we know that with docker we can restore any container to its initial state in a reproducible manner. But what about data? A common way to do so is with _volumes_.

test:에 데이터를 좀 만들어 봅시다 
Let's create some data in `test`:

```bash
mkdir test #test 폴더 만들기
cd test #test 폴더로 들어가기
touch file1.txt file2.txt file3.txt # touch = 빈 노트 만들기
echo "Hello from host" > file1.txt 
#echo "Hello from host" -> 문자열을 출력
# > (리다이렉션) 
# 원래는 화면에 출력될 내용이 file1.txt에 저장됨
# > 는 덮어쓰기, 이어쓰고 싶다면 >> 이용
cat file1.txt #cat file 👉 파일 읽기
cat ./file2.txt #./ = 현재 디렉토리 (지금 위치), 경로를 명확하게 할 때
```
해당 폴더에 script.py파일 하나 생성하고
```
python [script.py](http://script.py/) #script.py 파일을 Python으로 실행한다
```
[script.py](http://script.py/)를 list_files.py로 파일명 바꾸고 아래 코드 저장
Now let's create a simple script `test/list_files.py` that shows the files in the folder:

```python
from pathlib import Path

current_dir = Path.cwd()
current_file = Path(__file__).name

print(f"Files in {current_dir}:")

for filepath in current_dir.iterdir():
    if filepath.name == current_file:
        continue

    print(f"  - {filepath.name}")

    if filepath.is_file():
        content = filepath.read_text(encoding='utf-8')
        print(f"    Content: {content}")
```
이제 이것을 파이썬 컨테이너에 매핑해 보겠습니다.
Now let's map this to a Python container:

```bash
docker run -it --rm -v $(pwd)/test:/app/test --entrypoint=bash python:3.9.16-slim
```
- -it 👉 터미널처럼 사용 (입력 + 화면)
- --rm 👉 종료하면 컨테이너 자동 삭제
- -v $(pwd)/test:/app/test
    - 볼륨 매핑,
    - 의미: 내 컴퓨터 현재 위치/test→컨테이너/app/test,
    - 내 폴더를 Docker 안에 연결해서 파일을 공유한다
    - -v 때문에 host(내 컴퓨터) ↔ container(Docker) 가 연결됨
    - $(pwd)👉 현재 경로 출력
- --entrypoint=bash 👉 Python 대신 bash 실행
- python:3.9.16-slim 👉 Python 환경이 들어있는 이미지
- Docker는 원래 격리된 환경, 그래서 파일 공유하려면 → -v 필수
  
```bash
docker run -it \
    --rm \
    -v $(pwd)/test:/app/test \
    --entrypoint=bash \
    python:3.9.16-slim
```
컨테이너 내부에서 다음 명령을 실행하세요.
Inside the container, run:

```bash
cd /app/test
ls -la
cat file1.txt
python list_files.py
```
컨테이너 안에서 호스트 컴퓨터의 파일에 접근할 수 있는 것을 확인하실 수 있을 겁니다!
You'll see the files from your host machine are accessible in the container!

**[↑ Up](README.md)** | **[← Previous](README.md)** | **[Next →](02-virtual-environment.md)**
