# STM32 VSCode 개발 템플릿

STM32CubeMX에서 만든 CMake 프로젝트를 VSCode에서 바로 Configure, Build, Debug하기 위한 설정 템플릿이다.

이 저장소의 핵심 파일은 아래 하나다.

```text
STM32_VSCode_Template.zip
```

zip 안의 `copy_to_project_root` 내용을 STM32 프로젝트 루트에 복사해서 쓴다.

## 현재 기준 경로

현재 작업 기준 경로는 아래처럼 정리했다.

```text
도구 루트:      C:\step_tracer\stm_steptracer\StmCubeMx
프로젝트 루트:  C:\step_tracer\stm_steptracer\StmCubeMx\stm_steptracer
임시 폴더:      C:\step_tracer\stm_steptracer\StmCubeMx\tmp
```

예전에 쓰던 `D:\maze\StmCubeMx` 경로는 더 이상 기준으로 쓰지 않는다.

## 필요한 프로그램

아래는 미리 설치되어 있어야 한다.

```text
STM32CubeMX
Visual Studio Code 또는 Antigravity
STM32CubeIDE for Visual Studio Code 확장
Cortex-Debug 확장
GNU tools for STM32
OpenOCD
ST-LINK USB Driver
```

OpenOCD는 예를 들어 PowerShell에서 이렇게 설치할 수 있다.

```powershell
winget install --id xpack-dev-tools.openocd-xpack --source winget --accept-package-agreements --accept-source-agreements
```

## 적용 방법

1. STM32CubeMX에서 `Toolchain / IDE = CMake`로 프로젝트를 만든다.
2. VSCode에서 `CMakeLists.txt`가 있는 프로젝트 루트 폴더를 연다.
3. `STM32_VSCode_Template.zip`을 푼다.
4. `copy_to_project_root` 안의 내용을 프로젝트 루트에 복사한다.

복사 후 프로젝트 루트는 대략 이렇게 된다.

```text
stm_steptracer
├─ Core
├─ Drivers
├─ cmake
│  └─ gcc-arm-none-eabi.cmake
├─ .vscode
│  ├─ launch.json
│  ├─ settings.json
│  └─ c_cpp_properties.json
├─ .clangd
├─ CMakeLists.txt
├─ CMakePresets.json
└─ stm_steptracer.ioc
```

## 경로 확인

다른 PC나 다른 폴더에서 쓰면 아래 3개 파일의 경로를 먼저 확인한다.

```text
CMakePresets.json
cmake\gcc-arm-none-eabi.cmake
.vscode\launch.json
```

현재 템플릿의 기본 경로는 아래 기준이다.

```text
C:/step_tracer/stm_steptracer/StmCubeMx
```

JSON/CMake 파일 안에서는 Windows `\` 대신 `/`를 쓰는 편이 안전하다.

## Configure와 Build

`CMake: Configure`는 실제 컴파일이 아니라 빌드 준비 단계다.

이 단계에서 CMake가 `CMakeLists.txt`, `CMakePresets.json`, 툴체인 파일을 읽고 아래 파일들을 만든다.

```text
build\Debug\CMakeCache.txt
build\Debug\build.ninja
```

그 다음 `CMake: Build`가 실제로 `arm-none-eabi-gcc`를 실행해서 `.elf`를 만든다.

기본 순서는 이렇다.

```text
Ctrl + Shift + P
CMake: Select Configure Preset
Debug
CMake: Configure
CMake: Build
```

성공하면 아래 파일이 생긴다.

```text
build\Debug\stm_steptracer.elf
```

## 폴더를 옮겼을 때

프로젝트 위치를 옮겼다면 기존 `build` 폴더는 지우고 다시 Configure한다.

```powershell
Remove-Item -Recurse -Force .\build
```

이걸 안 하면 `CMakeCache.txt`나 `build.ninja` 안에 예전 경로가 남아서 Configure/Build가 실패한다.

## Debug

디버그는 `.vscode\launch.json`을 쓴다.

확인할 값은 아래 4개다.

```json
"executable": "${workspaceFolder}/build/Debug/stm_steptracer.elf",
"serverpath": "C:/step_tracer/stm_steptracer/StmCubeMx/openocd/.../bin/openocd.exe",
"gdbPath": "C:/step_tracer/stm_steptracer/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gdb.exe",
"searchDir": [
    "C:/step_tracer/stm_steptracer/StmCubeMx/openocd/.../openocd/scripts"
]
```

프로젝트 이름이 `stm_steptracer`가 아니면 `executable`의 `.elf` 파일명도 프로젝트 이름에 맞게 바꾼다.

## 자주 나는 문제

### Configure failed

대부분 예전 `build` 캐시가 남았거나 경로가 틀린 경우다.

```powershell
Remove-Item -Recurse -Force .\build
```

그 다음 `CMake: Configure`를 다시 실행한다.

### CMake가 D:\maze\StmCubeMx를 계속 찾음

`build` 폴더가 예전 경로를 들고 있는 상태다. `build`를 지우고 다시 Configure한다.

### CMAKE_MAKE_PROGRAM 또는 Ninja 경로 오류

`CMakePresets.json`의 `CMAKE_MAKE_PROGRAM`을 확인한다.

현재 템플릿은 사용자 이름 한글 깨짐을 피하려고 아래처럼 잡는다.

```json
"CMAKE_MAKE_PROGRAM": "$env{LOCALAPPDATA}/stm32cube/bundles/ninja/1.13.2+st.1/bin/ninja.exe"
```

버전이 다르면 실제 `ninja.exe` 위치에 맞게 바꾼다.

### launch.json에 노란 밑줄이 많음

`Cortex-Debug` 확장이 설치 또는 활성화되어 있는지 확인한다.

설치 후에도 남으면 `Developer: Reload Window`를 실행한다.

### OpenOCD config file not found

`.vscode\launch.json`의 `searchDir`이 OpenOCD의 `scripts` 폴더를 가리키는지 확인한다.

`bin` 폴더가 아니라 `openocd/scripts` 폴더여야 한다.

### USB 안전 제거가 안 됨

전역 `TEMP/TMP`를 USB 드라이브로 잡아두면 백그라운드 프로그램이 USB를 계속 잡을 수 있다.

현재 기준은 C:의 아래 경로를 쓴다.

```text
C:\step_tracer\stm_steptracer\StmCubeMx\tmp
```
