# <p align="center"> StressTest
### 서비스의 StressTest 진행 프로젝트 

---

<h2 style="font-size: 25px;"> 개발팀원👨‍👨‍👧‍👦<br>
<br>

|<img src="https://avatars.githubusercontent.com/u/127727927?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/98442485?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/66353700?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|
|[@부준혁](https://github.com/BooJunhyuk)|[@신혜원](https://github.com/haewoni)|[@이연희](https://github.com/LeeYeonhee-00)|

---

<br>

## 프로젝트 목적 🌷
기업 IT 서비스의 핵심 고객 만족 요소는 서비스의 신뢰도라고 생각했습니다. <br>
특히, 금융 서비스와 전자상거래 플랫폼에서는 안정적인 인프라를 기반으로 한 연속적인 서비스 제공이 필수적이기 때문입니다. <br>

이에, **고가용성 시스템 설계를 목표**로 , AWS EC2의 우분투 서버에서 부하 테스트를 진행했습니다. <br>

서비스의 부하 테스트에서 성능 문제는 크게 두 가지 원인으로 나눌 수 있습니다 <br>
1. 서비스 자체의 문제 <br>
2. 해당 서비스를 호스팅하는 VM(서버)의 성능 문제 <br>

이번 프로젝트에서는 서비스 자체의 문제보다는, 해당 서비스를 호스팅하는 서버(VM)의 성능 문제를 분석하는 데 중점을 두었습니다. <br>
다양한 부하 상황에서 VM 성능 변화와 자원 사용률을 관찰하고, 서버의 성능 한계를 파악하는 것이 주요 목표였습니다.<br>

<br>

## 테스트 케이스 :star:

다음 세 가지 상태에서 **stress-ng**를 사용하여 VM의 성능을 테스트했습니다 <br>

1. 초기 상태 (빈 인스턴스) <br>
2. Spring 애플리케이션 실행 후 <br>
3. MySQL 데이터베이스 실행 후 <br>

각 상태에서 CPU, I/O, 메모리(VM) 성능을 측정했습니다. <br>

<br>

## 테스트 과정 :mag_right:

1. EC2 인스턴스 준비 (ubuntu 24.04, t2.micro 인스턴스로 테스트 하였습니다.)
2. stress-ng 설치
  ```
  sudo apt update
  sudo apt install stress-ng
  ```
3. stress 테스트 실행
  ```
  stress-ng --cpu 2 --io 1 --vm 1 --vm-bytes 256M --timeout 30s --metrics-brief
  ```
4. Docker 설치
```
sudo apt install docker.io
sudo systemctl start docker
```
5. 서비스 설치 <br>
   5-1. 간단한 Spring 애플리케이션 설치 <br>
   ```
   docker pull yeonheelee/springapp
   docker run -d -p 8999:8999 yeonheelee/springapp
   ```
   5-2. 데이터베이스 서버 설치 <br>
   ```
   docker pull mysql:latest
   docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mypassword mysql:latest
   ```

<br>

## 결과 분석 🌓

### Stress Test 결과
- 빈 인스턴스에서의 stress 테스트
![image](https://github.com/user-attachments/assets/37aee4c4-7bb2-4aeb-8098-ef23583cb4d8)

- 기존에 올렸던 spring app 실행 후 stress 테스트
![image](https://github.com/user-attachments/assets/00e5e6bd-efdd-496c-adb5-1ed5ff922860)

  
- 데이터베이스 서버 stress 테스트
![image](https://github.com/user-attachments/assets/5723d23d-b145-4c3e-a1c1-b52f8caa6a62)


  <br>
  
### 1. CPU 성능 (bogo ops/s)

| 상태 | 성능 | 변화율 |
|------|------|--------|
| 초기 상태 | 543.18 | - |
| Spring 앱 실행 후 | 555.16 | +2.2% |
| MySQL 실행 후 | 208.11 | -61.7% |

- Spring 애플리케이션 실행 후 CPU 성능에 큰 변화가 없었습니다.
- 그러나 MySQL 실행 후 CPU 성능이 약 62% 감소했습니다. 이는 MySQL이 상당한 CPU 리소스를 사용하고 있음을 나타냅니다.


### 2. I/O 성능 (bogo ops/s)

| 상태 | 성능 | 변화율 |
|------|------|--------|
| 초기 상태 | 8445.54 | - |
| Spring 앱 실행 후 | 7306.25 | -13.5% |
| MySQL 실행 후 | 1819.41 | -78.5% |

- Spring 애플리케이션 실행 후 I/O 성능이 약 13% 감소했습니다.
- MySQL 실행 후 I/O 성능이 초기 상태 대비 약 78% 감소했습니다. 이는 MySQL의 디스크 I/O 작업이 상당히 많음을 보여줍니다.
  

### 3. 메모리(VM) 성능 (bogo ops/s)

| 상태 | 성능 | 변화율 |
|------|------|--------|
| 초기 상태 | 10804.15 | - |
| Spring 앱 실행 후 | 10847.75 | +0.4% |
| MySQL 실행 후 | 0.00 | -100% |

- Spring 애플리케이션은 메모리 성능에 거의 영향을 주지 않았습니다.
- 그러나 MySQL 실행 후 메모리 성능이 0으로 떨어졌습니다. 이는 MySQL이 가용 메모리의 대부분을 사용하고 있어 VM 스트레서가 작업을 수행할 수 없었음을 의미합니다.

<br>

## 결론 🌕

1. **Spring 애플리케이션의 영향**
   - CPU와 메모리 성능에 미미한 영향
   - I/O 성능에 약간의 영향 (13.5% 감소)

2. **MySQL의 영향**
   - 모든 측면(CPU, I/O, 메모리)에서 성능을 크게 저하
   - 특히 메모리 성능이 완전히 저하됨 (100% 감소)

3. **리소스 사용**
   - MySQL이 VM의 리소스를 대부분 소비하는 것으로 보임
   - 데이터베이스 작업이 많은 애플리케이션의 경우, 리소스 사용량이 크게 증가할 수 있음

4. **최적화 필요성**
   - MySQL VM에 더 많은 리소스 할당 고려
   - 데이터베이스 서버 분리 고려

<br>

## 느낀점 🤔

테스트 진행을 하며 복잡도가 낮은 간단한 Spring Boot 애플리케이션의 경우, <br> 
성능 비교 수행 후 유의미한 결과를 얻지 못한 것이 아쉬웠습니다. <br><br>
추후 규모가 있는 서비스 구현 시 애플리케이션과 서비스를 모두 고려한 스트레스 테스트를 진행한다면, <br>
안정적이고 신뢰도 있는 서비스를 구현할 수 있을 것이라고 생각했습니다.  <br>
또한, 서비스 자체의 테스트를 위해 nGrinder와 같은 API를 사용해 볼 예정입니다.<br>
