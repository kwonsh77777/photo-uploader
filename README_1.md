# 📷 구글 드라이브 사진 업로더 (GitHub Pages + Apps Script)

- **화면**: GitHub Pages (PWA — 휴대폰 홈 화면에 설치 가능)
- **업로드 처리**: Google Apps Script 웹 앱 → 드라이브 폴더에 `YYYYMM월` 폴더 자동 생성
- **기능**: 일자별 갤러리 · 체크 후 끌어내려 일괄 선택 · 올리기(휴대폰 임시 저장 → Wi‑Fi 연결 시 업로드) · 중복 사진 건너뜀

## 저장소 파일 구성 (GitHub에 올릴 파일)

```
index.html
manifest.webmanifest
sw.js
icon-192.png
icon-512.png
icon-maskable-512.png
apple-touch-icon.png
README.md
```
(아이콘 PNG 4개가 홈 화면 아이콘이 됩니다. 기존 `icon.svg`는 더 이상 쓰지 않으므로 저장소에서 삭제해도 됩니다.)
※ `Code.gs`는 GitHub이 아니라 **Apps Script**에 붙여넣는 파일입니다. (보관용으로 저장소에 같이 올려도 됩니다.)

## 1단계. Apps Script 백엔드 배포
1. https://script.google.com 에서 새 프로젝트 생성
2. `Code.gs` 내용 전체를 붙여넣고, 맨 위 `SECRET_TOKEN` 을 **나만 아는 긴 문자열**로 변경
3. 프로젝트 설정 → "appsscript.json 매니페스트 표시" 체크 → 아래 내용으로 교체
   ```json
   {
     "timeZone": "Asia/Seoul",
     "dependencies": {},
     "exceptionLogging": "STACKDRIVER",
     "runtimeVersion": "V8",
     "oauthScopes": ["https://www.googleapis.com/auth/drive"],
     "webapp": { "executeAs": "USER_DEPLOYING", "access": "ANYONE" }
   }
   ```
4. 함수 목록에서 `authorize` 실행 → 드라이브 권한 승인 (최초 1회)
5. 배포 → 새 배포 → 유형 **웹 앱**
   - 다음 사용자로 실행: **나**
   - 액세스 권한: **모든 사용자**
6. 나온 **웹 앱 URL(…/exec)** 복사

> `Code.gs`를 수정한 뒤에는 배포 → 배포 관리 → 편집 → **새 버전**으로 다시 배포해야 반영됩니다.

## 2단계. GitHub Pages 배포
1. GitHub에서 새 저장소 생성 (예: `photo-uploader`, Public)
2. 위 5개 파일을 저장소 루트에 업로드 (Add file → Upload files)
3. Settings → Pages → Source: **Deploy from a branch** → Branch: `main` / `/ (root)` → Save
4. 1~2분 후 `https://<내아이디>.github.io/photo-uploader/` 접속 가능

## 3단계. 휴대폰에서 사용
1. 위 주소를 휴대폰 크롬(또는 사파리)으로 열기
2. 처음 뜨는 **⚙ 설정**에 `웹 앱 URL`과 `SECRET_TOKEN` 입력 → **연결 테스트** → 저장
   (이 값은 그 휴대폰 브라우저에만 저장되며 GitHub 저장소에는 들어가지 않습니다)
3. 메뉴 → **홈 화면에 추가**(설치)하면 앱처럼 사용 가능
4. `+ 사진 불러오기` → 일자별로 확인 → 체크(끌어내려 일괄 선택) → `올리기`

## 동작 규칙
- 폴더명: 사진 파일의 수정(촬영)일 기준 `YYYYMM월` (예: `202611월`)
- 중복: 같은 폴더에 **이름과 용량이 같은 파일**이 있으면 건너뜀. 이름만 같고 내용이 다르면 `_1`을 붙여 저장
- Wi‑Fi: 안드로이드 크롬은 자동 판별. 아이폰/PC는 판별 불가 → 하단 "확인 불가 시 업로드 허용" 옵션 사용
- 업로드는 앱 화면이 열려 있는 동안 진행됩니다. (임시 저장된 사진은 앱을 닫았다 열어도 남아 있고, 다시 열면 이어서 올립니다)
- 한 장당 40MB 초과 시 오류 처리

## 보안 주의
- 웹 앱을 "모든 사용자"로 배포하므로 **URL과 토큰을 외부에 공개하지 마세요.** 토큰이 틀리면 서버가 요청을 거부합니다.
- 토큰이 노출되면 `Code.gs`의 `SECRET_TOKEN`을 바꾸고 새 버전으로 재배포한 뒤, 앱 ⚙ 설정에서 새 토큰을 입력하세요.
