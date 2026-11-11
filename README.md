# 코이콘 KOICON 스토어

Montage의 캐릭터 굿즈 중심 반응형 홈페이지입니다. HTML·CSS·JavaScript로 구현했으며 빌드 없이 `dist`를 배포합니다.

## 현재 운영 상태

- Firebase 프로젝트: `koicon-montage-2026`
- 웹 앱: `1:296796279867:web:3f34eac76b44ac1515381e`
- Firestore: `(default)`, Standard, 서울 `asia-northeast3`, 무료 티어
- 이메일/비밀번호 회원가입·로그인·비밀번호 재설정 활성화
- 시안 상품 5개를 실제 Firestore에 등록. 아직 상품 사양·가격·재고는 미확정이므로 판매 준비 상태입니다.
- 로그인한 회원의 구매·협업·강연 문의는 Firestore에 저장됩니다. 마이페이지에서 본인 문의를 확인할 수 있습니다.
- 비회원 문의는 작가 이메일을 이용합니다. 이메일 앱에서 전송을 완료해야 합니다.
- 사용자 선택에 따라 무료 플랜의 **구매 문의 방식**으로 운영합니다. 장바구니의 상품·수량을 문의에 담습니다. 온라인 결제·서버 주문·자동 재고 차감은 활성화하지 않았습니다.
- 장바구니·찜은 현재 브라우저에 저장됩니다.
- 관리자 상품·재고·공개 상태 편집, 구매 문의, 교육·협업 문의, 소개글 편집은 실제 Firestore를 사용합니다.
- 후기·고객사·수상 이력 등 제공되지 않은 사실은 생성하지 않았습니다.

## 관리자 최초 로그인

Firebase를 인증한 소유자 Google 계정의 이메일로 관리자 계정을 생성했습니다. 관리자 UID는 `montage-owner-2026`, 관리자 권한은 신뢰할 수 있는 Firebase MCP에서 부여한 `admin: true` 커스텀 클레임입니다. 임의로 비밀번호를 설정하거나 저장하지 않았습니다.

홈페이지 로그인에서 소유자 이메일을 입력하고 **비밀번호 재설정**을 눌러, 본인의 메일함에서 비밀번호를 직접 설정한 다음 로그인하세요. 로그인 후 페이지 최하단의 **관리자페이지**로 들어갑니다. 일반 회원은 관리 기능을 사용할 수 없습니다. 작가의 공개 문의 이메일과 Firebase 소유자 계정 이메일은 서로 다를 수 있습니다.

## 로컬 실행

프로젝트 폴더에서 `npm run dev`를 실행한 뒤 http://127.0.0.1:4173 을 엽니다. HTML 더블 클릭은 모듈 실행을 지원하지 않습니다. `dist/config.js`에는 공개용 Firebase 웹 앱 설정만 들어 있으며 서비스 계정 키·OAuth 토큰·관리자 비밀번호는 넣지 않습니다.

## 구성

`dist/index.html`: 5개 대메뉴와 35개 세부 메뉴, 전체 너비 메인 슬라이드, 브랜드→굿즈→제작→강연→연락처.

`dist/style.css`: 빨강·네이비·크림색의 반응형 디자인, 움직이는 로고와 캐릭터, 모션 감소 설정.

`dist/app.js`: 상품 슬라이드·필터·상세·찜·장바구니·계정·문의·관리자 화면.

`dist/firebase-service.js`: Firebase Auth·Firestore 연결, 소유자 문의 조회, 관리자 데이터 관리. Cloud Functions 어댑터는 향후 선택적으로 사용할 수 있지만 현재 `ordersEnabled: false`입니다.

`firebase/firestore.rules`: 공개 상품과 소개글, 작성자·관리자 전용 문의/주문, 엄격한 필드·자료형·길이 검사. 클라이언트의 주문 직접 작성과 역할 상승을 차단합니다.

`firebase/catalogue.json`: 제공된 사진에 맞춘 상품 미리보기 데이터. 재실행하면 같은 ID가 이미 있어 실패하므로 최초 등록용입니다.

`firebase/functions`: 향후 Blaze 플랜을 선택할 경우에만 배포할 서버 주문 처리. DB 가격 기반 금액 계산·트랜잭션 재고 차감·주문 취소 재고 복원·상태 전환 검증을 구현했습니다. 현재 원격 배포하지 않았습니다.

## Firebase MCP

공식 Firebase MCP 설정은 `.mcp.json`과 `.codex/config.toml`에 있습니다. `.firebaserc`는 코이콘 전용 프로젝트를 선택합니다. 공식 MCP 서버를 실제 실행해 웹 앱 등록·SDK 설정 조회·상품 등록·관리자 클레임 설정·규칙 검증을 수행했습니다.

```text
npx -y firebase-tools@latest mcp --only auth,firestore
```

`scripts/firebase-mcp.mjs`는 Windows의 공식 npx 런타임을 사용하는 MCP 호출 도우미입니다. 자체 원격 API나 비밀 서비스 키를 사용하지 않습니다. 다른 운영체제에서는 npx 실행 경로를 조정하세요.

## 검증과 배포

- `npm run check`: JavaScript 문법과 주문 로직 단위 테스트 5개.
- `node scripts/verify-live-firebase.mjs`: 두 일회성 계정으로 실제 회원가입·로그인·문의 저장·본인 조회와 권한 차단 검증. 테스트 계정은 종료 시 삭제합니다.
- 실제 테스트 문의의 삭제 명령 목록은 무시되는 `.sites-runtime/qa-cleanup.json`에 생성됩니다. `node scripts/firebase-mcp.mjs --batch .sites-runtime/qa-cleanup.json`으로 테스트 문의를 정리합니다.
- 보안 규칙은 Firebase MCP 검증 및 실제 배포 시 문법 컴파일 검사를 통과했습니다.
- 규칙 변경 배포: `npx -y firebase-tools@latest deploy --only firestore --project koicon-montage-2026`.
- 인증 설정 변경 배포: `npx -y firebase-tools@latest deploy --only auth --project koicon-montage-2026`.
- Sites의 기존 비공개 사이트는 `.openai/hosting.json`의 프로젝트 ID를 유지해 업데이트합니다. Firebase Hosting을 추가로 배포하지 않았습니다.

규칙은 초기 보안 구성입니다. 공개 범위를 넓히기 전 실제 업무 권한과 함께 추가 검토하세요. 관리자 화면의 상담 종료·개인정보 삭제는 직접 수행하는 운영 기능이며, 문의 종료 후 30일 이내 삭제 기준을 자동으로 실행하는 스케줄러는 포함하지 않습니다. 사이트의 개인정보 안내에 맞게 운영자가 이행해야 합니다.

공식 참고: https://firebase.google.com/docs/ai-assistance/mcp-server / https://firebase.google.com/docs/web/setup / https://firebase.google.com/docs/firestore/security/rules-conditions

연락처: 작가 Montage / 010-4171-1230 / jhkim7@nate.com / 화성시 동탄.
