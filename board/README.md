# 아리아 소셜보드 (woo/board)

인스타그램·스레드 소통 현황 대시보드의 **홈 화면 설치형 웹앱(PWA)** 입니다.
정적 파일만 있고 서버·DB·추적 스크립트는 없습니다.

- 앱 주소: https://choungyujin.github.io/woo/board/
- **잠금**: 앱을 열면 6자리 PIN 입력 화면이 먼저 뜹니다. PIN을 넣으면 브라우저에서
  `data.enc.json` 을 복호화해 대시보드를 그립니다. 한 번 풀면 “이 기기에서 잠금 기억하기”로
  다음부터 바로 열립니다(상단 **🔒 잠금** 버튼으로 즉시 다시 잠글 수 있음).

## 파일

| 파일 | 역할 |
| --- | --- |
| `index.html` | 잠금 화면 + 대시보드 (평문 데이터 없음) |
| `data.enc.json` | **암호문**. `v / kdf / iter / salt / iv / ct` — PIN 없이는 열 수 없음 |
| `manifest.webmanifest` | 앱 이름 `아리아 소셜보드`, `display: standalone`, 아이콘 |
| `sw.js` | 앱 껍데기만 캐시하는 서비스워커 (암호문은 캐시하지 않음) |
| `icons/` | 192·512·maskable·apple-touch 아이콘 |
| `robots.txt`, `<meta name="robots" content="noindex, nofollow">` | 검색엔진 노출 차단 |

## 암호화 방식

```
KDF   PBKDF2-SHA256, 600,000회, 32바이트  (입력 = PIN 문자열 UTF-8)
암호   AES-256-GCM, IV 12바이트(매번 랜덤), AAD 없음
본문   UTF-8 JSON
파일   { "v":1, "alg":"AES-GCM", "kdf":"PBKDF2-SHA256", "iter":600000,
        "salt":"base64 16B", "iv":"base64 12B", "ct":"base64", "updated":"..." }
```

`salt` 는 고정값입니다(기억된 기기 키를 유지하기 위해). `iv` 는 암호화할 때마다 새로 만듭니다.
PIN을 바꾸면 기존에 기억된 기기 키는 무효가 되고, 각 기기에서 PIN을 한 번 다시 입력하면 됩니다.

## 데이터 갱신

1. 평문 JSON(예: 작업 폴더의 `data.plain.json`)을 최신 내용으로 고칩니다.
2. `python3 encrypt.py` 로 같은 PIN·salt·iter 로 다시 암호화해 `data.enc.json` 을 만듭니다.
3. 저장소 `main` 에 `data.enc.json` 을 커밋·푸시합니다.
4. 앱을 열거나 **새로고침**을 누르면 `?v=타임스탬프` 로 새 암호문을 받아 복호화합니다.

`accounts` 상태값: `m` = 맞팔, `o` = 상대가 나를 안 따라옴, `i` = 내가 상대를 안 따라감.

## 주의

- 이 저장소는 공개 저장소지만 **평문 개인정보는 없습니다.** 암호문만 있고, 복호화는 PIN을 아는
  기기에서만 됩니다. `noindex` 로 검색 노출도 막습니다.
- 비밀번호·로그인 정보·인스타그램 원본 내보내기(zip)·개인 서명키는 저장하지 않습니다.
