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

수정 대상은 아래 3개다.

```text
CMakePresets.json
cmake\gcc-arm-none-eabi.cmake
.vscode\launch.json
```

### 6-1. CMakePresets.json 수정

아래 항목에서 GCC 경로와 임시 폴더 경로를 수정한다.

```json
"PATH": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin;$penv{PATH}",
"TEMP": "D:/maze/StmCubeMx/tmp",
"TMP": "D:/maze/StmCubeMx/tmp"
```

그리고 아래 컴파일러 경로도 수정한다.

```json
"CMAKE_C_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_ASM_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gcc.exe",
"CMAKE_CXX_COMPILER": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-g++.exe"
```

`TEMP`, `TMP`는 GCC가 임시 파일을 만드는 위치다.

한글 사용자 이름 경로에서 오류가 나면 아래처럼 영문 경로의 임시 폴더를 만들어서 사용한다.

```text
D:\maze\StmCubeMx\tmp
```

### 6-2. cmake/gcc-arm-none-eabi.cmake 수정

아래 줄을 본인 GCC 경로로 수정한다.

```cmake
set(TOOLCHAIN_PATH "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin")
```

이 파일은 CMake에게 실제 ARM용 GCC가 어디 있는지 알려주는 파일이다.

### 6-3. .vscode/launch.json 수정

아래 항목을 수정한다.

```json
"executable": "${workspaceFolder}/build/Debug/my_project.elf",
"serverpath": "D:/maze/StmCubeMx/openocd/.../bin/openocd.exe",
"gdbPath": "D:/maze/StmCubeMx/stm32tools/14.3.1+st.2/bin/arm-none-eabi-gdb.exe",
"searchDir": [
    "D:/maze/StmCubeMx/openocd/.../openocd/scripts"
]
```

`executable`의 `.elf` 이름은 새 프로젝트 이름과 맞아야 한다.

예를 들어 프로젝트 이름이 `my_project`이면 아래처럼 쓴다.

```json
"executable": "${workspaceFolder}/build/Debug/my_project.elf"
```

OpenOCD가 아래 구조라면:

```text
D:\maze\StmCubeMx\openocd\xpack-openocd-0.12.0-7
```

`launch.json`은 이런 식으로 맞춘다.

```json
"serverpath": "D:/maze/StmCubeMx/openocd/xpack-openocd-0.12.0-7/bin/openocd.exe",
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

---

## 12. 템플릿 zip 안에 들어있는 파일

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

이 파일들은 새 프로젝트의 소스코드가 아니라, VSCode가 STM32 프로젝트를 빌드하고 디버그할 수 있게 해주는 설정 파일이다.

