# 🛒 Smart Mart AutoBot Project
> ROS 2 + 비전 AI + 웹 대시보드를 통합한 **스마트 쇼핑카트 및 완전 무인화 마트 시스템**

---

## 📌 Overview

Smart Mart AutoBot은 대형 마트에서 발생하는 **고객의 쇼핑 피로도**, **관리자의 재고 파악 지연**, 그리고 **결제 대기 병목 현상**을 해결하기 위해 기획된 지능형 자동화 시스템입니다.

TurtleBot4 모바일 로봇(AMR) 2대를 각각 **고객 추종용 스마트 카트(AMR 1)**와 **자동 재고 보충 로봇(AMR 2)**으로 분리 운용하며, 딥러닝 비전(YOLOv8)과 웹 기반 실시간 DB 동기화 기술을 결합하여 매장 내 자원 흐름을 최적화했습니다.

---

## 💡 Motivation

* **쇼핑 편의성 극대화:** 무거운 카트를 직접 끌지 않고, 로봇이 안전한 거리를 유지하며 고객을 자율적으로 추종합니다.
* **실시간 오차 없는 구매 처리:** 영상 인식 오류로 인한 오결제를 막기 위해 ROI(관심 구역) 설정 및 60프레임 연속 탐지 알고리즘을 적용했습니다.
* **시스템 최적화 (System Optimization):** 한정된 컴퓨팅 자원 내에서 무거운 비전 연산, ROS 2 주행, Flask 통신이 동시에 충돌 없이 돌아가도록 멀티스레딩 및 상태 머신(State Machine) 아키텍처를 적용했습니다.

---

## 🏗 System Architecture

### 핵심 구성 요소

| 구성 요소 | 역할 |
| :--- | :--- |
| **ROS 2 Humble** | 노드 간 비동기 메시지 통신 (로봇 이동, 상태 전송) |
| **AMR 1 (Smart Cart)** | OAK-D RGB-D 센서 + YOLOv8을 활용한 타겟 추종 (P 제어 적용) |
| **AMR 2 (Restock Robot)** | Nav2 웨이포인트 주행 및 LiDAR(0.5m 이내) 기반 긴급 제동 |
| **Flask + SocketIO** | 고객 및 관리자 웹 대시보드, 실시간 영상/데이터 양방향 스트리밍 |
| **SQLite DB** | 유저 정보, 장바구니, 매대/창고 재고 상태 실시간 트랜잭션 관리 |

---

## 🔄 System Flow

### 1. 고객 추종 및 쇼핑 (AMR 1)
1. 고객이 입구 구역(Entrance Zone) 진입 시 비전 센서가 감지 (`customer position` 발행).
2. AMR 1이 도킹 해제 후 고객을 찾아 일정한 거리(0.65m)를 유지하며 자율 추종 모드(`FOLLOW`) 돌입.
3. 카메라 ROI 내에 상품이 인식(60프레임 유지)되면 고객 대시보드에 구매 팝업 생성.

### 2. 결제 및 예산 관리
1. 고객 구매 승인 시 DB 매대 재고 즉시 차감 및 장바구니 동기화.
2. 누적 금액이 설정된 예산을 초과하면 로봇에서 경고 알람(`beep_node`) 발생.
3. 고객이 계산대 구역(Counter Zone) 도착 시 결제 진행 및 로봇 귀환(`finish_shopping`).

### 3. 지능형 재고 보충 (AMR 2)
1. 매대 재고가 0이 되면 관리자 대시보드로 알람 전송 (`out_of_stock`).
2. 관리자 승인 시 AMR 2가 창고에서 해당 상품의 웨이포인트로 정밀 이동.
3. 보충 작업 완료 후 자동으로 충전 스테이션 복귀 및 도킹.

---

## 🛠 Tech Stack

* **Environment:** Ubuntu 22.04 LTS, ROS 2 Humble
* **Robotics:** TurtleBot4 (AMR), Nav2 (Navigation Stack)
* **AI & Vision:** YOLOv8, OpenCV, CvBridge, OAK-D Camera, 2D Web Cam
* **Backend / DB:** Python 3.10, Flask, Flask-SocketIO, SQLite3
* **Frontend:** HTML/CSS, JavaScript

---

## 📂 Project Structure

```text
mart_autobot/
├── amr1.py                   # 고객 추종 및 상태 머신 제어 (RGB-D + P 제어)
├── AMR2_control.py           # Nav2 기반 상품별 웨이포인트 주행 및 충돌 방지
├── beep_node.py              # 예산 초과 시 로봇 오디오 알람 발생
├── person_detection.py       # YOLOv8 기반 구역(입구, 계산대) 내 고객 감지
├── customer_monitor.py       # 고객용 웹 서버, 실시간 비전 상품 탐지 및 장바구니 DB 관리
└── admin_monitor_topic.py    # 관리자용 웹 서버, 재고 모니터링 및 로봇 출동 승인
