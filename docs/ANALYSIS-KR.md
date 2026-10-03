# RustDesk 전수조사 분석 및 활용 가이드 (한국어)

> 이 문서는 `bmshin94/rustdesk` 저장소를 코드 레벨까지 전수조사한 결과와,
> 설치·사용·수익화·AI 에이전트 활용 방안을 정리한 것입니다.
> 작성일: 2026-10-03 / 기준 버전: `1.5.0`

## 관련 GitHub 주소

| 구분 | 주소 |
|---|---|
| 이 저장소 (포크) | https://github.com/bmshin94/rustdesk |
| 원본 업스트림 | https://github.com/rustdesk/rustdesk |
| 서버 (hbbs/hbbr) | https://github.com/rustdesk/rustdesk-server |
| 서버 데모 (직접 구현용) | https://github.com/rustdesk/rustdesk-server-demo |
| 공유 서브모듈 | https://github.com/rustdesk/hbb_common |
| 공식 문서 | https://github.com/rustdesk/doc.rustdesk.com |
| 바이너리 다운로드 | https://github.com/rustdesk/rustdesk/releases |
| 나이틀리 빌드 | https://github.com/rustdesk/rustdesk/releases/tag/nightly |
| FAQ 위키 | https://github.com/rustdesk/rustdesk/wiki/FAQ |
| Flathub | https://flathub.org/apps/com.rustdesk.RustDesk |
| F-Droid (Android) | https://f-droid.org/en/packages/com.carriez.flutter_hbb |
| 공식 사이트 / 가격 | https://rustdesk.com · https://rustdesk.com/pricing.html |
| 번역 디렉터리 | https://github.com/rustdesk/rustdesk/tree/master/src/lang |

---

## 1. 이게 뭔가

**RustDesk** = TeamViewer / AnyDesk를 대체하는 **오픈소스 원격 데스크톱 프로그램**의 전체 소스코드.
이 저장소는 원본 `rustdesk/rustdesk`를 포크한 것이며, `CLAUDE.md` / `AGENTS.md`(AI 에이전트 작업 규칙)가
추가되어 있다 (`Merge PR #1: docs: add CLAUDE.md project guide`).

| 항목 | 값 |
|---|---|
| 버전 | 1.5.0 |
| 라이선스 | AGPL-3.0 |
| 저장소 크기 | 약 20MB (.git 제외) |
| 지원 플랫폼 | Windows / macOS / Linux / Android / iOS / Web |
| 지원 언어 | 55개 (`src/lang/`) |

### 코드 규모 (실측)

| 언어 | 파일 수 | 줄 수 |
|---|---|---|
| Rust | 286 | 170,494 |
| Dart (Flutter) | 136 | 76,084 |
| Python | 15 | 5,617 |
| C / C++ / Obj-C | 24 | 11,861 |
| Markdown | 81 | 7,397 |
| YAML (CI) | 17 | 4,001 |

총 **25만 줄 이상의 상용급 제품**. 라이브러리나 예제가 아니라 완성된 애플리케이션.

---

## 2. 폴더 구조 (직접 확인)

```
rustdesk/
├── src/                          Rust 본체
│   ├── client.rs (226KB)         원격 접속하는 쪽
│   ├── server.rs (33KB)          제어당하는 쪽
│   ├── rendezvous_mediator.rs    P2P 연결 핵심 (92KB)
│   ├── flutter_ffi.rs (96KB)     Rust <-> Flutter 브리지
│   ├── ipc.rs (85KB)             프로세스 간 통신
│   ├── keyboard.rs (58KB)        키보드 입력 변환
│   ├── clipboard.rs / clipboard_file.rs   클립보드(텍스트+파일)
│   ├── port_forward.rs / port_forward_mux.rs  포트 포워딩·터널
│   ├── auth_2fa.rs               2단계 인증 (TOTP)
│   ├── privacy_mode.rs           프라이버시 모드
│   ├── virtual_display_manager.rs 가상 모니터
│   ├── kcp_stream.rs             KCP 전송
│   ├── updater.rs                자동 업데이트
│   ├── custom_server.rs          자체 서버 설정
│   ├── lan.rs                    LAN 자동 탐색
│   ├── whiteboard/               화면 위 주석 그리기
│   ├── server/                   기능 서비스
│   │   ├── video_service.rs      화면 전송
│   │   ├── audio_service.rs      소리 전송
│   │   ├── input_service.rs      마우스/키보드 주입
│   │   ├── clipboard_service.rs  복사·붙여넣기
│   │   ├── terminal_service.rs   원격 터미널 (portable-pty)
│   │   ├── printer_service.rs    원격 프린터
│   │   ├── display_service.rs    다중 모니터
│   │   ├── drm_capturer.rs       리눅스 DRM 캡처
│   │   ├── video_qos/            네트워크 품질 자동 조절
│   │   └── wayland.rs, uinput.rs 리눅스 전용
│   ├── platform/                 OS 분기 (windows.rs / linux.rs / macos.mm)
│   ├── privacy_mode/             윈도우 5가지 구현 + macOS
│   ├── lang/                     55개 언어 (ko.rs 포함)
│   └── ui/                       구형 Sciter UI (deprecated)
│
├── flutter/                      현재 UI
│   ├── lib/desktop/pages/        연결·파일관리·원격화면·터미널·카메라·포트포워딩
│   ├── lib/mobile/               Android / iOS
│   ├── lib/models/               상태 관리
│   └── android/ ios/ windows/ macos/ linux/ web/
│
├── libs/
│   ├── scrap/                    화면 캡처 + 코덱
│   │   └── common/: codec.rs vpxcodec.rs aom.rs hwcodec.rs vram.rs camera.rs record.rs
│   │   └── dxgi/(Win) x11/ wayland/(Linux) quartz/(macOS) android/
│   ├── enigo/                    마우스·키보드 자동 조작
│   ├── clipboard/                파일 복사-붙여넣기
│   ├── hbb_common/               서버와 공유 (git 서브모듈)
│   ├── base/                     클라이언트 전용 공통 (옵션키, fs.rs, protobuf)
│   ├── virtual_display/          가상 디스플레이 드라이버
│   ├── remote_printer/           원격 프린터 드라이버
│   └── portable/                 무설치 실행 버전
│
├── res/                          리소스 + Server Pro 관리 API 스크립트
│   ├── ab.py users.py devices.py admin-roles.py
│   ├── device-groups.py user-groups.py audits.py strategies.py
│   ├── msi/ rpm.spec PKGBUILD DEBIAN/   설치 패키지
│   └── 아이콘, 로고, rustdesk.service (systemd)
│
├── .github/workflows/            CI/CD 11개 (전 플랫폼 자동 빌드)
├── docs/                         26개 언어 README + 기여 가이드
├── build.py (57KB)               빌드 오케스트레이션
├── Dockerfile / entrypoint.sh    도커 빌드
├── CLAUDE.md / AGENTS.md         AI 에이전트 작업 규칙
└── Cargo.toml                    의존성 + 기능 플래그
```

### Cargo 워크스페이스 멤버
`libs/scrap`, `libs/hbb_common`, `libs/base`, `libs/enigo`, `libs/clipboard`,
`libs/virtual_display`, `libs/virtual_display/dylib`, `libs/portable`, `libs/remote_printer`

### 주요 기능 플래그 (`Cargo.toml`)
`flutter`, `hwcodec`(GPU 인코딩), `vram`, `mediacodec`(Android),
`drm` / `drm-wake`(리눅스 DRM 캡처·디스플레이 웨이크), `unix-file-copy-paste`,
`screencapturekit`(macOS), `inline`, `linux-pkg-config`

---

## 3. 작동 원리

### 연결 흐름 (`src/rendezvous_mediator.rs`)

```
[A: 내 PC]            [중계서버 hbbs]            [B: 제어할 PC]
   │ (1) ID 등록 ──────────▶│◀────────── (1) ID 등록
   │ (2) "B에 연결" ───────▶│
   │                        │─ (3) "A가 찾는다" ──▶│
   │ (4) NAT 홀펀칭 (punch_hole)
   │◀═══════ P2P 직접 연결 (서버는 빠짐) ═══════▶│
   │ (5) 실패 시에만 relay 서버 경유 (암호화 터널)
```

핵심 함수: `register_peer` → `handle_punch_hole` → `handle_request_relay` → `create_relay`

- **P2P 성공 시 화면 데이터가 서버를 지나가지 않음** → "내 화면 유출 걱정 없음"의 근거
- 최신 버전은 **WebRTC + KCP** 추가 → 불안정한 회선에서도 끊김 감소
  (Cargo.toml의 `[patch.crates-io]` 주석에 webrtc-util IPv6 바이트순서 버그,
  webrtc-sctp의 1s RTO floor 문제와 KCP 혼잡제어 대체 이유가 상세히 기록됨)
- 종단간 암호화는 NaCl 기반 (`hbb_common`)

### 화면 전송 파이프라인

```
캡처(libs/scrap: DXGI/X11/Wayland/Quartz)
  → 인코딩(VP8/VP9/AV1/H264/H265, GPU 가속 시 hwcodec/vram)
  → 전송(video_service.rs)
  → 수신·디코딩(client.rs)
  → 렌더링(remote_page.dart)
  → 네트워크 저하 시 video_qos/가 화질 자동 조절
```

### 입력 주입 파이프라인

```
Flutter가 좌표 캡처 → protobuf 메시지 → 암호화 전송
  → input_service.rs 수신 → libs/enigo
  → Windows: SendInput / Linux: uinput / macOS: CGEvent
```

---

## 4. 기능 전체 목록 (코드 근거)

| 기능 | 근거 |
|---|---|
| 원격 화면 보기/조작 | `video_service.rs`, `input_service.rs` |
| 양방향 소리 | `audio_service.rs`, `audio_resampler.rs` |
| 파일 전송 | `libs/base/src/fs.rs`, `file_manager_page.dart` |
| 클립보드 공유(텍스트+파일) | `clipboard.rs`, `clipboard_file.rs` |
| 원격 터미널 | `server/terminal_service.rs` |
| 원격 프린터 | `server/printer_service.rs`, `libs/remote_printer` |
| 원격 카메라 보기 | `view_camera_page.dart`, `scrap/common/camera.rs` |
| TCP 포트 포워딩/터널 | `port_forward.rs`, `port_forward_mux.rs` |
| 프라이버시 모드 | `privacy_mode/` (Windows 5종 + macOS) |
| 가상 모니터 추가 | `virtual_display_manager.rs`, `libs/virtual_display` |
| 화이트보드 주석 | `src/whiteboard/` |
| 2단계 인증 | `auth_2fa.rs` (TOTP) |
| 다중 모니터 | `display_service.rs` |
| LAN 탐색 / Wake-on-LAN | `lan.rs`, `wol-rs` |
| 세션 녹화 | `scrap/common/record.rs`, `hbbs_http/record_upload.rs` |
| 스크린샷 | `client/screenshot.rs` |
| 자체 서버 지정 | `custom_server.rs` |
| 자동 업데이트 | `updater.rs` |
| SSO / OIDC | `hbbs_http/account.rs` |

---

## 5. 설치 및 사용법

### 방법 A — 그냥 쓰기 (5분, 추천)

1. https://github.com/rustdesk/rustdesk/releases 에서 받기
2. 양쪽 PC에 설치(또는 무설치 실행)
3. 9자리 ID + 1회용 비밀번호 확인
4. 제어 측에서 상대 ID 입력 → 연결

무인 접속: 설정에서 **영구 비밀번호** 지정 + 서비스로 설치(자동 시작).
리눅스는 `res/rustdesk.service`(systemd) 활용.

### 방법 B — 자체 서버 구축 (1~2시간)

서버는 이 저장소가 아니라 `rustdesk/rustdesk-server`.

```bash
docker run --name hbbs -p 21115:21115 -p 21116:21116 -p 21116:21116/udp \
  -p 21118:21118 -v ./data:/root -td --net=host rustdesk/rustdesk-server hbbs

docker run --name hbbr -p 21117:21117 -p 21119:21119 \
  -v ./data:/root -td --net=host rustdesk/rustdesk-server hbbr
```

- 필요 포트: TCP `21115~21119`, UDP `21116`
- 클라이언트 설정에 내 서버 주소 + `data/id_ed25519.pub`의 Key 입력

### 방법 C — 소스 빌드 (1~3시간)

```bash
# 1) 시스템 패키지 (Ubuntu/Debian)
sudo apt install -y zip g++ gcc git curl wget nasm yasm libgtk-3-dev clang \
  libxcb-randr0-dev libxdo-dev libxfixes-dev libxcb-shape0-dev \
  libxcb-xfixes0-dev libasound2-dev libpulse-dev cmake make \
  libclang-dev ninja-build libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev

# 2) vcpkg + 미디어 라이브러리
git clone https://github.com/microsoft/vcpkg
cd vcpkg && git checkout 2023.04.15 && cd ..
vcpkg/bootstrap-vcpkg.sh
export VCPKG_ROOT=$HOME/vcpkg
vcpkg/vcpkg install libvpx libyuv opus aom

# 3) Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# 4) 소스 (서브모듈 필수)
git clone --recurse-submodules https://github.com/bmshin94/rustdesk
cd rustdesk

# 5) 빌드
cargo run                 # Sciter 구버전 UI
python3 build.py --flutter  # Flutter 현행 UI (Flutter SDK 필요)
```

도커 빌드:
```bash
docker build -t "rustdesk-builder" .
docker run --rm -it -v $PWD:/home/user/rustdesk \
  -v rustdesk-git-cache:/home/user/.cargo/git \
  -v rustdesk-registry-cache:/home/user/.cargo/registry \
  -e PUID="$(id -u)" -e PGID="$(id -g)" rustdesk-builder
```

주의: `--recurse-submodules` 누락 시 `libs/hbb_common`이 비어 빌드 실패.
디스크 20GB 이상 필요. Fedora는 libvpx `-fPIC` 패치 필요(README 참조).

---

## 6. 플러그인? 스킬? MCP? → 전부 아님

| 종류 | RustDesk가 이건가 |
|---|---|
| 플러그인 (앱 확장) | 아님 |
| 스킬 (AI 지시문 묶음) | 아님 |
| MCP 서버 (AI 도구 프로토콜) | 아님 |
| **독립 애플리케이션** | **이것** |

RustDesk는 Rust로 작성되어 네이티브 실행 파일로 컴파일되는 **최종 제품**이다.
(이 저장소에 플러그인 프레임워크 코드는 없음 — `grep`으로 확인)

혼동 지점: `CLAUDE.md` / `AGENTS.md`는 **"RustDesk를 AI로 개발할 때의 규칙 문서"**일 뿐,
RustDesk가 AI 도구라는 뜻이 아니다. `GEMINI.md`는 10바이트 사실상 빈 파일.

단, RustDesk는 **MCP로 감쌀 수 있는 좋은 재료**다:
CLI 인자(`core_main.rs`), IPC(`ipc.rs`), REST API 스크립트(`res/*.py`).

---

## 7. API 토큰이 필요한가

| 상황 | 토큰 |
|---|---|
| 그냥 원격 접속 | 불필요 (ID + 비밀번호) |
| 자체 서버(무료 OSS) 운영 | 불필요 (Key만) |
| 주소록 로그인/동기화 | 간접 사용 |
| **Server Pro 관리 자동화** | **필요 (Bearer 토큰)** |
| SSO / OIDC | 내부 토큰 사용 |

### 코드 근거

`res/ab.py`:
```python
def get_personal_ab(url, token):
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(f"{url}/api/ab/personal", headers=headers)
```

`src/hbbs_http/account.rs`에서 사용하는 엔드포인트:
`/api/login-options`, `/api/oidc/auth`, `/api/oidc/auth-query`

### Server Pro 관리 스크립트 (`res/`)

| 스크립트 | 기능 |
|---|---|
| `ab.py` | 주소록 조회/공유 관리 |
| `users.py` | 사용자 생성/조회/삭제 |
| `devices.py` | 기기 목록, 오프라인 기간 필터 |
| `admin-roles.py` | 관리자 role |
| `device-groups.py` / `user-groups.py` | 그룹 관리 |
| `audits.py` | 감사 로그 |
| `strategies.py` | 정책 배포 |

사용 예:
```bash
python3 res/devices.py --url http://내서버:21114 --token <TOKEN> view
```

주의: Server Pro는 유료(20대 이하 무료 티어 있음). 무료 OSS 서버에는
웹 콘솔·주소록·감사로그·API가 없다. 토큰은 절대 코드/깃허브에 하드코딩 금지.

---

## 8. AI 에이전트 구축에 도움이 되는가 → 크게 됨

### 방향 1 — AI에게 "손발" 달아주기 (가장 유망)

현재 AI 에이전트의 한계는 "실제 컴퓨터를 보고 조작하기". RustDesk가 그 인프라다.

```
AI 에이전트 (Claude / GPT / Cursor)
      │ MCP 프로토콜
      ▼
  rustdesk-mcp  ← 직접 만들 레이어
      ├── screenshot()     ← libs/scrap, client/screenshot.rs
      ├── click() / type() ← libs/enigo
      ├── run_command()    ← server/terminal_service.rs
      ├── send_file()      ← libs/base/src/fs.rs
      ├── list_devices()   ← res/devices.py API
      └── audit_log()      ← res/audits.py API
      ▼
  원격 PC (고객사 / 공장 HMI / 레거시 시스템)
```

차별점: **원격**이라는 것. AI가 자기 컨테이너가 아니라 실제 고객 PC나
사내 레거시 시스템을 조작할 수 있다. `res/*.py`가 이미 Python + Bearer 토큰
구조라 MCP 서버 래핑이 쉽다 (MVP 1~2주).

### 방향 2 — AI 코딩 방법론 교재 (`AGENTS.md`)

`AGENTS.md`는 AI 에이전트 운용 규칙의 교과서급 예시다. 핵심 발췌:

- **Be minimally invasive**: "이상적인 수정 diff는 줄을 추가하고 아무것도 수정/삭제하지 않는다"
- **Scope check**: 공유 trait/구조체/시그니처 변경 전, 버그가 한 경로에만 국한된 건지 확인
- "관계없는 호출자가 시그니처 때문에 `Default::default()`나 `None`을 넣어야 한다면
  diff가 너무 넓다 — 멈추고 재설계하라"
- **Mandatory regression-surface check**: 완료 전 변경한 기존 파일·런타임 경로를
  명시적으로 보고하고 각각이 왜 불가피했는지 설명
- `unwrap()`/`expect()` 금지, Tokio 런타임 중첩 금지, 락을 `.await` 넘어 보유 금지
- import는 크레이트당 `use` 하나

### 방향 3 — 에이전트 인프라 패턴 학습

| RustDesk 패턴 | AI 에이전트 적용 |
|---|---|
| `rendezvous_mediator.rs` NAT 홀펀칭 | 분산 에이전트 P2P 통신 |
| `video_qos/` 적응형 품질 조절 | 토큰·비용 예산에 따른 모델 동적 선택 |
| `ipc.rs` 프로세스 간 통신 | 멀티 에이전트 메시지 버스 |
| `src/platform/` OS 추상화 | 멀티 프로바이더 추상화 |
| `auth_2fa.rs` 권한 체계 | 에이전트 액션 승인 게이트 |
| `audits.py` 감사 로그 | 에이전트 행동 추적·롤백 |

주의: "RustDesk에 AI를 넣는" 방향보다 **"AI가 RustDesk를 도구로 쓰는"** 방향이 정답.

---

## 9. React / PHP로 만들 수 있는가

### 핵심 엔진은 불가능

| 필요 작업 | JS/PHP로 안 되는 이유 |
|---|---|
| 초당 60회 화면 캡처 | OS 커널 API 직접 호출 (DXGI/X11/Quartz) |
| 비디오 인코딩 | 네이티브 성능 필수 |
| 마우스/키보드 주입 | SendInput / uinput / CGEvent = OS 권한 |
| NAT 홀펀칭 | Raw UDP 소켓 (브라우저 불가) |
| 시스템 서비스 상주 | 브라우저·웹서버는 OS 서비스가 될 수 없음 |

### 주변 영역은 매우 적합

```
┌──────────────────────────────────────────────┐
│ React 프론트엔드  ← 만들 영역                  │
│ · 관리자 대시보드 (기기 목록, 온라인 상태)       │
│ · 접속 이력 / 감사 로그 시각화                  │
│ · 세션 녹화 플레이어                           │
│ · 사용자·그룹 관리 UI                          │
│ · 원클릭 연결 링크 생성기 (rustdesk://)         │
│ · 라이선스·과금 화면                           │
├──────────────────────────────────────────────┤
│ PHP/Laravel 백엔드  ← 만들 영역                │
│ · Server Pro API 프록시 (Bearer 토큰)          │
│ · 멀티테넌시 (고객사별 분리)                    │
│ · 결제 연동 (토스페이먼츠 / Stripe)             │
│ · 알림 (기기 오프라인 → 슬랙/카톡)              │
├──────────────────────────────────────────────┤
│ RustDesk (수정하지 않음) — Rust                │
│ · 서버: hbbs/hbbr (Docker)                    │
│ · 클라이언트: 공식 바이너리                     │
└──────────────────────────────────────────────┘
```

즉시 가능한 프로젝트:
1. **원클릭 연결 포털** (★, 1주) — `rustdesk://connection/new/{id}?password={temp}` 링크/QR 생성
2. **관리 대시보드** (★★, 2~4주) — `res/*.py`의 API 호출을 PHP로 이식
3. **MSP 멀티테넌트 SaaS** (★★★★, 3~6개월) — 위 둘 + 결제 + 고객사 격리

추천 스택: React + TypeScript + TanStack Query + shadcn/ui / Laravel / PostgreSQL /
Docker Compose / Caddy(자동 HTTPS)

AGPL 관점에서도 **RustDesk 코드를 수정하지 않고 API로만 통신**하면
React/PHP 코드는 전염 대상이 아니다 (법률 자문 권장).

---

## 10. 유튜브 강의 제작 가능성 → 좋은 소재

| 요소 | 평가 |
|---|---|
| 검색 수요 | 높음 ("팀뷰어 무료", "원격 접속 프로그램") |
| 한국어 콘텐츠 경쟁 | 매우 부족 |
| 즉각 효용 | "비용 절감" = 강한 후킹 |
| 시각적 임팩트 | 원격 제어는 보여주기 쉬움 |
| 수익 연결 | 영상 → 구축 의뢰 전환 |

### 시리즈 기획 (3트랙)

**트랙 A — 일반인/실무자 (조회수)**
1. 팀뷰어 연 100만원? 이걸로 0원 만들기 (10분)
2. 설치 5분 — 집에서 회사 PC 쓰기 (12분)
3. 부모님 컴퓨터 원격으로 고쳐드리기 (10분)
4. 스마트폰으로 내 PC 조종 (8분)
5. 파일 전송·원격 프린터·터미널 숨은 기능 5가지 (12분)

**트랙 B — IT 담당자 (수익 전환)**
6. 자체 서버 구축 완전판 — Docker 30분 컷 (25분)
7. 도메인 + HTTPS + Caddy 리버스 프록시 (20분)
8. Server Pro vs 무료 OSS 비교 (15분)
9. 기업 보안 설정: 2FA, IP 화이트리스트, 감사로그 (20분)
10. API 토큰으로 기기 관리 자동화 (18분)
11. 사내 PC 100대 자동 배포 (MSI/GPO) (22분)

**트랙 C — 개발자 (브랜딩)**
12. 25만 줄 Rust 프로젝트 코드 투어 (30분)
13. NAT 홀펀칭 원리 (20분)
14. 소스 빌드 + 자사 로고 리브랜딩 (25분)
15. Rust <-> Flutter FFI 동작 원리 (22분)
16. AI 코딩 에이전트로 오픈소스 기여하기 (25분)
17. **RustDesk를 AI 에이전트의 손발로 만들기 (MCP)** (30분) ← 최고 차별화

### 반드시 지킬 것

1. **면책 고지 필수** — README 최상단 경고를 그대로 노출.
   "반드시 본인 소유 기기 또는 명시적 동의를 받은 기기에만 사용" 자막.
   이게 없으면 "해킹 강의"로 신고 위험.
2. **개인정보 마스킹** — 실제 ID / 비밀번호 / IP / 도메인 블러 처리
3. **라이선스 표기** — AGPL-3.0, 상표는 RustDesk 소유, "비공식 가이드" 명시
4. **버전 명시** — "v1.5.0 기준" (UI가 자주 변경됨)
5. 실패·에러 해결 장면을 남기면 체류시간 상승

제작 팁: OBS + 2화면 분할 녹화 / 썸네일에 숫자·비교 강조 /
설명란에 명령어 전문 + 타임스탬프(SEO) / 영어 자막 추가 시 해외 트래픽

---

## 11. 수익화 아이디어

### 법적 전제 (AGPL-3.0)

| 행위 | 가능 | 조건 |
|---|---|---|
| 사내 사용 | 가능 | 제약 없음 |
| 설치·구축·교육 **서비스료** | 가능 | 코드 배포 안 하면 의무 없음 |
| 수정 없이 배포 | 가능 | 라이선스·저작권 고지 유지 |
| 수정해서 배포 | 주의 | 수정 소스 AGPL 공개 의무 |
| 수정해서 네트워크 서비스 | 주의 | 사용자에게 소스 제공 (AGPL 13조) |
| "RustDesk" 상표로 영업 | 불가 | 상표권 별개 — 자체 브랜드 사용 |
| 클로즈드 소스 판매 | 불가 | — |

**핵심 전략: 코드를 수정하지 않는 사업 모델이 가장 안전.**
서비스·운영·컨설팅·별도 저작물(대시보드 등)로 수익화.
※ 실제 사업 전 반드시 변호사 자문. 위 내용은 법률 조언이 아니다.

### 아이디어 1 — 관리형 호스팅 SaaS (1순위 추천)

문제: RustDesk는 공짜지만 서버 구축·포트 개방·HTTPS·백업·모니터링·장애 대응이 어렵다.

제품: "원격 접속 서버, 대신 돌려드립니다"
- 고객 전용 hbbs/hbbr 인스턴스 (격리)
- 고객 도메인 + 자동 HTTPS
- 관리 대시보드 (React/PHP 자체 제작)
- **서버 주소가 미리 박힌 설치 파일** 제공
- 자동 백업, 24/7 모니터링, 업데이트 대행, 한국어 지원

| 플랜 | 기기 수 | 월 요금 | 타깃 |
|---|---|---|---|
| Starter | 10대 | 2만원 | 소상공인 |
| Business | 50대 | 7만원 | 중소기업 |
| Pro | 200대 | 20만원 | 중견기업 |
| Enterprise | 무제한 | 50만원+ | 온프레미스 + SLA |

원가: VPS 월 1~2만원(고객 10~20개 공유 가능), 도메인·인증서 사실상 0원.
고객 30곳 확보 시 월 매출 150~200만원 / 원가 20만원 미만.
난이도 ★★★ / MVP 2~3개월 / 손익분기 고객 5~10곳.

리스크: 공식이 직접 호스팅 제공 → 차별화는 한국어 지원·국내 데이터센터·세금계산서.
Server Pro 재판매 조건 확인 필수. 무료 OSS 기반이면 기능 제약을
자체 대시보드로 메우는 게 오히려 차별점.

### 아이디어 2 — 기업 전환 컨설팅 (초기 자본 0원, 현금화 최속)

제품: "팀뷰어 라이선스 비용, 1년 안에 회수해 드립니다"

| 패키지 | 내용 | 가격 |
|---|---|---|
| 진단 | 현황 분석 + 전환 계획서 + 절감 리포트 | 50~100만원 |
| 구축 | 서버 구축 + 보안 설정 + 100대 배포 + 교육 | 300~800만원 |
| 유지보수 | 월 점검·업데이트·장애 대응 | 월 30~100만원 |
| 리브랜딩 | 자사 로고·이름 커스텀 빌드 | +200~500만원 |

영업 논리:
1. ROI가 숫자로 나옴 ("연 800만원 → 구축비 500만원, 2년차부터 연 800만원 절감")
2. 보안 규제 대응 — 의료법, 전자금융감독규정, 망분리 → "화면이 해외 서버로 안 갑니다"
3. 기술 장벽 — 담당자는 "할 수는 있는데 시간이 없다"

타깃 우선순위: 병원·의원 → 제조업(공장 설비 PC) → IT 유지보수 업체(MSP, B2B2B)
→ 학교·학원 → 법무·회계

난이도 ★★(기술) / ★★★★(영업) / 첫 계약 1~3개월.
**핵심은 레퍼런스 1건** — 첫 고객을 반값에라도 확보해 사례 확보.

### 아이디어 3 — RustDesk x AI 에이전트 (블루오션)

**제품 A: RustDesk MCP 서버**
- 기본: 오픈소스 무료 → 인지도·커뮤니티
- 클라우드 버전: 인증·승인 게이트·감사로그·멀티테넌시 포함, 월 5~30만원
- 기업 커스텀 구축: 단건 500~2000만원

**제품 B: AI 자동 원격 지원 봇**
```
고객 "프린터가 안 돼요"
  → AI가 접속 → 화면 캡처 → 진단
  → 간단: 자동 해결 / 복잡: 엔지니어에게 요약 + 세션 인계
```
헬프데스크 티켓의 상당 부분이 반복 작업 → 1차 대응 자동화로 인건비 직접 절감.

**제품 C: 레거시 시스템 자동화 (가장 돈 되는 니치)**
20년 된 ERP, 공장 HMI, 정부 시스템에는 API가 없다 → AI Vision + RustDesk =
"화면 기반 RPA, 그런데 원격으로". 기존 RPA보다 유리: 원격 적용 범위,
API 없는 시스템 대응, 낮은 원가.
타깃: 제조 MES/HMI, 물류, 공공 레거시 / 구축 1000~5000만원 + 유지보수.

난이도 ★★★★ / MCP MVP 1~2주 / 상용 6~12개월.
리스크: 보안·법적 리스크 최상급 → 명시적 동의, 승인 게이트, 전체 감사로그,
세션 녹화 필수. AGPL: RustDesk를 별도 프로세스로 호출(링크 금지).

### 아이디어 4~10 (요약)

| # | 아이디어 | 난이도 | 수익 | 기간 |
|---|---|---|---|---|
| 4 | 교육 콘텐츠 (유튜브 + 유료 강의 + 전자책) | ★★ | 월 50~300만원 | 2~4개월 |
| 5 | 리브랜딩 제품화 (AGPL 소스 공개 필수) | ★★★★ | 라이선스 | 4~8개월 |
| 6 | 업종 특화 패키지 (병원 전용 등) | ★★★ | 건당 500~2000만원 | 3~6개월 |
| 7 | React/PHP 대시보드 단독 판매 (별도 저작물) | ★★★ | 건당·구독 | 2~4개월 |
| 8 | 배포 자동화 툴 (MSI/GPO/Intune/Ansible) | ★★ | 번들 | 1~2개월 |
| 9 | 오픈소스 기여 → 전문가 포지셔닝 (`ko.rs` 번역 완성) | ★ | 간접 | 상시 |
| 10 | MSP 파트너 네트워크 (화이트라벨) | ★★★★ | 레버리지 최대 | 6~12개월 |

### 실행 로드맵

```
[0~1개월] 기반 — 비용 0원
  · 로컬 소스 빌드 성공
  · Docker 자체 서버 구축 + 실사용
  · src/lang/ko.rs 번역 기여 → PR 머지 (레퍼런스)
  · 유튜브 트랙 A 1~3편

[1~3개월] 첫 현금 — 아이디어 2
  · 지인/소기업 1곳 반값 구축 → 사례 확보
  · React/PHP 간단 대시보드 제작
  · 유튜브 트랙 B → 설명란 "구축 문의"
  · 목표: 유료 계약 1건

[3~6개월] 반복 수익 — 아이디어 1
  · 관리형 호스팅 MVP (Docker + Laravel + React + 결제)
  · 베타 고객 5곳 / 목표 MRR 100만원

[6~12개월] 차별화 — 아이디어 3
  · rustdesk-mcp 오픈소스 공개
  · 유튜브 17편 "AI 에이전트의 손발"
  · 기업 커스텀 1건 수주
```

### 리스크 총정리

| 리스크 | 완화 |
|---|---|
| AGPL 위반 | 코드 미수정 원칙, 수정 시 공개, 변호사 자문 |
| 상표권 | "RustDesk" 상호 사용 금지, 자체 브랜드 |
| 공식과 경쟁 | 한국어·로컬 지원·세금계산서·업종 특화 |
| 오용 책임 | 계약서 용도 제한, 로그·동의 확보 |
| 업스트림 급변 | 버전 고정(pin), 업데이트 테스트 절차 |
| 인력 부족 | 아이디어 2로 현금 확보 후 확장 |

**추천 조합: 아이디어 2(현금) → 1(반복수익) → 3(미래) + 4(마케팅 채널)**
유튜브(4번)는 1·2·3 전부의 영업 채널이므로 가장 먼저 시작.

---

## 12. 당신에게 무슨 도움이 되는가 (정리)

### 실질적 이득
1. **비용 절감** — 팀뷰어 기업용 연 수십~수백만원 → 0원
2. **사업화 가능** — AGPL 범위 내에서 구축·운영 대행은 합법
3. **최고급 학습 교재** — 크로스플랫폼 추상화, Tokio 비동기, NAT 홀펀칭,
   Rust↔Flutter FFI, 하드웨어 코덱 연동
4. **포크했으니 개조 자유** — 리브랜딩, 서버 하드코딩, 기능 추가/제거
5. **AI 작업 환경 완비** — `CLAUDE.md` + `AGENTS.md`로 AI 에이전트가 규칙대로 작업

### 솔직한 제약
- 빌드 난이도 높음 (vcpkg + Flutter SDK + 플랫폼 툴체인, 20GB+)
- Rust 모르면 코드 수정 사실상 불가
- AGPL-3.0 — 수정 후 네트워크 서비스 시 소스 공개 의무
- 이 저장소는 **클라이언트만** — 서버는 `rustdesk/rustdesk-server` 별도

### 난이도 3단계
```
레이어 1 (★)     그냥 쓴다 — 공짜 팀뷰어. 오늘 당장 가능
레이어 2 (★★★)   내 서버 + 브랜딩 — 여기서 돈이 나온다
레이어 3 (★★★★★) 코드 개조 — Rust/Flutter 필요, 학습 가치 최상
```

---

## 면책

- RustDesk 공식 경고: 개발팀은 비윤리적·불법적 사용을 지지하지 않으며,
  무단 접근·제어·사생활 침해는 가이드라인 위반이다. 오용에 대한 책임은 사용자에게 있다.
- **반드시 본인 소유 기기 또는 명시적 동의를 받은 기기에만 사용할 것.**
- 본 문서는 법률 조언이 아니다. AGPL-3.0 해석과 사업 모델은 변호사 자문을 받을 것.
- 수익 추정치는 시장 일반론에 기반한 예시이며 보장된 수치가 아니다.
- 이 문서는 비공식 분석 자료이며 RustDesk 프로젝트의 공식 문서가 아니다.
