Promtail + Loki + Grafana 
===

promtail + Loki + Grafana 연동 데모입니다.

<br>

AWS 환경
---
> **Note** : private subnet에서 동작하는 promtail이 public subnet에 있는 Loki에 로그를 보내기 위해서 NAT Gateway가 필요합니다.

![image](https://github.com/Chaerim1001/loki_grafana_demo/assets/89819254/5af7c0bd-0f19-4a41-99be-c86e129bda36)

- `public subnet`에 위치한 인스턴스에는 **Loki**와 **Grafana** 컨테이너가 동작합니다.
- `private subnet`에 위치한 인스턴스에는 로그가 쌓이는 **application**과 **promtail** 컨테이너가 동작합니다.

<br>

인스턴스 보안그룹 설정
---
(1) `public subnet`에 위치한 인스턴스의 인바운드 규칙 편집이 필요합니다.
- `Grafana` 대시보드 접속을 위해 **3000** 포트를 열어주세요.
- `Loki` 사용을 위해 `private subnet`의 **IPv4 CIDR**에 대해 **3100** 포트를 열어주세요. 


<br>

코드 설정  
---
> **Note** : 실행하려는 EC2 인스턴스에 docker-compose 설치가 완료되어 있어야 합니다.

### 1. application + promtail
(1) `private subnet`의 인스턴스에 접속하여 `server_promtail` 디렉토리의 내용을 복사해 주세요.

(2) `/docker-compose.yml`의 내용을 수정해 주세요.

```yml
version: "3"

services:
  application:
    image: ${YOUR_APPLICATION_IMAGE} # 원하는 이미지로 수정해 주세요.
    container_name: application
    labels:
      logging: "promtail"
      logging_jobname: "containerlogs"
    restart: always
    ports:
      - 80: ${APPLICATION_PORT}  # 원하는 포트 번호로 수정해 주세요.
      - 443: ${APPLICATION_PORT}  # 원하는 포트 번호로 수정해 주세요.
    networks:
      - app

# promtail 설정은 수정하지 않아도 됩니다.
```

(3) `/config/promtail.yaml` 파일을 수정해 주세요.

```yml
clients:
  - url: http://${PUBLIC_INSTANCE_PRIVATE_IP}:3100/loki/api/v1/push # EC2 인스턴스의 private_ip 주소로 수정해 주세요.
```

(4) docker-compose를 실행시켜 주세요.
```bash
$ docker-compose up -d
```

### 2. loki + grafana
(1) `public subnet`의 인스턴스에 접속하여 `loki_grafana` 디렉토리의 내용을 복사해 주세요.

(2) docker-compose를 실행시켜 주세요.
```bash
$ docker-compose up -d
```

<br>

동작 확인
---
(1) public subnet에 있는 인스턴스에 할당받은 public ip를 통해 Grafana 대시보드에 접속합니다.
```
http://${PUBLIC_IP}:3000
```
![image](https://github.com/Chaerim1001/loki_grafana_demo/assets/89819254/aeaf0676-a9fa-4eda-853b-4fbb99451407)


(2) **username**과 **password**를 입력하여 접속합니다.
- username: `admin`
- password: `admin`


(3) **Explore** 메뉴에 들어가 `label`과 `value` 선택 후 `Query`를 실행시킨다. (여러 컨테이너를 띄울 경우 `value` 선택에서 `docker-compose.yml`에 설정한 `container_name`으로 구분 가능합니다.)

- label: `container`
- value: `application`


![image](https://github.com/Chaerim1001/loki_grafana_demo/assets/89819254/a36f5e75-016b-45d0-a463-4ae0ddc9668e)


