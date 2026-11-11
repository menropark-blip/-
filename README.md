# 코이콘 KOICON 스토어

Montage의 이모티콘·캐릭터 굿즈 브랜드 홈페이지입니다. HTML, CSS, JavaScript로 작성했고 빌드가 필요 없습니다.

## 실행

이 폴더에서 `npm run dev`를 실행한 다음 http://127.0.0.1:4173 을 엽니다. `dist`가 배포할 웹 파일입니다. ES 모듈을 사용하므로 HTML 더블 클릭 대신 HTTP 서버를 사용하세요.

## 구현 범위

- 반응형 홈페이지, 전체 너비 메인 3개 슬라이드, 로고 호버 인터랙션과 캐릭터 움직임
- 브랜드 소개 → 굿즈 스토어 → 브랜드 제작 → 강연·교육 → 연락처
- 5개 대메뉴와 35개 세부 메뉴의 안내·문의 화면, 전체 사이트맵
- 제공된 굿즈 이미지 기반 5개 상품 미리보기, 카테고리 필터, 가로 슬라이드, 찜, 장바구니 수량·삭제·보관
- 이메일 문의 작성, 로그인·회원가입·비밀번호 재설정·내 주문 화면
- 관리자 상품·가격·재고·공개 상태 편집, 주문 상태 관리, 교육·협업 문의와 개인정보 삭제, 소개 콘텐츠 편집
- Firebase Auth / Firestore / callable Cloud Functions 연동 코드, 접근 제어 규칙, 공식 Firebase MCP 구성

상품명은 디자인을 위한 제안이며 가격·실물 사양·출시 일정은 미확정입니다. 후기·고객사·수상 이력 등 제공되지 않은 사실은 만들어 넣지 않았습니다.

## 현재 연결 상태

Firebase 프로젝트·인증 권한이 제공되지 않아 `dist/config.js`의 `firebase`는 `null`입니다. 로그인·회원가입은 연결 준비 상태로 표시합니다. 장바구니·찜은 이 브라우저에 저장됩니다. 관리자 미리보기 수정은 현재 페이지 메모리에만 반영되며 새로고침하면 초기화됩니다. 문의는 이메일 앱을 열 뿐 자동 발송하지 않습니다. 사용자가 이메일 앱에서 전송을 완료해야 합니다. 실제 결제 연동은 없으며 카드·계좌 정보를 수집하지 않습니다.

## Firebase 실제 연결 순서

1. Firebase 공식 플러그인을 연결하거나 CLI에서 `npx firebase-tools@latest login`으로 본인의 Google 계정을 인증합니다.
2. 전용 Firebase 프로젝트와 웹 앱을 만들고 Firestore 데이터베이스와 이메일/비밀번호 Authentication을 활성화합니다. Cloud Functions 배포에는 해당 프로젝트의 결제 요건을 확인해야 합니다.
3. 콘솔의 웹 앱 구성 객체를 `dist/config.js`의 `firebase: null` 대신 넣습니다. 이 공개 웹 설정에 Admin SDK 비밀키나 서비스 계정 JSON을 넣지 마세요.
4. `npx firebase-tools@latest use --add`로 이 폴더에 프로젝트를 선택합니다. 인증 허용 도메인에 실제 사이트 도메인을 추가합니다.
5. `firebase/functions`에서 `npm install`을 실행합니다. 루트에서 `npx firebase-tools@latest deploy --only firestore:rules,functions`를 실행합니다. 사이트 자체는 `dist`를 정적 호스팅하거나 Firebase Hosting으로 배포할 수 있습니다.
6. 관리자 계정을 가입한 후 신뢰할 수 있는 Admin SDK/MCP에서 해당 UID에 `admin: true` 커스텀 클레임을 부여합니다. 화면이나 이메일 주소만으로 관리자가 되지 않습니다. 재로그인 후 권한이 반영됩니다.
7. 관리자페이지에서 확정된 상품명·가격·재고·상품별 사진을 등록하고 판매 준비 체크를 해제합니다. 데이터베이스가 비어 있으면 시안 상품이 자동으로 실상품으로 판매되지 않습니다.
8. 테스트 계정으로 가입, 로그인, 문의, 주문 요청, 재고 감소, 취소 시 재고 복구, 타인 주문 접근 차단을 검증한 후 판매를 엽니다. 현재 자동 검증은 주문 순수 로직 대상이며 실제 Firebase·규칙 에뮬레이터 통합 검증은 미실행입니다.

주문은 서버가 DB의 상품 가격을 조회해 합계를 계산하고 트랜잭션에서 재고를 차감합니다. 결제 상태는 항상 `unpaid`로 시작합니다. 고객의 직접 주문 쓰기는 보안 규칙에서 금지합니다. 문의와 주문 읽기는 작성자·관리자에게만 허용합니다. 주문 취소는 한 번만 재고를 복구합니다. 배송 완료 주문은 현재 취소 처리할 수 없습니다. 온라인 결제, 반품/환불, 세금계산, 자동 배송비 계산은 별도 연동 범위입니다.

관리자 주문 탭의 합계는 **주문 요청 금액**입니다. 실제 결제 매출을 의미하지 않습니다. 개인정보 처리 안내의 보관 기준은 운영자가 실제로 이행해야 하며 자동 삭제 작업은 포함되어 있지 않습니다.

## Firebase MCP

공식 서버 실행 설정을 `.mcp.json`과 프로젝트용 `.codex/config.toml`에 준비했습니다. 명령은 `npx -y firebase-tools@latest mcp --only auth,firestore`입니다. 클라이언트가 프로젝트 구성을 신뢰·로드하고 Google 인증을 완료해야 도구가 연결됩니다. 구성 파일만으로 원격 프로젝트가 생성되거나 현재 Codex 세션에 연결되는 것은 아닙니다.

- https://firebase.google.com/docs/ai-assistance/mcp-server
- https://firebase.google.com/docs/web/setup
- https://firebase.google.com/docs/firestore/security/rules-conditions

## 검증

`npm run check`: JavaScript 문법 검사와 주문 검증 테스트 5개. 브라우저에서 PC/모바일 레이아웃, 메뉴, 상품·장바구니 흐름을 확인합니다.

## 주요 파일

`dist/index.html` 구조 / `dist/style.css` 디자인 / `dist/app.js` 화면 동작 / `dist/firebase-service.js` Firebase 어댑터 / `dist/config.js` 웹 프로젝트 구성 / `firebase/functions` 신뢰할 수 있는 주문 처리 / `firebase/firestore.rules` 데이터 접근 제어.

연락처: 작가 Montage / 010-4171-1230 / jhkim7@nate.com / 화성시 동탄.
