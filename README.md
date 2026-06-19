# 택배 수호자 (Parcel Guardian)
 
> YOLO 객체 인식과 STM32 이중 MCU 구조를 결합한 실시간 택배 도난 감지 보안 시스템
<!--
[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/335b5d65c68c80128f18c7779229e943?source=copy_link)
-->
[![YouTube](https://img.shields.io/badge/YOUTUBE-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%EC%98%81%EC%83%81%20%EB%B3%B4%EA%B8%B0-555555?style=for-the-badge&logo=youtube&logoColor=white&labelColor=FF0000)](https://youtu.be/IT9fTA3r7G0?si=RIphoJKs5mb-5Ph8)


## 1. Overview
 
| 항목 | 내용 |
|------|------|
| 플랫폼 | STM32F411RETx × 2, Python (Linux) |
| 언어 | C, Python |
| 도구 | STM32CubeIDE, VSCode, Roboflow |
| 통신 | UART (115200 / 9600 bps), SPI, Bluetooth |
| 개발 기간 | 2025.03 |
| 팀 구성 | 4인 팀 프로젝트 |
 
---

## 2. 주요 기능
 
- **도난 상황 인지 및 오인식 방지**: 박스 전용 카메라와 상시 감시 카메라를 독립적으로 운용하여 외부 환경으로 인한 오작동 최소화
- **실시간 도난 감지 및 증거 확보**: 무단 접근 및 도난 의심 상황 발생 시 자동으로 블랙박스 영상을 녹화하고 저장
- **안전한 사용자 인증**: 등록된 RFID 카드키를 통해 정당한 사용자를 판별하고 잠금 장치 해제 및 보안 모드 제어
- **유기적인 시스템 연동**: 감시 장치(PC)와 하드웨어 제어부(MCU) 간의 실시간 데이터 연동으로 공백 없는 모니터링 제공

 
## 3. 담당 역할
**AI 객체 인식 결과를 활용한 영상 처리 및 임베디드 - PC 간 데이터 통신 시스템 구축**
- **Pre-buffer 기반 이중 카메라 녹화 시스템 개발 (Python)**  
  OpenCV 비디오 스트림을 링 버퍼 형태로 메모리에 상시 유지하여, 이벤트 발생 전, 후 5초 영상을 유실 없이 병합 저장하는 알고리즘 구현
  
- **Python ↔ STM32 양방향 Serial 통신 설계 및 구현 (이벤트 코드 0~3 / 트리거 4)**
  인식 결과에 따른 이벤트 코드(0~3) 및 제어 트리거(4) 정의, 예외 처리를 포함한 신뢰성 높은 직렬 통신 파이프라인 구축

 
## 4. 시스템 아키텍처 및 핵심 구현
 
### 핵심 구현

![block_diagram](SecuritySytem_Blockdiagram.png)
 
| 구현 | 설명 |
|---|---|
| **① Pre-buffer 기반 이벤트 녹화** | 이벤트 발생 이전 영상을 메모리 버퍼에 상시 유지하다가 도난 의심 트리거 시 전후 영상을 합산 저장 → 이벤트 전 상황까지 증거로 확보 |
| **② DMA Circular 비동기 수신** | 중계 MCU에서 UART 수신을 DMA Circular 모드로 처리해 CPU 점유 없이 PC ↔ 제어 MCU 간 데이터를 실시간 중계 |
| **③ Non-blocking RFID FSM** | SPI 폴링 방식 대신 상태 전이 기반 구조로 RFID 인증과 UART 처리를 병행 동작 |

 
## 5. 트러블슈팅
 
| 발생 문제 | 발생 원인 | 해결 방안 | 결과 |
|-----------|-----------|-----------|------|
| 조명·배경 변화에 따른 인식 불안정 | OpenCV 색상·윤곽선 기반 인식의 환경 의존성 | YOLO 기반 딥러닝 모델로 전환 | 다양한 환경에서 안정적 인식 확보 |
| 카메라 1대 운용 시 박스 오인식 | 대규모 사전학습 모델이 커스텀 모델 출력을 덮어씀 | 카메라 2대로 분리하여 모델 독립 운용 | 인식 충돌 제거 |
| STM 송신값이 Python에서 미수신 | moserial과 Python이 동일 Serial 포트 동시 점유 | 포트를 하나의 프로그램만 점유하도록 수정 | 안정적 양방향 통신 확보 |
 
 
## 6. 디렉토리 구조 (Directory Structure)

| 파일 | 역할 |
|------|------|
| `main.py` | YOLO 인식 + 이벤트 판단 + Serial 통신 통합 |
| `relay_mcu/` | USART2 ↔ USART6 브리지, DMA Circular 수신 |
| `control_mcu/` | RFID 인증(SPI1), USART1 이벤트 수신, 도난 판단 |
