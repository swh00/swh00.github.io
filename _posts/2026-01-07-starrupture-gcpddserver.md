---
layout: post
title: "GCP 무료 크레딧으로 StarRupture 전용 서버 구축하기 (초보자 가이드)"
date: 2026-01-07 20:00:00 +0900
categories: [개발, 게임]
tags: [StarRupture, GCP, Docker, DedicatedServer, 스타럽]
---

# StarRupture 전용 서버 구축 가이드

**StarRupture(스타럽쳐)**는 팩토리오, 새티스팩토리 같은 공장 건설의 재미에 전투와 탐험이 결합된 게임입니다. 구글 클라우드(GCP) 무료 크레딧을 활용해 24시간 서버를 구축하는 방법을 정리했습니다. 

입문자분들도 아래 순서대로 클릭하고, 코드만 복사해서 붙여넣으면 바로 완성됩니다.

---

## 1. GCP VM 인스턴스 생성 및 네트워크 설정
![초기 화면](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20161133.jpg)
위 화면에서 VM 만들기 클릭

### 인스턴스 사양
![인스턴스 만들기-머신구성1](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20161519.jpg)
이름은 자유, 지역은 서울(자기 사는 곳에서 가까운 곳이 비교적 좋습니다).

![인스턴스 만들기-머신구성2](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20161602.jpg)
밑에 컴퓨터 설정이 있는데, 16gb 권장이라 e2-standard-4, 공장을 많이 지을 거다 하시면 e2-highmem-4를 추천드립니다. (저는 e2-highmem-4를 사용합니다)

![인스턴스 만들기-OS 및 스토리지](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20161704.jpg)
OS는 Ubuntu 22.04 LTS, 꼭 x86을 선택해주세요. 부팅 디스크는 SSD, 60GB로 해야 빠르고 안정적으로 할 수 있습니다.

![인스턴스 만들기-네트워킹](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20203059.jpg)
마지막으로 네트워킹에서 밑으로 내리면 외부 IPv4에서 고정 외부 IP 주소 예약을 선택하고, 아무 이름 입력 후 예약을 누릅니다. 화면 맨 아래 만들기를 누르면 끝! 

정리하자면 다음과 같습니다.
1. **이름**: `starrupture-server` (자유)
2. **지역**: **서울 (asia-northeast3)**
3. **머신 유형**: **e2-highmem-4** (추천) 또는 **e2-standard-4**
4. **OS 및 스토리지**: 
   - 운영체제: **Ubuntu 22.04 LTS (x86/64)**
   - 디스크: **60GB SSD 영구 디스크**
5. **네트워크 인터페이스**: 외부 IPv4 주소를 **고정(프리미엄)**으로 설정

그러면 VM이 생성되는데요, 생성될 동안 저희는 방화벽 설정을 해야 합니다.

![VPN 네트워크-방화벽 규칙1](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20161903.jpg)
![VPN 네트워크-방화벽 규칙2](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20162021.jpg)
왼쪽 상단 툴바를 눌러서 VPC 네트워크 -> 방화벽 -> 방화벽 규칙 만들기 클릭하면 위와 같은 창이 뜹니다. 사진과 같이 설정해주세요.

### 방화벽 설정 요약
1. **대상**: 네트워크의 모든 인스턴스
2. **소스 IPv4 범위**: `0.0.0.0/0`
3. **포트**: TCP와 UDP 모두 체크 후 **7777, 27015** 입력

---

## 2. 서버 구축 자동화 코드 실행 (원샷 스크립트)
이제 거의 다 왔습니다!

![VM 인스턴스 리스트](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20162100.jpg)
다시 VM 인스턴스 창에 돌아가시면 생성된 VM을 볼 수 있습니다. **외부 IP**를 기억해주세요. 맨 오른쪽의 **SSH**를 클릭합니다.

![VM 인스턴스-SSH 권한](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20162110.jpg)
Authorize를 누르면 아래와 같이 서버 컴퓨터에 접속됩니다.

![VM 인스턴스-SSH 접속화면](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20195856.jpg)

![VM 인스턴스-SSH 명령어입력](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20195910.jpg)
중요한 부분입니다! 아래의 **코드들을 차례대로 복사해서 붙여넣으세요.(Ctrl + V 안되면 마우스 우클릭)**

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

sudo rm -rf ~/starrupture-server
mkdir -p ~/starrupture-server/data/server
mkdir -p ~/starrupture-server/data/steamcmd
sudo chown -R 1000:1000 ~/starrupture-server
sudo chmod -R 777 ~/starrupture-server
cd ~/starrupture-server
```

```bash
cat << 'EOF' > Dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN dpkg --add-architecture i386 && apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates curl xvfb wine wine32 wine64 winbind winetricks dbus-x11 \
    lib32gcc-s1 lib32stdc++6 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN useradd -m steam
WORKDIR /home/steam
USER steam

RUN xvfb-run winetricks -q vcrun2015

USER root
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh && chown steam:steam /entrypoint.sh

USER steam
ENTRYPOINT ["/entrypoint.sh"]
EOF

cat << 'EOF' > entrypoint.sh
#!/bin/bash

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/steam/steamcmd

if [ ! -f "/home/steam/steamcmd/steamcmd.sh" ]; then
    mkdir -p /home/steam/steamcmd
    cd /home/steam/steamcmd
    curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
fi

cd /home/steam/steamcmd
chmod +x steamcmd.sh linux32/steamcmd linux32/steamerrorreporter

./steamcmd.sh \
    +@sSteamCmdForcePlatformType windows \
    +force_install_dir /home/steam/starrupture-server \
    +login anonymous \
    +app_update 3809400 validate \
    +quit

rm -f /tmp/.X99-lock
Xvfb :99 -screen 0 1024x768x16 &
export DISPLAY=:99
sleep 5

if [ -d "/home/steam/starrupture-server/StarRupture/Binaries/Win64" ]; then
    cd "/home/steam/starrupture-server/StarRupture/Binaries/Win64"
    exec wine StarRuptureServerEOS-Win64-Shipping.exe \
        -Log -port=7777 -multihome=0.0.0.0 -unattended -NoNativeSnd
else
    exit 1
fi
EOF

cat << 'EOF' > docker-compose.yml
services:
  starrupture:
    build: .
    container_name: starrupture-dedicated
    restart: always
    ports:
      - "7777:7777/udp"
      - "7777:7777/tcp"
    volumes:
      - ./data/server:/home/steam/starrupture-server
      - ./data/steamcmd:/home/steam/steamcmd
    environment:
      - WINEDEBUG=-all
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
EOF
```

```bash
sudo docker compose up --build -d
sudo docker logs -f starrupture-dedicated
```

---

## 3. 게임 접속 및 세션 관리
![SSH-로그 결과](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20195842.jpg)
중간에 위와 같이 나오면 Tab 누르고 Enter 누르시면 됩니다.
![SSH-로그 결과](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20210409.jpg)
대략 10분 뒤, 위 사진처럼 로그가 더 이상 나오지 않고 서버 실행 메시지가 뜨면 성공입니다! (확인 후 `Ctrl + C`로 빠져나오기)

![서버 실행 성공 화면](assets/img/2026post/2026-01-07-starrupture-gcpddserver/화면%20캡처%202026-01-07%20211001.jpg)

* **서버 등록**: 게임 실행 -> 서버 관리 -> VM의 **외부 IP** 입력 후 비밀번호를 설정합니다.
* **세션 생성**: 새로운 세션을 생성하고 잠시 기다리면 게임이 시작됩니다.
* **재접속**: 이후에는 서버 참가 메뉴에서 `당신의 IP:7777`와 비밀번호를 입력해 기존 맵에 참여할 수 있습니다.

---

# StarRupture 서버 시작/종료 명령어
SSH 터미널에서 서버를 시작하거나 종료할 때 사용하는 명령어들입니다.

<details>
<summary>명령어 모음 보기 (클릭)</summary>

### 1. 업데이트 후 재시작

```bash
cd ~/starrupture-server
sudo docker compose up --build -d
```

### 2. 서버 중지

```bash
cd ~/starrupture-server
sudo docker compose down
```

### 3. 실시간 로그 확인 (`Ctrl + C`로 종료)

```bash
sudo docker logs -f starrupture-dedicated
```

</details>

# 유의사항
게임에서 한 번 서버 관리에서 세션 불러오기를 하면, 서버를 껏다 다시 키지 않는 이상 새로운 세션을 만들거나 다른 세션을 선택할 수 없습니다.(제 코드 버그인듯합니당...)

위 방법으로 안된다면, 아래 영상을 보고 참고하시는 것도 좋을 것 같습니다!
![How-to StarRupture Dedicated Server on Ubuntu Linux](https://www.youtube.com/watch?v=mmHwD8R-GeI)

즐거운 스타럽쳐 되세요!
