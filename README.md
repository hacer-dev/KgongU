# 📑 K-공유: 사내 문서 공유 드라이브

## 1. 소개
- 사내 문서 공유 서버 → 클라이언트에서 탐색기와 동일하게 **네트워크 드라이브** 접근 가능
- 현재 WebDAV 로 구성됨

---

## 2. 전체 아키텍처

```
[Server (Wails: Go Backend + Vue Frontend)] ── WebDAV/HTTPS ── [Client (Wails: Go Backend + Vue Frontend)]
   - GUI: 공유폴더/사용자 관리             - GUI: 로그인/연결 관리
   - Go Backend: WebDAV 서버 실행         - Go Backend: WebDAV 클라이언트 연동
   - 사용자 인증                       - 파일 API (WebDAV 프로토콜)
```

## 3. 서버 설계 (Wails: Go Backend + Vue Frontend)

![Image](https://github.com/user-attachments/assets/18687797-2fac-4282-9752-c4f57083987f)

### 3.1 기능
- **GUI (Vue)**:  
  - 공유폴더 루트 지정 및 관리 (예: `D:\CompanyShare`)
  - 사용자 계정 생성, 수정, 삭제 (ID/PW 관리)
  - 서버 상태 모니터링  
- **Go Backend**:  
  - **WebDAV 프로토콜 기반 파일 서비스 실행**  
  - 사용자 인증 처리 (GUI에서 생성된 ID/PW 사용)
  - 파일 시스템 접근 및 관리  

### 3.2 WebDAV API (프로토콜)
- `PROPFIND`: 디렉토리 목록 및 파일 속성 조회  
- `GET`: 파일 다운로드  
- `PUT`: 파일 업로드  
- `DELETE`: 파일 삭제  
- `MKCOL`: 디렉토리 생성  
- `MOVE`, `COPY`: 파일/디렉토리 이동 및 복사  

---

## 4. 클라이언트 설계 (Wails: Go Backend + Vue Frontend)

![Image](https://github.com/user-attachments/assets/6ca3d6a2-5d8c-4633-9443-4941a9acc2f7)

### 4.1 구성
- **Go (Wails 백엔드)**  
  - Wails 런타임을 통해 Vue 프론트엔드와 통신  
  - **WebDAV 클라이언트 라이브러리 연동**:  
    - 사용자 인증 (로그인)
    - WebDAV 서버 마운트/연결 관리  
    - 파일 탐색기에서 네트워크 드라이브로 연결하는 기능 제공 (예: `net use` 명령어 활용)  
  - 파일 스트리밍 및 캐싱 로직 구현 (필요시)  

- **Vue (Wails 프론트엔드)**  
  - GUI: 서버 IP/ID/PW 로그인, 연결 상태, 드라이브 연결/해제 버튼 제공  
  - Wails Go 백엔드와 IPC로 통신하여 WebDAV 연결 제어 및 상태 표시  

### 4.2 동작 흐름
1. 클라이언트 로그인 (Vue UI → Wails Go 백엔드)  
2. Wails Go 백엔드에서 WebDAV 서버에 연결 및 네트워크 드라이브 매핑 (예: `net use K: http://webdav.server.com/share /user:user /password:pass`)
3. 탐색기에서 `K:\` (또는 지정된 드라이브 문자) 열면 WebDAV 서버 파일 실시간 스트리밍  

---

## 5. 설치/실행

### 5.1 요구사항
- Windows 10/11  

### 5.2 설치
- 단일 실행파일로 되어 있음 (따로 설치할 필요없음)


---

## 6. 개발 현황

### 현재 진행 상황 (2025-09-25 기준)
- **서버**: 핵심 기능 구현 완료.
  - 사용자 구성 디렉토리(`%APPDATA%`)에 `settings.json` 저장.
  - Bcrypt를 사용한 비밀번호 해시 저장 및 Basic 인증 구현.
  - HTTPS 기반의 WebDAV 서비스 제공
  - 프로그램 실행 시 Windows 방화벽 규칙 자동 추가/삭제
  - 시작 시 SSL 인증서(`server.crt`, `.key`) 자동 생성 (IP 기반).
  - 클라이언트의 인증서 설치를 돕기 위한 인증서 제공 엔드포인트(`/cert`) 구현
- **클라이언트**: 핵심 기능 구현 완료.
  - `net use` 명령어를 활용하여 네트워크 드라이브를 연결/해제하는 기능 구현.
  - WebDAV 서버를 지정된 드라이브 문자로 연결/해제 (`net use`)
  - 마지막 접속 정보(서버 주소, ID 등)를 사용자 구성 디렉토리(`%APPDATA%`)에 `connection_settings.json` 저장.
  - '원클릭'으로 서버의 자체 서명 인증서를 PC에 자동으로 설치하는 기능 구현
  - 프로그램 정상 종료 시(창 닫기, Ctrl+C) 마운트된 드라이브 자동 연결 해제
  - 작업관리자에서 프로그램 제거시 드라이브 연결 해지되지 않음

