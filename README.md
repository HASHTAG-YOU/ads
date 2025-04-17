---

# 실시간 항법 정보 처리 시스템

## Introduction

본 프로젝트는 사용자가 업로드한 선박 항법(GPS) 데이터를 검증 및 파싱하여 Redis에 저장하고, WebSocket을 통해 실시간으로 제공하는 백엔드 시스템입니다.

## Table of Contents

- [Development Environment and Tech Stack](#development-environment-and-tech-stack)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Testing](#testing)

## 1. Development Environment and Tech Stack

- **Programming Language:** Python 3.11
- **Framework:** FastAPI  
  → 경량 고성능 API 서버 구축을 위해 선택했습니다.
- **ASGI Server:** Uvicorn  
  → FastAPI와 최적화된 고성능 비동기 서버, WebSocket 통신 지원.
- **Database:** Redis  
  → 실시간 데이터 저장과 빠른 조회를 위한 인메모리 데이터베이스 사용.

### Core Libraries

- `fastapi`: 비동기 웹 API 서버 구축 프레임워크
- `uvicorn[standard]`: FastAPI 실행용 고성능 ASGI 서버
- `python-multipart`: 파일 업로드 요청 처리 지원
- `redis`: Redis 서버와의 연결 및 데이터 저장/조회 처리
- `pytest`: API 기능 및 예외 상황 자동화 테스트 프레임워크
- `httpx`: HTTP 요청 및 WebSocket 통신 테스트 클라이언트

---

## 2. Getting Started


### 전체 시스템 실행

아래 명령어로 FastAPI 서버, Redis 서버를 빌드 및 실행할 수 있습니다. 이후 `test_api.py`가 자동 실행되어 API 테스트를 완료합니다.

```bash
docker compose up --build
```


**접속 주소:**

- **Swagger UI:** [http://localhost:8000/docs](http://localhost:8000/docs)
- **Static Page (index.html):** [http://localhost:8000/](http://localhost:8000/)


※ 별도의 포트 설정이 필요한 경우, 루트 디렉터리의 .env 파일에서 FASTAPI_PORT 및 REDIS_PORT 값을 직접 수정하여 적용할 수 있습니다.

### 요약 명령어

| 명령어 | 설명 |
|:-------|:-----|
| `docker compose up --build` | FastAPI + Redis + API 테스트 실행 |
| `docker compose up api redis --build` | FastAPI + Redis만 실행 |

---

## 3. Project Structure

```
.
├── .env                  # 환경 변수 파일 (FastAPI 포트, Redis 포트 설정)
├── docker-compose.yml    # FastAPI + Redis를 함께 실행하는 Docker 설정 파일
├── Dockerfile            # FastAPI 서버 Docker 이미지 빌드 파일
├── requirements.txt      # 프로젝트에 필요한 Python 라이브러리 목록
├── static/
│   └── index.html         # 기본 정적 페이지 (파일 업로드 UI)
├── app/
│   ├── main.py            # FastAPI 엔트리포인트 (라우팅, 예외 처리 설정)
│   ├── exception.py       # 커스텀 예외 및 에러 코드 관리
│   ├── gga_parser.py      # GGA 문장 파싱 로직
│   ├── redis_handler.py   # Redis 연결 및 데이터 저장/조회 처리
│   ├── websocket.py       # WebSocket을 통한 실시간 데이터 전송 처리
│   └── schemas.py         # 에러 응답 스키마 정의 (Pydantic 모델)
├── test/                  
│   ├── valid_gga.txt              # 정상 GGA 문장 테스트 파일
│   ├── invalid_extension.pdf      # 잘못된 확장자 테스트 파일
│   ├── invalid_encoding.txt       # UTF-8 인코딩 오류 테스트 파일
│   ├── large_file.txt              # 10MB 초과 테스트 파일
│   ├── no_gpgga.txt                # GPGGA 문장 없는 테스트 파일
│   ├── all_checksum_failed.txt     # 체크섬 오류 테스트 파일
│   ├── gga_parsing_failed.txt      # GGA 문장 파싱 오류 테스트 파일
│   ├── mixed_valid_invalid.txt     # 정상+오류 문장이 섞인 테스트 파일
│   └── example.txt                 # 정상 테스트 파일
├── test_api.py            # 주요 API 기능 검증용 pytest 스크립트
```

---

4. Testing

테스트 실행 방법

아래 명령어로 API 서버와 Redis 서버가 준비된 상태에서 테스트를 실행할 수 있습니다.

docker compose up test_api --build

서버가 이미 실행 중이라면 --build 없이 docker compose up test_api만으로도 테스트를 재실행할 수 있습니다.

4.2. 테스트 항목
	•	정상 GGA 문장 업로드: 유효한 GGA 데이터가 정상적으로 저장되는지 검증
	•	잘못된 파일 확장자 업로드: .pdf 등 잘못된 확장자 파일 업로드 시 오류 처리 검증
	•	파일 크기 초과 오류: 10MB를 초과하는 파일 업로드 시 예외 처리 검증
	•	UTF-8 인코딩 오류 파일 업로드: 비정상 인코딩 파일 업로드 시 오류 처리 검증
	•	GPGGA 문장이 없는 파일 업로드: 데이터가 없는 경우 적절한 에러 응답 검증
	•	모든 GGA 문장의 체크섬 검증 실패: 체크섬이 실패한 모든 문장에 대한 예외 처리 검증
	•	GGA 문장 형식 파싱 실패: 잘못된 포맷의 문장을 처리할 때 발생하는 예외 검증
	•	WebSocket 정상 연결 및 데이터 수신: WebSocket을 통한 실시간 데이터 수신 정상 여부 검증

4.3. 테스트 데이터

테스트 케이스별로 다음 파일들을 사용합니다.

test/
├── valid_gga.txt              # 정상 GGA 문장
├── invalid_extension.pdf      # 잘못된 파일 확장자
├── invalid_encoding.txt       # UTF-8 인코딩 오류 파일
├── large_file.txt             # 10MB 초과 파일
├── no_gpgga.txt               # GPGGA 문장이 없는 파일
├── all_checksum_failed.txt    # 모든 GGA 문장의 체크섬 오류
├── gga_parsing_failed.txt     # GGA 문장 형식 오류
├── mixed_valid_invalid.txt    # 정상/오류 GGA 문장이 섞인 파일



좋아, 전체 형식과 문법, 들여쓰기까지 통일성 있게 완전히 다듬었어.
바로 저장해서 README로 쓸 수 있도록 깔끔하게 정리해줄게:

⸻

실시간 항법 정보 처리 시스템

Introduction

본 프로젝트는 사용자가 업로드한 선박 항법(GPS) 데이터를 검증 및 파싱하여 Redis에 저장하고, WebSocket을 통해 실시간으로 제공하는 백엔드 시스템입니다.

Table of Contents
	•	1. Development Environment and Tech Stack
	•	2. Getting Started
	•	3. Project Structure
	•	4. Testing

⸻

1. Development Environment and Tech Stack
	•	Programming Language: Python 3.11
	•	Framework: FastAPI
→ 경량 고성능 API 서버 구축을 위해 선택했습니다.
	•	ASGI Server: Uvicorn
→ FastAPI와 최적화된 고성능 비동기 서버, WebSocket 통신 지원.
	•	Database: Redis 7.2
→ 실시간 데이터 저장과 빠른 조회를 위한 인메모리 데이터베이스 사용.

Core Libraries
	•	fastapi: 비동기 웹 API 서버 구축 프레임워크
	•	uvicorn[standard]: FastAPI 실행용 고성능 ASGI 서버
	•	python-multipart: 파일 업로드 요청(multipart/form-data) 지원
	•	redis: Redis 서버와의 연결 및 데이터 저장/조회 처리
	•	pytest: API 기능 및 예외 상황 자동화 테스트 프레임워크
	•	httpx: HTTP 요청 및 WebSocket 통신 테스트 클라이언트

⸻

2. Getting Started

2.1. 전체 시스템 실행

아래 명령어로 FastAPI 서버와 Redis 서버를 빌드 및 실행할 수 있습니다. 이후 test_api.py가 자동 실행되어 API 테스트가 완료됩니다.

docker compose up --build

접속 주소:
	•	Swagger UI: http://localhost:8000/docs
	•	Static Page (index.html): http://localhost:8000/

※ 별도의 포트 설정이 필요한 경우, 루트 디렉터리의 .env 파일에서 FASTAPI_PORT, REDIS_PORT 값을 수정할 수 있습니다.

FASTAPI_PORT=8081
REDIS_PORT=6384

2.2. 요약 명령어

명령어	설명
docker compose up --build	FastAPI + Redis + API 테스트까지 전체 실행
docker compose up api redis --build	FastAPI 서버와 Redis만 실행 (테스트 제외)



⸻

3. Project Structure

.
├── .env                  # 환경 변수 파일 (FastAPI 포트, Redis 포트 설정)
├── docker-compose.yml    # FastAPI + Redis를 함께 실행하는 Docker 설정 파일
├── Dockerfile            # FastAPI 서버를 Docker 이미지로 빌드하기 위한 파일
├── requirements.txt      # 프로젝트에 필요한 Python 라이브러리 목록
├── static/
│   └── index.html         # 파일 업로드용 기본 정적 페이지
├── app/
│   ├── main.py            # FastAPI 엔트리포인트 (라우팅 및 예외 처리 설정)
│   ├── exception.py       # 커스텀 예외 및 에러 코드 관리
│   ├── gga_parser.py      # GGA 문장 파싱 로직
│   ├── redis_handler.py   # Redis 연결 및 데이터 저장/조회 로직
│   ├── websocket.py       # WebSocket을 통한 실시간 데이터 전송 처리
│   └── schemas.py         # 에러 응답 스키마 정의 (Pydantic 모델)
├── test/
│   ├── valid_gga.txt              # 정상 GGA 문장 테스트 파일
│   ├── invalid_extension.pdf      # 잘못된 확장자 테스트 파일
│   ├── invalid_encoding.txt       # UTF-8 인코딩 오류 테스트 파일
│   ├── large_file.txt              # 10MB 초과 테스트 파일
│   ├── no_gpgga.txt                # GPGGA 문장이 없는 테스트 파일
│   ├── all_checksum_failed.txt     # 체크섬 오류 테스트 파일
│   ├── gga_parsing_failed.txt      # GGA 문장 파싱 오류 테스트 파일
│   ├── mixed_valid_invalid.txt     # 정상+오류 문장이 섞인 테스트 파일
│   └── example.txt                 # 예제 파일 (테스트 미사용)
├── test_api.py            # 주요 API 기능 검증용 pytest 테스트 스크립트



⸻

4. Testing

4.1. 테스트 실행 방법

API 서버와 Redis 서버가 준비된 상태에서 테스트를 실행하려면 다음 명령어를 사용합니다.

docker compose up test_api --build

※ 서버가 이미 실행 중이라면 --build 없이 docker compose up test_api만으로 재실행할 수 있습니다.

4.2. 테스트 항목
	•	정상 GGA 문장 업로드: 유효한 GGA 데이터가 정상적으로 저장되는지 검증
	•	잘못된 파일 확장자 업로드: .pdf 파일 등 비정상 확장자 업로드 오류 검증
	•	파일 크기 초과 오류: 10MB를 초과하는 파일 업로드 시 예외 처리 검증
	•	UTF-8 인코딩 오류 업로드: 비정상 인코딩 파일 업로드 시 오류 처리 검증
	•	GPGGA 문장 없는 파일 업로드: GGA 데이터가 없는 파일 처리 검증
	•	모든 GGA 문장의 체크섬 실패: 체크섬 검증에 실패하는 데이터 처리 검증
	•	GGA 문장 형식 파싱 실패: 잘못된 포맷의 GGA 데이터에 대한 예외 처리 검증
	•	WebSocket 정상 연결 및 데이터 수신 테스트: 실시간 데이터 송수신 정상 여부 검증

4.3. 테스트 데이터 파일

테스트 케이스별로 다음 파일들을 사용합니다.

test/
├── valid_gga.txt
├── invalid_extension.pdf
├── invalid_encoding.txt
├── large_file.txt
├── no_gpgga.txt
├── all_checksum_failed.txt
├── gga_parsing_failed.txt
├── mixed_valid_invalid.txt


