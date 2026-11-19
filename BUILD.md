# 재현 빌드

이 절차는 제공된 원본 APK 한 버전에만 적용된다. 다른 APK를 입력하면 빌드가 중단된다.

```text
원본 SHA256
2c7792387492b6c3deebb52c162eeb0e147461de1c49e2d148490bec30194484
```

현재 소스는 복구 후 개발한 candidate7 계열이고, 리소스 계층은 **2.1-stock-black: 53개 파일**이다. 이전 candidate4의 소스 ZIP·재빌드 보고서를 최신 네이티브 코드와 혼용하지 않는다. 새 ZIP을 만들기 전의 개발 작업 트리와 배포 ZIP은 동일하다고 가정하지 말고, 배포 시 생성한 파일별 해시 목록을 확인한다.

## 합본 버전과 재현 기준

이 문서의 최종 개발판은 `yagu-v0.7.0-dev.apk`,59,333,007바이트, SHA256 `233d1f91579b8a41f3df3e9f951be454c5ed8453a7f8c049ef4e03cc8c3a3dc5`이다. 공유 C `0f6d03496c02621452295ffb44d9ea7fc2805828f3ccf570d3f423ddbc5bff5b`, 양 ABI369개 수정 지점, 선수 스키마4/51·기록 스키마4/40을 기준으로 한다.

로컬 카탈로그 모듈(52곳/ABI,6,012건 명령 검증)을 포함한 최종 합본이다. 빌드 시점의 실제 C 조각·빌더·리소스·공개 인증서와 출력 해시를 함께 기록한다. 아래 빌드 명령이 미래 소스에서도 위 개발판 해시를 만든다고 가정하지 않는다.

이전 e4 APK는 실제9이닝81스트라이크·27K·8:0 및 저장/재실행을 확인했으나5AB/3HR/2K로 전타석 홈런에 실패했다. 최종 v0.7.0-dev 후보의 수동 타격·전체 경기·구매는 미검증이다. 재현·서명·명령 검사가 이 실기 한계를 해소하지 않는다.

### 최종 네이티브 산출물

| ABI | libgameDSO.so SHA256 |
| --- | --- |
| arm64-v8a | `a123cae6374571402efa074e56f9db7ccce216819cb4bb4697e4a829f2b7a646` |
| armeabi-v7a | `674c709792d87652c5e856606a4bfc8f4bda8d0407a040baec94e65deca0196f` |

최종 패키지에서 원본 선수·경기장 리소스2,818개와4개 Cocos 라이브러리 보존, 서명·정렬·ZIP CRC를 확인했다. 이 파일 검사는 새 빌드의 수동 타격·상점 구매 성공 증거가 아니다.

## 필요한 입력과 도구

1. 새 디렉터리에 같은 릴리스의 `yagu-source.zip`과 `yagu-art-overlay.zip`을 푼다. `overlays/art/assets`와 `docs/art/art_overlay_manifest.json`이 있어야 한다.
2. Python3.12, `c++filt`가 포함된 binutils, Java, Android Build Tools35.0.0, Zig0.13.0을 준비한다.
3. `python3 -m pip install -r requirements.txt`로 분석·패치 의존성을 설치한다.
4. 자신의 서명 키·비밀번호 파일·그 키의 공개 인증서 DER을 공개 소스 밖에 준비한다. 개인키와 비밀번호 파일 권한은0600이어야 한다.

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

기본 키 별칭은 `yagu-local`이며, 다르면 `--key-alias`를 지정한다. 공개 인증서는 개인키가 아니므로 제공된 DER만으로 같은 서명을 만들 수 없다. 다른 인증서로 서명한 APK의 최종 해시는 달라진다.

## 빌드가 실제로 하는 일

빌드는 원본 APK를 덮어쓰지 않고 이미 존재하는 출력 APK도 거부한다. 입력 SHA256과53개 오버레이의 파일 수·바이트 수·해시·심볼릭 링크 여부를 검사한 후, 원본 gameDSO 추출, 양 ABI 네이티브 빌드, 앱 식별 정보 변경, 리소스 병합과 서명을 수행한다.

리소스 적용 순서는 다음과 같다.

1. 앱 이름·패키지 식별자.
2. 번들 SDK가 지원하는 수집 비활성화 메타데이터.
3. `2.1-stock-black` 리소스 계층.
4. 기존 위치·글꼴·스타일을 보존하는 브랜드 문구12개.
5. 원래 검사 리소스의 파일명 호환 복사.
6. 수정한 두 ABI의 `libgameDSO.so`.

브랜드 문구는 병합 리소스에도 포함돼 있지만 빌드가 다시 생성해 기존 해시와 대조한 뒤 명시적으로 마지막 텍스트 계층에 적용한다. 완성 APK는 정렬·서명·ZIP CRC와 리소스 해시 검사를 거치고, 실제 서명 인증서 DER이 네이티브 기대값을 계산한 인증서와 같은지도 확인한다. 이 과정은 게임을 실행하거나9이닝을 검증하지 않는다.

`YAGU_ZIG`, `YAGU_CERT_HASH`, `YAGU_SOURCE_APK`, `YAGU_ART_OVERLAY`는 빌드가 해당 입력에서 설정한다. 기존 서명 검사는 활성화한 채, 자신의 공개 인증서에서 계산한 `Android Signature.hashCode()`만 로컬 빌드의 기대값으로 사용한다. 계정·결제 인증이나 원래 실패 분기는 제거하지 않는다.

## 로고·식별 리소스 재생성

일반 빌드는 검증된 `overlays/art`를 사용하므로 이미지 라이브러리가 필요하지 않다. 리소스 자체를 다시 내보낼 때만 `requirements-art.txt`가 필요하다. 재생성은 포함된 생성 소스4장—로고2장, 제목1장, 아이콘1장—을 사용한다. 새 이미지 생성 서비스나 과거 생성 유니폼·얼굴 그림은 사용하지 않는다.

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
python3 build-work/art/tools/assemble_art_overlay.py \
  --base build/art-reexport \
  --base-manifest build-work/art/art_overlay_manifest.json \
  --brand build/brand-reexport \
  --brand-manifest build/brand-reexport.json \
  --out build/art-merged \
  --manifest build/art-merged.json
```

위 출력 디렉터리는 새 경로를 사용한다. 기본47개에 텍스트12개를 병합하며, 그중6개는 교체·6개는 추가이므로 최종 고유 파일은53개다. 현재 병합 파일의 총크기는2,382,750바이트다. 완성된 새 매니페스트를 기존 `docs/art/art_overlay_manifest.json`과 비교한 후 사용한다. 기본 내보내기는 `build-work/art/art_overlay_manifest.json`을 다시 작성하므로 이전 보고서가 필요하면 보관한다.

아틀라스 경계와 원래 프레임은 유지한다. `assets/bat/`, `assets/pit/`, `assets/defender/`, `assets/stadium/` 아래의 교체 파일은 없다. 검정 유니폼·장비는 원본 APK에 이미 있는 경로를 네이티브 선택자가 사용한다. Pumas의 로고는 별도로 보존한다.

## 네이티브 검증을 다시 실행할 때

`build-work/native/verify_*.py`는 소유 범위, 잘못된 포인터·모드, 상대 선수의 원래 동작, 원본 산술·테이블 상한, ASLR, 기존 UI 호출 인자 등을 실제 ARM 명령으로 검사한다. 테스트 결과 JSON의 소스 해시와 ABI를 확인한다. 소스가 바뀌었다면 이전 보고서를 새 결과로 표시하지 않는다.

```sh
export YAGU_ZIG=/absolute/path/zig
python3 build-work/native/verify_elite_world.py
python3 build-work/native/verify_honors.py
python3 build-work/native/verify_skill_points.py
python3 build-work/native/verify_mental_duel.py
python3 build-work/native/verify_team_mean.py
python3 build-work/native/verify_batter_presence.py
python3 build-work/native/verify_every_game_routing.py
```

원본 라이브러리는 빌드가 만드는 `analysis/native/lib/{ABI}/libgameDSO.so` 경로에 있어야 한다. 일부 검증기는 원본 APK를 `upload/baseball-superstars-2013-1-2-9.apk`에서 직접 읽는다. 해당 경로에는 자신의 해시가 일치하는 원본을 준비해야 하며, 공개 소스에는 APK를 넣지 않는다. 엘리트 검증기는 포함된 `analysis/data/tools/zt1_codec.py`와 원래 로스터·능력 상한 테이블을 사용한다.

테스트용 C 복사는 로컬 파일명인 quoted include를 재귀적으로 포함한다. `elite_world_policy.h`, `honors_payload.inc`, `award_record_payload.inc` 등 실제 소스가 참조하는 조각을 빠뜨리지 않는다. 테스트 출력·추출 `.so`·컴파일 산출물은 소스 배포 대상이 아니다.

SP1,604건·멘탈3,032건·팀 평균692건·타자 보호994건·원본 포스트시즌/컵 routing240건은 각 보고서에 기록된 특정 소스의 명령 검증이다. 멘탈/팀/출전 검증이 사용하는 `verify_original_plt_cache.py`도 포함한다. 이 helper는 불변 ELF 메타데이터만 캐시하며 실행 결과·에뮬레이터 메모리는 공유하지 않는다. 카탈로그 검증기는 `verify_local_catalog.py`이며 후속 모듈6,012건의 결과를 최종 APK 실기로 혼동하지 않는다.

추가한 `.inc`에는 `skill_points_payload.inc`, `mental_duel_payload.inc`, `team_mean_payload.inc`, `batter_presence_payload.inc`와 후속 카탈로그 조각이 있다. 실제 shared C의 quoted include 전체가 같은 소스 묶음에 들어 있어야 한다. 플레이어/기록51/40과 별도 SP·지갑·파생 팀 평균의 개수를 혼합하지 않는다.

현재 실제 Android 실행 증거는 [current-runtime-public.md](build-work/android/current-runtime-public.md)에서 APK 해시와 함께 확인한다. 서명·재현성·명령 검증·실제 경기는 서로 다른 증거다. 과거 checkpoint4의 바이트 일치 재빌드 기록은 그 과거 후보에만 적용한다.

## 개발판 카탈로그 가격

원본 일부 상점 가격은 서버 응답으로 채워지는 BSS다. APK만으로 운영 서버 가격을 모두 복원했다고 설명하지 않는다. 로컬 카탈로그는 APK 내 표의 유한 값을 명시적인 개발판 가격으로 채택하고, 내장 가격이 없는 헤어/염색은1G, 스킨은1Star로 정한다. 원본 BSS·NetResult·결제·영수증 코드는 변경하지 않는다. [LOCAL_CATALOG_POLICY.md](build-work/native/LOCAL_CATALOG_POLICY.md)가 채택한 표·가격과 정확한 UI 호출 범위를 정의한다.

이전317곳 preflight에는 카탈로그가 없었고, 현재369곳 개발판에는 포함된다. 배포 시 카탈로그 정책·C 조각·52개 호출 모듈·검증 JSON과 최종 APK 해시를 같은 패키징 시점으로 고정한다. 로컬 가격과 서버의 현재 가격은 다른 값의 출처다.

## 실제 APK 웹 스트림

`web/`은 Android가 렌더링한 실제 화면과 게임 PCM을 전달하고 터치를 되돌려 보낸다. Node.js22 이상에서 다음 명령으로 웹 단위 검증과 로컬 서버를 실행한다.

```sh
cd web
npm test
YAGU_STREAM_DIR=/absolute/path/stream npm run dev -- --host 127.0.0.1
```

별도로 실행 중인 Android 에뮬레이터와 [STREAM_BRIDGE.md](build-work/android/STREAM_BRIDGE.md)의 스트림 계약이 필요하다. Python gRPC 의존성과 `emulator_controller_pb2` 모듈은 설치한 공식 에뮬레이터의 `lib/emulator_controller.proto`에서 준비한다. SDK·생성된 SDK 바인딩·에뮬레이터 디스크·발견 토큰은 소스 번들에 포함하지 않는다. 에뮬레이터 실행 옵션, 그래픽 호환 환경과 네트워크 기본 차단은 브리지 문서를 따른다.

실제 API24 장치의 프레임·터치·게임 소리 캡처는 확인했다. 현재 소유자 비공개 테스트 URL은 [https://yagu-universal-play.taehyun051027.chatgpt.site](https://yagu-universal-play.taehyun051027.chatgpt.site)이다. 실제 스트림 API200은 확인했으나 릴레이 지연을 조정 중이며 브라우저 전체 플레이 확인은 별도 상태다. 프레임 수신이나 게임 소리가 없으면 클라이언트는 이를 대신 만들어 내지 않는다.

`web/.openai/hosting.json`은 비밀이 아닌 프로젝트·저장소 바인딩 명세만 포함한다. `npm run build`는 그 명세를 요구하므로 다른 Sites 프로젝트에서는 해당 프로젝트의 명세로 교체한다. 서비스 인증값이나 호스팅 권한은 공개 소스에 포함되지 않는다. 최신 웹 소스는 지정한 Sites 작업 트리에서 패키징하며 루트의 과거 `web/` 복사본과 혼용하지 않는다.

## 기존 검사 리소스와 수집 설정

`prepare_checker_asset_compat.py`는 원본 APK의 `assets/20140212.omg`를 SDK가 기대하는 `assets/20140120.omg` 이름으로 정확히 복사한다.12,576바이트와 원래265개 정책 항목·체크섬을 확인하며 파서·비교·실패 분기를 유지한다. 새 서버 응답이나 검사 정책을 만들지 않는다. 기존 Retry의 로컬 fallback과 이 호환 계층이 있는 checkpoint4에서 외부 통신을 차단한 채 커리어와 첫 경기까지 진행했다.

`prepare_privacy_metadata.py`는 번들 SDK가 읽는 여섯 메타데이터만 설정한다. 이전 저장 설정이 우선할 수 있으므로 이 플래그만으로 모든 외부 연결이 차단된다고 주장하지 않는다. 현재 런타임 시험은 별도의 기본 차단 프록시·닫힌 DNS·장치 네트워크 경계를 사용한다.

## 공개 소스의 경계

공개 묶음은 빌드·검증 코드, `.h`/`.inc` 조각, 선택한 보고서, 공개 인증서와 사용된 새 원화만 포함한다. 원본 APK·원본/수정 `.so`·컴파일러·SDK·개인키·비밀번호·토큰·AVD·게임 저장·실시간 화면/소리/입력 큐는 제외한다. 서명 APK 자체는 소스 ZIP과 구분되는 별도 결과물이다. [SOURCE_RELEASE_AUDIT.md](docs/SOURCE_RELEASE_AUDIT.md)의 선택 기준과 최종 번들 해시 목록을 확인한다.
