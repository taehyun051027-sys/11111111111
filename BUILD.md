# 재현 빌드

이 소스는 제공된 원본 APK 한 버전에만 적용됩니다. SHA256이 다르면 중단합니다.

```text
2c7792387492b6c3deebb52c162eeb0e147461de1c49e2d148490bec30194484
```

## 필요한 파일과 도구

1. `yagu-source.zip`을 풉니다.
2. 같은 디렉터리에 `yagu-art-overlay.zip`을 풀어 `overlays/art/assets`가 있도록 합니다.
3. Python 3.12, `c++filt`가 포함된 binutils, Java, Android Build Tools 35.0.0, Zig 0.13.0을 준비합니다.
4. `python3 -m pip install -r requirements.txt`로 분석·패치 의존성을 설치합니다.
5. 본인의 서명 키와 비밀번호 파일, 그 키의 공개 인증서 DER을 준비합니다. 서명 관련 파일은 공개 저장소 밖에 두고 권한을 0600으로 설정합니다.

```sh
python3 build.py \
  --apk /absolute/path/original.apk \
  --zig /absolute/path/zig \
  --build-tools /absolute/path/android-build-tools/35.0.0 \
  --public-certificate /private/path/public-certificate.der \
  --keystore /private/path/signing-key.jks \
  --password-file /private/path/keystore-password.txt \
  --output /absolute/path/yagu.apk
```

빌드는 원본 파일을 덮어쓰지 않습니다. 기존 출력 APK가 있으면 새 출력 이름을 지정해야 합니다. 원본 라이브러리 추출, 두 ABI의 네이티브 패치 생성, 앱 식별자와 이름 변경, 리소스 병합, 서명, 정렬·CRC·오버레이 검사, 실제 서명 인증서 일치 검사를 순서대로 수행합니다.

리소스 적용 순서는 앱 식별자 → 수집 비활성화 메타데이터 → 병합 원화 v1.2 → 브랜드 문구 v1.2 → 기존 검사 리소스 파일명 호환 계층 → 네이티브 라이브러리입니다. 원화 묶음에는 문구 변경이 이미 병합되어 있습니다. 빌드는 동일한 12개 문구 파일을 다시 생성해 해시를 확인한 뒤 마지막 리소스 계층으로 적용합니다. 메뉴 상단의 `야구`와 투타겸업 설명이 오래된 원화 계층에 의해 되돌아가지 않도록 순서를 명시했습니다. 기존 화면 좌표·글꼴·스타일 필드는 바꾸지 않습니다.

다른 인증서로 서명하면 완성 APK의 바이트와 해시는 달라집니다. 이 재현 빌드는 동일한 입력·코드·도구로 네이티브 패치와 원화 리소스를 재생성하는 절차입니다. 제공된 공개 인증서만으로 배포 APK와 동일한 서명을 만들 수는 없습니다.

## 원화 리소스 재생성

기본 APK 빌드는 미리 검증된 `overlays/art`를 사용하므로 이미지 라이브러리가 필요하지 않습니다. 포함된 생성 원화 17장에서 리소스를 다시 내보낼 때만 다음 의존성과 명령이 필요합니다. 새 그림을 다시 생성하는 과정은 없습니다.

```sh
python3 -m pip install -r requirements-art.txt
python3 build-work/art/tools/build_art_overlay.py \
  --apk /absolute/path/original.apk \
  --out build/art-reexport
python3 build-work/art/tools/make_brand_delta.py \
  --apk /absolute/path/original.apk \
  --base-overlay build/art-reexport \
  --out build/brand-reexport \
  --manifest build/brand-reexport.json
```

두 출력 디렉터리는 기본 원화 → 브랜드 문구 순서로 병합합니다. 원화 도구가 생성하는 보고서는 `build-work/art/art_overlay_manifest.json`이며, 배포용으로 병합된 전체 해시 목록은 `docs/art/art_overlay_manifest.json`입니다. 재생성 전 보고서가 필요하면 별도로 보관하십시오. 포함된 `verification/rebuild_verification.json`은 candidate4의 양쪽 ABI와 원화 1,408개가 독립 재빌드에서 원래 후보와 일치했음을 기록합니다.

## 서명 호환 설정

원본 게임은 시작할 때 `Android Signature.hashCode()`를 등록된 서명 값과 비교합니다. 직접 빌드한 APK는 본인의 인증서로 다시 서명되므로, 그 공개 인증서에서 계산한 값으로 로컬 빌드의 기대값을 설정합니다. 검사 활성화 플래그와 실패 분기, 네트워크 계정·결제·권한 코드는 그대로 유지합니다.

`build.py`는 `YAGU_CERT_HASH`를 공개 인증서로부터 계산하고, `YAGU_ZIG`로 명시적으로 제공된 컴파일러를 사용합니다. 완성 APK의 실제 공개 인증서가 지정된 DER과 같지 않으면 실패합니다. 공개 DER은 개인키가 아닙니다.

## 검증의 범위

`verification/`의 보고서는 해당 빌드 시점의 결과입니다. 소스를 수정해 다시 빌드하면 기존 검증 보고서를 새 결과처럼 사용하면 안 됩니다. 명령 단위 CPU 검사, 서명·파일 검사, 실제 Android 플레이 검사는 각각 다른 증거입니다. 런타임 검증과 제한은 `docs/runtime_observations.md`에 기록합니다.

네이티브 검증은 `build-work/native/verify_*.py`에 있습니다. 원본 APK·수정 라이브러리·Python 의존성이 필요한 테스트가 있으므로 각 파일의 실행 경로를 확인하십시오. 원본 APK와 대형 SDK·에뮬레이터·개인키는 이 소스 묶음에 포함하지 않습니다.

검증을 다시 실행할 때는 `YAGU_ZIG=/absolute/path/zig`를 설정합니다. 빌드가 추출한 라이브러리는 `analysis/native/lib/{ABI}/libgameDSO.so`에 있어야 합니다. 의상 경로 테스트는 기본적으로 `overlays/art/assets`를 읽고, 필요하면 `YAGU_ART_OVERLAY`로 다른 디렉터리를 지정할 수 있습니다. 기존 검증 JSON은 `verification/native`에 보관하고, 재실행 결과는 검증 스크립트 옆에 생성합니다.

## 웹 개발 화면

`web/`은 실제 Android 프레임을 표시하고 터치를 전달합니다. APK를 JavaScript 게임으로 다시 구현하지 않습니다. `web/CLIENT_CONTRACT.md`와 `build-work/android/STREAM_BRIDGE.md`의 실행 계약을 확인하십시오. 개발 세션의 에뮬레이터가 꺼지면 연결 상태가 오프라인으로 바뀝니다. 이 스냅샷에서 실제 음성 전송은 확인하지 않았습니다.

Node.js 22 이상에서 `cd web && npm test`로 웹 입력·릴레이 검증을 실행할 수 있습니다. 로컬 서버는 `YAGU_STREAM_DIR=/absolute/path/stream npm run dev -- --host 127.0.0.1`로 실제 에뮬레이터 스트림 디렉터리를 지정합니다. 배포용 Worker는 Sites의 비공개 접근 경계와 `BUCKET` 바인딩을 전제로 합니다. 사이트별 배포 설정·서비스 인증값은 이 묶음에 포함하지 않으며, 공개 소스만으로 호스팅 권한이나 실행 중인 에뮬레이터가 생기지는 않습니다.

## 기존 검사 리소스와 수집 설정

`prepare_checker_asset_compat.py`는 원본 APK 안에 이미 있는 `assets/20140212.omg`를 SDK가 기대하는 `assets/20140120.omg` 이름으로 정확히 복사한다. 파일 SHA256과 원래265개 정책 항목의 체크섬을 확인하며 정책·검사 코드·실패 분기를 수정하지 않는다. 정상 Retry 동작이 원래 로컬 fallback을 호출하는 경로다. 실제 오프라인 진행 성공은 아직 검증 중이다.

`prepare_privacy_metadata.py`는 번들된 SDK가 읽는 여섯 개 메타데이터만 설정한다. 이전 저장 설정이 우선할 수 있으므로 이 플래그만으로 외부 통신 전체 차단을 주장하지 않는다. 제공된 런타임 기록은 기본 차단 환경에서 별도로 갱신한다.
