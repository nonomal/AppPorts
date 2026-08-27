---
outline: deep
---

# 데이터 마이그레이션 기본 구현

![](https://pic.cdn.shimoko.com/appports/%E6%88%AA%E5%B1%8F2026-05-08%2008.38.05.png)

AppPorts의 데이터 마이그레이션 기능은 앱과 관련된 데이터 디렉토리(예: `~/Library/Application Support`, `~/Library/Caches` 등)를 외장 저장소로 마이그레이션하여 로컬 디스크 공간을 확보합니다.

## 핵심 전략: 심볼릭 링크

데이터 디렉토리 마이그레이션은 **Whole Symlink** 전략을 사용합니다:

1. 원본 로컬 디렉토리 전체를 외장 저장소로 복사
2. 관리 링크 메타데이터(`.appports-link-metadata.plist`)를 외부 디렉토리에 기록
3. 원본 로컬 디렉토리를 같은 볼륨의 숨겨진 안전 백업으로 이름 변경
4. 원본 경로에 외부 복사본을 가리키는 심볼릭 링크 생성
5. 심볼릭 링크 생성 후 로컬 안전 백업 정리

```
~/Library/Application Support/SomeApp
    → /Volumes/External/AppPortsData/SomeApp  (symlink)
```

## 마이그레이션 흐름

```mermaid
flowchart TD
    A[데이터 디렉토리 선택] --> B{권한 및 보호 검사}
    B -->|실패| Z[종료]
    B -->|통과| C{대상 경로 충돌 감지}
    C -->|관리 메타데이터 있음| D[자동 복구 모드]
    C -->|충돌 없음| E[외장 저장소로 복사]
    D --> E
    E --> F[관리 링크 메타데이터 기록]
    F --> G[로컬 디렉토리를 안전 백업으로 이름 변경]
    G -->|실패| H[외부 복사본을 유지하고 중지]
    G -->|성공| I[심볼릭 링크 생성]
    I -->|실패| J[로컬 안전 백업 복원 및 외부 복사본 유지]
    I -->|성공| K[로컬 안전 백업 정리]
    K -->|성공| L[마이그레이션 완료]
    K -->|실패| M[마이그레이션 완료; 안전 백업 유지]
```

## 관리 링크 메타데이터

AppPorts는 외부 디렉토리에 `.appports-link-metadata.plist` 파일을 기록하여 해당 디렉토리가 AppPorts에 의해 관리됨을 식별합니다. 메타데이터에는 다음이 포함됩니다:

| 필드 | 설명 |
|------|------|
| `schemaVersion` | 메타데이터 버전 번호 (현재 1) |
| `managedBy` | 관리자 식별자 (`com.shimoko.AppPorts`) |
| `sourcePath` | 원본 로컬 경로 |
| `destinationPath` | 외장 저장소 대상 경로 |
| `dataDirType` | 데이터 디렉토리 유형 |

이 메타데이터는 스캔 시 AppPorts가 관리하는 링크와 사용자가 생성한 심볼릭 링크를 구분하는 데 사용되며, 마이그레이션 중단 시 자동 복구를 지원합니다.

자동 복구는 엄격한 일치를 사용합니다. 외부 대상이 이미 존재하는 경우 AppPorts는 `schemaVersion`, `managedBy`, `sourcePath`, `destinationPath`, `dataDirType`이 현재 작업과 모두 일치할 때만 복구 가능한 대상으로 간주합니다. 일치하는 메타데이터가 없는 실제 디렉토리는 충돌로 처리되며, 디렉토리 크기가 비슷하다는 이유만으로 복구하거나 관리 전환하지 않습니다.

재링크와 정규화는 디렉토리에만 적용됩니다. AppPorts는 외부 일반 파일을 데이터 디렉토리로 재링크하거나 이동하지 않고 거부하여, 파일이 로컬 심볼릭 링크로 대체되는 상황을 방지합니다.

## 지원되는 데이터 디렉토리 유형

| 유형 | 경로 예시 |
|------|----------|
| `applicationSupport` | `~/Library/Application Support/` |
| `preferences` | `~/Library/Preferences/` |
| `containers` | `~/Library/Containers/` |
| `groupContainers` | `~/Library/Group Containers/` |
| `caches` | `~/Library/Caches/` |
| `webKit` | `~/Library/WebKit/` |
| `httpStorages` | `~/Library/HTTPStorages/` |
| `applicationScripts` | `~/Library/Application Scripts/` |
| `logs` | `~/Library/Logs/` |
| `savedState` | `~/Library/Saved Application State/` |
| `dotFolder` | `~/.npm`, `~/.vscode` 등 |
| `custom` | 사용자 정의 경로 |

## 복원 흐름

1. 로컬 경로가 유효한 외부 디렉토리를 가리키는 심볼릭 링크인지 확인
2. 로컬 심볼릭 링크 제거
3. 외부 디렉토리를 로컬로 다시 복사
4. 외부 디렉토리 삭제 (최대한 시도)

복사가 실패하면 일관성을 유지하기 위해 심볼릭 링크를 자동으로 재구성합니다.

## 오류 처리 및 롤백

마이그레이션 프로세스의 각 핵심 단계에는 롤백 메커니즘이 포함됩니다:

- **복사 실패**: 추가 작업을 수행하지 않음; 복사된 외부 파일 정리
- **로컬 안전 백업 이동 실패**: 마이그레이션을 중지하고 외부 복사본을 유지합니다. 로컬 원본은 삭제되지 않습니다
- **심볼릭 링크 생성 실패**: 가능한 경우 로컬 안전 백업을 원래 경로로 복원하고 외부 복사본도 유지하여 양쪽 데이터가 동시에 사라지는 상황을 피합니다
- **안전 백업 정리 실패**: 마이그레이션은 완료된 것으로 간주됩니다. 로컬에 `.appports-migration-backup-*` 폴더가 남으며 확인 후 수동으로 삭제할 수 있습니다

이 설계는 어떤 단계에서든 실패 시 데이터 손실이 없고 시스템 상태가 일관되도록 보장합니다.
