# vLLM + Open WebUI 통합 실행 환경

이 프로젝트는 [vLLM](https://github.com/vllm-project/vllm)을 사용하여 고성능 LLM(거대 언어 모델) API 서버를 구축하고, [Open WebUI](https://github.com/open-webui/open-webui)를 통해 사용자 친화적인 채팅 인터페이스를 제공하는 방법을 설명합니다. 모든 환경은 Docker를 통해 컨테이너화되어 있어 간편하게 배포하고 관리할 수 있습니다이 저장소는 [vLLM](https://github.com/vllm-project/vllm)을 기반으로 한 LLM 추론 서버와
[Open WebUI](https://github.com/open-webui/open-webui)를 함께 실행할 수 있도록 구성된 Docker Compose 환경입니다.

로컬에 저장된 SFT 모델을 빠르게 배포하고, Web UI를 통해 사용자 친화적인 인터페이스로 활용할 수 있습니다.

---

## 🧱 프로젝트 구조

```bash
.
├── model/
│   └── midm/         # 👈 여기에 사용할 LLM 모델 파일들을 위치시킵니다.
├── docker-compose.yml
└── Dockerfile
```

---

## 🚀 사용 방법

### 1. 모델 준비

사용하고자 하는 LLM을 Hugging Face Hub 등에서 다운로드하여 위 `model/midm` 디렉토리에 위치시킵니다.

### 2. Docker 이미지 빌드

vLLM을 실행할 Docker 이미지를 빌드합니다. `Dockerfile`이 있는 경로에서 아래 명령어를 실행하세요.


```
docker build -t vllm-env .
```

### 3. 서비스 실행

`docker-compose.yml` 파일이 있는 경로에서 아래 명령어를 실행하여 vLLM API 서버와 Open WebUI를 동시에 실행합니다. `-d` 옵션은 백그라운드에서 실행하도록 합니다.


```
docker-compose up -d
```

### 4. Open WebUI 접속 및 사용

모든 서비스가 정상적으로 실행되면 웹 브라우저를 열고 다음 주소로 접속합니다.

* **URL**: `http://localhost:8080`

처음 접속 시 계정을 생성해야 합니다. 계정 생성 후 로그인하면 채팅 화면이 나타납니다.

채팅 화면 상단의 모델 선택 드롭다운 메뉴를 클릭하면 vLLM을 통해 서빙되고 있는 모델 (`midm`)이 표시됩니다. 해당 모델을 선택하고 바로 채팅을 시작할 수 있습니다.

## 🛠️ 설정 및 사용자화

`docker-compose.yml` 파일의 `vllm` 서비스 섹션에서 다양한 옵션을 조정하여 환경을 최적화할 수 있습니다.

* **모델 변경**:
  * `command`의 `--model /model/midm` 부분을 새로운 모델이 위치한 디렉토리 이름으로 변경합니다. (예: `--model /model/my-awesome-model`)
* **GPU 리소스 설정**:
  * `--tensor-parallel-size`: 여러 GPU를 사용하여 모델을 병렬로 실행할 경우 GPU 개수를 지정합니다.
  * `--gpu-memory-utilization`: GPU 메모리 사용률을 0.0에서 1.0 사이 값으로 설정합니다. (예: 90% 사용 시 `0.9`)
* **성능 관련 설정**:
  * `--max-model-len`: 모델이 처리할 수 있는 최대 시퀀스 길이를 설정합니다. 모델의 사양에 맞춰 조정해야 합니다.
  * `--max-num-seqs`: 한 번에 처리할 수 있는 최대 시퀀스 수를 지정합니다.

**예시 (GPU 메모리 90% 사용, 텐서 병렬화 1로 설정):**


```
    command: >
      python -m vllm.entrypoints.openai.api_server
      --model /model/midm
      --port 8000 
      --max-model-len 2048
      --tensor-parallel-size 1 
      --gpu-memory-utilization 0.9 # 20% -> 90%
      --max-num-seqs 8
      --kv-cache-dtype fp8
```

설정을 변경한 후에는 `docker-compose up -d --build` 명령어로 서비스를 다시 시작하여 변경 사항을 적용해야 합니다.

## ⚙️ 환경설정 요약

### 📌 `docker-compose.yml` 주요 옵션

#### vllm 서비스


| 항목                       | 설명                                |
| -------------------------- | ----------------------------------- |
| `--model`                  | 로컬에 저장된 SFT 모델 경로 지정    |
| `--gpu-memory-utilization` | GPU 메모리 사용률 제한              |
| `--kv-cache-dtype`         | 캐시 데이터 형식 (예: fp8, fp16 등) |

#### open-webui 서비스


| 환경변수              | 설명                               |
| --------------------- | ---------------------------------- |
| `OPENAI_API_BASE_URL` | vLLM API 경로 지정                 |
| `USE_CUDA_DOCKER`     | Docker 내 GPU 사용 설정            |
| `WEBUI_SECRET_KEY`    | 인증 키 (간단히 EMPTY로 설정 가능) |


---

## 📜 라이선스

- vLLM: Apache 2.0
- Open WebUI: Apache 2.0
- 본 프로젝트는 연구 및 개발 용도로 구성되었습니다.
