---

# Avikus Backend Pre-assignment

## Introduction

본 프로젝트는 NMEA GGA 데이터를 실시간으로 처리하고, 파일 업로드 및 WebSocket 기반 전송 기능을 제공하는 FastAPI 기반 백엔드 시스템입니다.

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

### 1.1. Core Libraries

- `fastapi`: 비동기 웹 API 서버 구축 프레임워크
- `uvicorn[standard]`: FastAPI 실행용 고성능 ASGI 서버
- `python-multipart`: 파일 업로드 요청 처리 지원
- `redis`: Redis 서버와의 연결 및 데이터 저장/조회 처리
- `pytest`: API 기능 및 예외 상황 자동화 테스트 프레임워크
- `httpx`: HTTP 요청 및 WebSocket 통신 테스트 클라이언트

---

## 2. Getting Started


### 2.1. Docker로 전체 시스템 실행

아래 명령어로 FastAPI 서버, Redis 서버를 빌드 및 실행할 수 있습니다. 이후 `test_api.py`가 자동 실행되어 API 테스트를 완료합니다.

```bash
docker compose up --build
```


**접속 주소:**

- **Swagger UI:** [http://localhost:8000/docs](http://localhost:8000/docs)
- **Static Page (index.html):** [http://localhost:8000/](http://localhost:8000/)


(※ 별도 포트 지정이 필요한 경우 .env 파일을 수정하세요.)

```bash
FASTAPI_PORT=8081
REDIS_PORT=6384
```

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
│   ├── valid_gga.txt              # 정상 GGA 문장 샘플
│   ├── invalid_extension.pdf      # 잘못된 확장자 테스트 파일
│   ├── invalid_encoding.txt       # UTF-8 인코딩 오류 테스트 파일
│   ├── large_file.txt              # 10MB 초과 테스트 파일
│   ├── no_gpgga.txt                # GPGGA 문장 없는 테스트 파일
│   ├── all_checksum_failed.txt     # 체크섬 오류 테스트 파일
│   ├── gga_parsing_failed.txt      # GGA 문장 파싱 오류 테스트 파일
│   ├── mixed_valid_invalid.txt     # 정상+오류 문장이 섞인 테스트 파일
│   └── example.txt                 # 정상 예시 파일
├── test_api.py            # 주요 API 기능 검증용 pytest 스크립트
```

---

## 4. Testing

### 4.1. 테스트 실행 방법

```bash
docker compose up test_api --build
```

### 4.2. 테스트 항목

- 정상 GGA 문장 업로드
- 잘못된 파일 확장자 업로드 (.pdf 등)
- 파일 크기 초과 오류 (10MB 초과)
- UTF-8 인코딩 오류 파일 업로드
- GPGGA 문장이 없는 파일 업로드
- 모든 GGA 문장의 체크섬 검증 실패
- GGA 문장 형식 파싱 실패
- WebSocket 정상 연결 및 데이터 수신 테스트

### 테스트 데이터

테스트 파일 목록:

```
test/valid_gga.txt
test/invalid_extension.pdf
test/invalid_encoding.txt
test/large_file.txt
test/no_gpgga.txt
test/all_checksum_failed.txt
test/gga_parsing_failed.txt
test/mixed_valid_invalid.txt
```

---
