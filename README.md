# 택배 수호자 (Parcel Guardian)
 
> YOLO 객체 인식과 STM32 이중 MCU 구조를 결합한 실시간 택배 도난 감지 보안 시스템
 
[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/335b5d65c68c80128f18c7779229e943?source=copy_link)
 
---
 
## 개요
 
| 항목 | 내용 |
|------|------|
| 플랫폼 | STM32F411RETx × 2, Python (Linux) |
| 언어 | C, Python |
| 도구 | STM32CubeIDE, VSCode, Roboflow |
| 통신 | UART (115200 / 9600 bps), SPI, Bluetooth |
| 개발 기간 | 2025.03.30 |
| 팀 구성 | 4인 |
 
## 주요 기능
 
- **이중 YOLO 인식** — Camera 1 (박스) / Camera 2 (사람) 모델과 카메라를 분리하여 오인식 방지
- **RFID 잠금 인증** — RC522 SPI 직접 제어, non-blocking FSM으로 카드키 UID 인증
- **다중 UART 통신** — PC ↔ 중계 MCU ↔ 제어 MCU, DMA Circular 모드 기반 비동기 직렬 통신
- **이벤트 자동 녹화** — 도난 의심 시 pre-buffer 포함 양방향 카메라 동시 녹화 저장

## 파일 구성
 
| 파일 | 역할 |
|------|------|
| `main.py` | YOLO 인식 + 이벤트 판단 + Serial 통신 통합 |
| `relay_mcu/` | USART2 ↔ USART6 브리지, DMA Circular 수신 |
| `control_mcu/` | RFID 인증(SPI1), USART1 이벤트 수신, 도난 판단 |
 
## 주요 구현
 
- **커스텀 YOLO 학습** — 택배 박스 이미지 1,500장 직접 수집·라벨링 후 모델 학습
- **DMA Circular 비동기 수신** — 중계 MCU에서 UART 수신을 DMA로 처리해 CPU 점유 없이 데이터 중계
- **non-blocking RFID FSM** — SPI 폴링 없이 상태 전이 기반으로 RFID 인증과 UART 처리를 병행
- **pre-buffer 녹화** — 이벤트 발생 전후 영상을 메모리 버퍼에 유지하다 트리거 시 자동 저장

## 담당 역할
 
- YOLO 커스텀 모델 학습 및 실시간 박스 인식 구현
- 이벤트 기반 이중 카메라 녹화 시스템 구현
- Python ↔ STM32 양방향 Serial 통신 설계 및 구현
