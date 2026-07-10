# STM 보드 개발

STM32CubeMX로 새 프로젝트를 만들고, VSCode에서 CMake 빌드와 ST-LINK 업로드/디버그까지 바로 이어가기 위한 템플릿이다.

이 저장소의 핵심 파일은 아래 zip이다.

```text
STM32_VSCode_Template.zip
```

이 zip은 STM32 코드 자체가 아니라, 새 STM32 프로젝트에 복사해서 쓰는 VSCode/CMake/OpenOCD 설정 템플릿이다.

---

## 1. 설치해야 하는 프로그램

먼저 아래 프로그램을 설치한다.

| 항목 | 역할 |
|---|---|
| STM32CubeMX | 핀, 클럭, ADC, TIM, SPI, I2C 같은 주변장치 설정과 코드 생성 |
| Visual Studio Code | 코드 편집, 빌드, 디버그 실행 |
| STM32CubeIDE for Visual Studio Code 확장 | VSCode에서 STM32Cube 프로젝트를 인식하고 CMake 빌드를 돕는 확장 |
| Cortex-Debug 확장 | VSCode에서 STM32 디버그/업로드를 실행하는 확장 |
| STM32Cube Bundles Manager의 GNU tools for STM32 | `arm-none-eabi-gcc`, `arm-none-eabi-gdb` 같은 ARM 컴파일/디버그 도구 |
| OpenOCD | GDB와 ST-LINK 사이를 연결해서 STM32에 코드를 올리는 도구 |
| ST-LINK USB Driver | PC가 ST-LINK를 인식하게 해주는 드라이버 |

VSCode 확장은 최소한 아래 두 개는 설치한다.

```text
STM32CubeIDE for Visual Studio Code
Cortex-Debug
```

OpenOCD는 PowerShell에서 아래 명령으로 설치할 수 있다.

```powershell
winget install --id xpack-dev-tools.openocd-xpack --source winget --accept-package-agreements --accept-source-agreements
```

---

## 2. 작업 폴더 추천

한글 사용자 이름 경로에서 GCC나 OpenOCD가 꼬일 수 있으므로, 프로젝트는 가능하면 영문 경로 아래에 만든다.

추천 예시는 아래와 같다.

```text
D:\maze\StmCubeMx
```

새 프로젝트 이름이 `my_project`라면 최종 구조는 이런 식이 된다.

```text
D:\maze\StmCubeMx\my_project
```

---

## 3. STM32CubeMX에서 새 프로젝트 만들기

1. STM32CubeMX 실행
2. MCU 또는 보드 선택
3. Pinout & Configuration에서 핀 설정
4. Clock Configuration에서 클럭 설정
5. 필요한 주변장치 설정
   - GPIO
   - ADC
   - TIM/PWM
   - SPI
   - I2C
   - USART
   - DMA
   - NVIC
6. Project Manager로 이동
7. Project Name에 프로젝트 이름 입력

예시는 아래와 같다.

```text
Project Name: my_project
Project Location: D:\maze\StmCubeMx
Toolchain / IDE: CMake
Application Structure: Advanced
```

`Do not generate the main()`은 체크하지 않는다.

그 다음 `GENERATE CODE`를 누른다.

성공하면 아래 같은 폴더가 생긴다.

```text
D:\maze\StmCubeMx\my_project
├─ Core
├─ Drivers
├─ cmake
├─ CMakeLists.txt
├─ CMakePresets.json
├─ my_project.ioc
├─ startup_*.s
└─ *_FLASH.ld
```

이 단계의 의미는 CubeMX가 STM32 초기화 코드와 CMake 프로젝트 기본 파일을 만들어주는 것이다.

---

## 4. VSCode에서 프로젝트 열기

VSCode에서 아래 폴더를 연다.

```text
D:\maze\StmCubeMx\my_project
```

주의할 점은 `Core` 폴더만 여는 것이 아니라, `CMakeLists.txt`가 있는 프로젝트 루트 폴더를 열어야 한다.

---

## 5. 템플릿 zip 적용하기

이 저장소에서 아래 파일을 다운로드한다.

```text
STM32_VSCode_Template.zip
```

압축을 풀면 아래 구조가 있다.

```text
copy_to_project_root
├─ .clangd
├─ CMakePresets.json
├─ cmake
│  └─ gcc-arm-none-eabi.cmake
└─ .vscode
   ├─ c_cpp_properties.json
   ├─ launch.json
   └─ settings.json
```

`copy_to_project_root` 폴더 안의 내용물을 새 STM32 프로젝트 루트에 복사한다.

예시로 `my_project`에 적용하면 아래 위치에 들어가야 한다.

```text
D:\maze\StmCubeMx\my_project\.clangd
D:\maze\StmCubeMx\my_project\CMakePresets.json
D:\maze\StmCubeMx\my_project\cmake\gcc-arm-none-eabi.cmake
D:\maze\StmCubeMx\my_project\.vscode\launch.json
D:\maze\StmCubeMx\my_project\.vscode\settings.json
D:\maze\StmCubeMx\my_project\.vscode\c_cpp_properties.json
```

기존 파일이 있으면 덮어써도 된다.

---

## 6. 경로 수정하기

템플릿 파일에는 기존 PC 기준 경로가 들어있으므로, 본인 PC 경로에 맞게 수정해야 한다.

여기서 헷갈리기 쉬운 포인트는 "어떤 경로를 넣어야 하는지"다.

먼저 본인 PC에서 아래 4개 경로를 찾는다.

| 이름 | 의미 | 예시 |
|---|---|---|
| 프로젝트 루트 경로 | `CMakeLists.txt`가 있는 새 STM32 프로젝트 폴더 | `D:\maze\StmCubeMx\my_project` |
| GCC bin 경로 | `arm-none-eabi-gcc.exe`, `arm-none-eabi-gdb.exe`가 들어있는 폴더 | `D:\maze\StmCubeMx\stm32tools\14.3.1+st.2\bin` |
| OpenOCD 실행파일 경로 | `openocd.exe` 파일의 전체 경로 | `D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7\bin\openocd.exe` |
| OpenOCD scripts 경로 | `interface\stlink.cfg`, `target\stm32g4x.cfg`를 찾을 수 있는 scripts 폴더 | `D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7\openocd\scripts` |

경로를 찾기 어렵다면 PowerShell에서 아래처럼 확인한다.

```powershell
where.exe arm-none-eabi-gcc
where.exe arm-none-eabi-gdb
where.exe openocd
```

`where.exe`로 안 나오면 직접 설치한 폴더 안에서 찾는다.

```powershell
Get-ChildItem -Recurse "D:\maze\StmCubeMx\stm32tools" -Filter arm-none-eabi-gcc.exe
Get-ChildItem -Recurse "D:\maze\StmCubeMx\stm32tools" -Filter arm-none-eabi-gdb.exe
Get-ChildItem -Recurse "D:\maze\StmCubeMx\openocd" -Filter openocd.exe
```

JSON과 CMake 파일 안에서는 Windows `\` 대신 `/`를 쓰는 것을 추천한다.

예를 들어:

```text
D:\maze\StmCubeMx\stm32tools\14.3.1+st.2\bin
```

이 경로는 설정 파일 안에서 아래처럼 쓴다.

```text
D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin
```

수정 대상은 아래 3개다.

```text
CMakePresets.json
cmake\gcc-arm-none-eabi.cmake
.vscode\launch.json
```

### 6-1. CMakePresets.json 수정

이 파일은 CMake가 빌드할 때 어떤 GCC를 쓸지, 임시 파일은 어디에 만들지 알려주는 파일이다.

아래의 `GCC bin 경로` 부분을 본인 PC의 GCC bin 경로로 바꾼다.

그리고 `TEMP`, `TMP`도 같이 바꿔야 한다.  
이 둘은 GCC가 빌드 중 임시 파일을 만드는 폴더다. 한글 사용자 이름 경로를 타면 `Cannot create temporary file` 같은 오류가 날 수 있으므로, 가능하면 영문 경로의 임시 폴더를 직접 만들어서 넣는다.

```json
"PATH": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin;$penv{PATH}",
"TEMP": "D:/maze/StmCubeMx/tmp",
"TMP": "D:/maze/StmCubeMx/tmp"
```

예를 들어 본인 GCC가 아래에 있다면:

```text
D:\tools\stm32\gnu-tools-for-stm32\14.3.1+st.2\bin
```

`CMakePresets.json`에서는 이렇게 바꾼다.

```json
"PATH": "D:/tools/stm32/gnu-tools-for-stm32/14.3.1+st.2/bin;$penv{PATH}",
"TEMP": "D:/maze/StmCubeMx/tmp",
"TMP": "D:/maze/StmCubeMx/tmp"
```

여기서 `D:/maze/StmCubeMx/tmp`는 예시다. 본인 PC에서 임시 폴더를 `D:\stm32_tmp`로 만들었다면 아래처럼 바꾼다.

```json
"TEMP": "D:/stm32_tmp",
"TMP": "D:/stm32_tmp"
```

중요한 점은 `TEMP`와 `TMP` 둘 다 바꾸는 것이다. 하나만 바꾸면 어떤 도구는 여전히 기존 `C:\Users\사용자이름\AppData\Local\Temp` 쪽을 볼 수 있다.

그리고 아래 컴파일러 경로도 같은 GCC bin 경로 기준으로 바꾼다.

```json
"CMAKE_C_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_ASM_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_CXX_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-g++.exe"
```

위의 예시 경로를 쓰는 사람이라면 이렇게 바꾼다.

```json
"CMAKE_C_COMPILER": "D:/tools/stm32/gnu-tools-for-stm32/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_ASM_COMPILER": "D:/tools/stm32/gnu-tools-for-stm32/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_CXX_COMPILER": "D:/tools/stm32/gnu-tools-for-stm32/14.3.1+st.2/bin/arm-none-eabi-g++.exe"
```

임시 폴더가 없다면 직접 만든다.

```powershell
mkdir D:\maze\StmCubeMx\tmp
```

빌드 로그에 아직도 아래처럼 `C:\Users\...\Temp`가 보이면, VSCode를 완전히 껐다가 다시 켠 뒤 `CMake: Delete Cache and Reconfigure`를 실행한다.

```text
Cannot create temporary file in C:\Users\...\Temp
```

### 6-2. cmake/gcc-arm-none-eabi.cmake 수정

이 파일도 CMake에게 실제 ARM용 GCC가 어디 있는지 알려주는 파일이다.

아래 줄을 본인 GCC bin 경로로 수정한다.

```cmake
set(TOOLCHAIN_PATH "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin")
```

예를 들어 본인 GCC bin 경로가 아래라면:

```text
D:\tools\stm32\gnu-tools-for-stm32\14.3.1+st.2\bin
```

이렇게 쓴다.

```cmake
set(TOOLCHAIN_PATH "D:/tools/stm32/gnu-tools-for-stm32/14.3.1+st.2/bin")
```

### 6-3. .vscode/launch.json 수정

이 파일은 디버그/업로드할 때 어떤 `.elf`를 올릴지, OpenOCD와 GDB가 어디 있는지 알려주는 파일이다.

아래 항목을 수정한다.

```json
"executable": "${workspaceFolder}/build/Debug/my_project.elf",
"serverpath": "D:/maze/StmCubeMx/openocd/.../bin/openocd.exe",
"gdbPath": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gdb.exe",
"searchDir": [
    "D:/maze/StmCubeMx/openocd/.../openocd/scripts"
]
```

`executable`은 빌드 결과로 생기는 `.elf` 파일 경로다.

`my_project` 부분은 CubeMX에서 정한 Project Name과 맞춰야 한다.

예를 들어 프로젝트 이름이 `my_project`이면 아래처럼 쓴다.

```json
"executable": "${workspaceFolder}/build/Debug/my_project.elf"
```

프로젝트 이름이 `stm_test`이면 이렇게 바꾼다.

```json
"executable": "${workspaceFolder}/build/Debug/stm_test.elf"
```

`serverpath`에는 `openocd.exe` 파일의 전체 경로를 넣는다.

OpenOCD 실행파일이 아래에 있다면:

```text
D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7\bin\openocd.exe
```

`launch.json`에는 이렇게 쓴다.

```json
"serverpath": "D:/maze/StmCubeMx/openocd/xpack-openocd-0.12.0-7/bin/openocd.exe"
```

`gdbPath`에는 `arm-none-eabi-gdb.exe` 파일의 전체 경로를 넣는다.

GDB가 아래에 있다면:

```text
D:\maze\StmCubeMx\stm32tools\14.3.1+st.2\bin\arm-none-eabi-gdb.exe
```

`launch.json`에는 이렇게 쓴다.

```json
"gdbPath": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gdb.exe"
```

`searchDir`에는 `openocd.exe`가 있는 `bin` 폴더가 아니라, OpenOCD의 `scripts` 폴더를 넣는다.

OpenOCD scripts 폴더가 아래에 있다면:

```text
D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7\openocd\scripts
```

`launch.json`에는 이렇게 쓴다.

```json
"searchDir": [
    "D:/maze/StmCubeMx/openocd/xpack-openocd-0.12.0-7/openocd/scripts"
]
```

정리하면 `launch.json`의 핵심은 아래 4개다.

```json
"executable": "${workspaceFolder}/build/Debug/my_project.elf",
"serverpath": "D:/maze/StmCubeMx/openocd/xpack-openocd-0.12.0-7/bin/openocd.exe",
"gdbPath": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gdb.exe",
"searchDir": [
    "D:/maze/StmCubeMx/openocd/xpack-openocd-0.12.0-7/openocd/scripts"
]
```

---

## 7. 경로가 맞는지 확인하기

VSCode 터미널 또는 PowerShell에서 아래처럼 확인한다.

```powershell
Test-Path "D:\maze\StmCubeMx\stm32tools\14.3.1+st.2\bin\arm-none-eabi-gcc.exe"
Test-Path "D:\maze\StmCubeMx\stm32tools\14.3.1+st.2\bin\arm-none-eabi-gdb.exe"
Test-Path "D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7\bin\openocd.exe"
```

전부 `True`가 떠야 한다.

OpenOCD는 아래 명령으로도 확인할 수 있다.

```powershell
openocd --version
```

---

## 8. CMake Configure 실행

이 단계는 빌드를 시작하기 전에 CMake가 프로젝트 구조를 읽고 `build/Debug` 폴더와 `CMakeCache.txt`를 만드는 과정이다.

VSCode에서:

1. `Ctrl + Shift + P`
2. `CMake: Select Configure Preset`
3. `Debug` 선택
4. `CMake: Configure` 실행

이미 설정이 꼬인 상태라면 아래를 실행한다.

```text
CMake: Delete Cache and Reconfigure
```

성공하면 터미널에 아래 비슷한 문구가 나온다.

```text
Configuring done
Generating done
Build files have been written to: .../build/Debug
```

그리고 아래 파일이 생긴다.

```text
build\Debug\CMakeCache.txt
```

---

## 9. 빌드하기

빌드는 C 코드를 컴파일해서 `.elf` 실행 파일을 만드는 과정이다.

방법은 둘 중 하나다.

```text
Ctrl + Shift + B
```

또는 VSCode 아래쪽의 `Build` 버튼을 누른다.

성공하면 아래처럼 나온다.

```text
Memory region         Used Size  Region Size  %age Used
RAM:                  ...
FLASH:                ...
build finished successfully.
```

그리고 아래 파일이 생긴다.

```text
build\Debug\my_project.elf
```

이 `.elf` 파일이 STM32에 올라갈 실제 펌웨어 파일이다.

---

## 10. 디버그/업로드하기

디버그/업로드는 `.elf` 파일을 ST-LINK와 OpenOCD를 통해 STM32 플래시에 넣고, 필요하면 코드 실행을 멈추거나 변수 값을 보는 과정이다.

먼저 하드웨어를 연결한다.

```text
ST-LINK SWDIO
ST-LINK SWCLK
ST-LINK GND
Target Board Power
```

VSCode에서:

1. 왼쪽 `Run and Debug` 아이콘 클릭
2. 상단에서 `프로젝트이름 Debug` 선택
3. 초록색 실행 버튼 클릭

성공하면 코드가 STM32에 업로드되고 `main()`에서 멈춘다.

계속 실행하려면 상단 디버그 바의 Continue 버튼을 누른다.

---

## 11. 자주 나는 오류

### Error: not a CMake build directory

뜻:

```text
build/Debug/CMakeCache.txt가 없어서 빌드할 수 없음
```

해결:

```text
CMake: Configure
```

또는

```text
CMake: Delete Cache and Reconfigure
```

### liblto_plugin.dll error loading plugin

뜻:

```text
GCC 경로가 한글 사용자 폴더나 잘못된 위치를 보고 있을 가능성이 큼
```

해결:

```text
CMakePresets.json
cmake/gcc-arm-none-eabi.cmake
```

두 파일의 GCC 경로를 확인하고 다시 `Delete Cache and Reconfigure`를 한다.

### Cannot create temporary file in C:\Users\...\Temp

뜻:

```text
GCC가 임시 파일을 한글 사용자 경로에 만들다가 실패함
```

해결:

`CMakePresets.json`에서 아래를 영문 경로로 지정한다.

```json
"TEMP": "D:/maze/StmCubeMx/tmp",
"TMP": "D:/maze/StmCubeMx/tmp"
```

### OpenOCD config file not found

뜻:

```text
OpenOCD는 실행됐지만 interface/stlink.cfg 또는 target/stm32g4x.cfg를 못 찾음
```

해결:

`.vscode/launch.json`의 `searchDir`를 OpenOCD scripts 폴더로 맞춘다.

### ST-LINK not found

뜻:

```text
PC가 ST-LINK를 못 찾음
```

확인할 것:

```text
ST-LINK USB driver 설치
USB 케이블
ST-LINK 연결
보드 전원
SWDIO/SWCLK/GND 연결
```
