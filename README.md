# marketing-automation-releases

**Marketing Automation Desktop** 의 distribution channel. 빌드된 desktop 설치
파일 (.dmg / setup.exe) 과 Tauri 자동 업데이트 manifest (`latest.json`) 를
호스팅한다.

[**최신 릴리스 보기**](https://github.com/iLab-Research/marketing-automation-releases/releases/latest)

---

## 1. 설치

### macOS (Apple Silicon)

1. 최신 릴리스에서 **`Marketing.Automation_<version>_aarch64.dmg`** 다운로드.
2. .dmg 더블클릭 → Finder 에 `Marketing Automation` 볼륨이 마운트.
3. `Marketing Automation.app` 을 `Applications` 폴더로 드래그.
4. .dmg 볼륨 eject (Finder 사이드바 ⏏ 또는 `Cmd+E`).
5. **첫 실행 시 Gatekeeper "손상된 앱입니다 — 휴지통으로 옮기세요" 경고**
   가 뜸 (코드사이닝/notarization 미적용 단계의 정상 동작). 다음 명령으로
   quarantine 속성 한 번 해제:

   ```bash
   sudo xattr -dr com.apple.quarantine "/Applications/Marketing Automation.app"
   ```

6. 정상 실행:

   ```bash
   open "/Applications/Marketing Automation.app"
   ```

   (또는 Launchpad / Spotlight 검색)

이후 Tauri 자동 업데이트가 새 .app 을 받을 때 부모 프로세스의 quarantine
state 를 상속하므로 위 명령을 다시 실행할 필요 없음.

### Windows (x64)

1. 최신 릴리스에서 **`Marketing.Automation_<version>_x64-setup.exe`** 다운로드.
2. 더블클릭 → NSIS installer 가 안내. 기본값으로 설치.
3. 첫 실행 시 SmartScreen 경고 (`Windows Defender SmartScreen prevented an
   unrecognized app from starting`) 가 뜨면 **"추가 정보"** → **"실행"** 클릭.
4. 시작 메뉴 / Desktop 아이콘으로 실행.

이후 자동 업데이트는 사용자 동의 후 `passive` 모드 (UI 표시 + 자동 진행)
로 NSIS installer 가 in-place 갱신.

---

## 2. 자동 업데이트 (Tauri Updater)

앱이 시작될 때 백그라운드로 다음 URL 을 체크:

```
https://github.com/iLab-Research/marketing-automation-releases/releases/latest/download/latest.json
```

이 manifest 가 현재 설치된 버전보다 높은 SemVer 를 가리키면, 운영체제별 자산
을 다운로드해서:

- **Windows**: NSIS installer 를 background 에서 실행, 앱 재시작 후 새 버전.
- **macOS**: `.app.tar.gz` 를 풀어 `/Applications/Marketing Automation.app` 에
  swap, 앱 재시작.

ed25519 서명 (`*.sig`) 으로 무결성을 검증하므로 중간에 변조된 자산은 거부됨.

---

## 3. 자산 구성

각 릴리스는 6개 자산을 가진다:

| 파일 | 역할 | Platform |
|---|---|---|
| `Marketing.Automation_<ver>_x64-setup.exe` | Windows 첫 설치 + 자동 업데이트 payload | Windows x64 |
| `Marketing.Automation_<ver>_x64-setup.exe.sig` | 위 파일의 ed25519 서명 | Windows x64 |
| `Marketing.Automation_<ver>_aarch64.dmg` | macOS 첫 설치 disk image | macOS Apple Silicon |
| `Marketing.Automation.app.tar.gz` | macOS 자동 업데이트 payload | macOS Apple Silicon |
| `Marketing.Automation.app.tar.gz.sig` | 위 파일의 ed25519 서명 | macOS Apple Silicon |
| `latest.json` | Tauri updater manifest (양 platform 통합) | 공통 |

> 자산 이름에 공백이 점(`.`)으로 치환되어 보이는 이유 — GitHub Releases 가
> 업로드된 파일명의 공백을 자동으로 점으로 바꿔서 서빙하기 때문 (원본 빌드
> 산출물은 `Marketing Automation_...`).

---

## 4. Troubleshooting

### macOS — "손상되어 열 수 없습니다" 가 계속 뜸

`sudo xattr -dr com.apple.quarantine ...` 명령을 잘못된 경로에 적용했거나
앱을 다른 경로로 옮긴 경우. 정확한 절대경로 (드래그 install 후 위치) 로
재시도:

```bash
sudo xattr -dr com.apple.quarantine "/Applications/Marketing Automation.app"
```

또는 우클릭 → "다른 이름으로 열기" → 확인 dialog 에서 "열기" (System Settings →
Privacy & Security 에서 "그래도 열기" 버튼이 뜰 수도 있음).

### macOS — 자동 업데이트가 동작 안 함

1. 앱 메뉴 → About 에서 현재 버전 확인.
2. [최신 릴리스 페이지](https://github.com/iLab-Research/marketing-automation-releases/releases/latest)
   의 버전이 더 높은지 확인.
3. 인터넷 연결 + GitHub.com 접근 가능 여부.
4. 그래도 안 되면 .dmg 를 받아 수동 재설치 (위 §1 절차).

### Windows — SmartScreen 이 계속 차단

설치자 본인이 운영자라면 "추가 정보 → 실행" 으로 한 번 우회하면 SmartScreen
이 학습. 향후 같은 publisher (= 같은 인증서) 의 새 릴리스는 자동 통과.
코드사이닝 미적용 단계이므로 SmartScreen 학습은 파일 hash 기준.

### Windows — 설치 후 실행 시 백엔드 연결 오류

PyInstaller 로 묶인 sidecar 가 시작 안 된 상황. 일반적으로:

1. 앱 완전 종료 (시스템 트레이 아이콘도) 후 재실행.
2. Windows Defender 가 PyInstaller binary 를 quarantine 했을 수도 — Defender
   설정에서 `marketing-automation-backend.exe` 를 예외에 추가.

---

## 5. 보안 노트

- **서명 키**: Tauri ed25519 (자체 발급). Apple Developer ID / Microsoft
  Authenticode 인증서는 미적용. 이로 인해 첫 실행 시 OS-level 경고가 뜨지만,
  자동 업데이트의 무결성은 ed25519 서명으로 보장.
- **공개키** 는 앱 binary 에 임베드. 모든 platform 의 새 release 는 동일 키
  로 서명되어야 함.
- 의심스러운 출처에서 자산을 받지 말 것. 정식 채널은
  [이 repo 의 Releases 페이지](https://github.com/iLab-Research/marketing-automation-releases/releases)
  뿐이다.

