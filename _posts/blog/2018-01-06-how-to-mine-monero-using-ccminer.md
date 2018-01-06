---
layout: post
title: "How to mine monero using ccminer"
date: 2018-01-06 00:00:00
categories: blog
author: juner
tags: [cryptocurrency, monero, mining]
share: true
comments: true
---

# 마이닝 풀 허브에서 모네로 채굴
 - 잠자는 그래픽 카드여 아오지로 가즈아!!!

## 먼저 출처를 밝힙니다...
 - [마이닝풀허브에서 모네로 채굴](http://www.cryptocoin.kr/entry/%EB%A7%88%EC%9D%B4%EB%8B%9D%ED%92%80%ED%97%88%EB%B8%8C%EC%97%90%EC%84%9C-%EB%AA%A8%EB%84%A4%EB%A1%9C-%EC%B1%84%EA%B5%B4-Monero-mining-by-ccminer)
 - [모네로 채굴하는 방법](http://lifeinmd.tistory.com/214)
 - [모네로 채굴 풀 비교 사이트](http://www.cryptocoin.kr/entry/%EB%AA%A8%EB%84%A4%EB%A1%9C-%EC%B1%84%EA%B5%B4%ED%92%80-%EB%B9%84%EA%B5%90-%EB%B6%84%EC%84%9D-%EC%82%AC%EC%9D%B4%ED%8A%B8-moneropoolscom)
 - [모네로 ccminer 기준 애프터버너설정 정보](https://www.ddengle.com/miningbitcoin/2865333)

## 배경
 - 이더리움(ETH)을 채굴하다가 채산성이 너무 안나와서
 - cpu로도 채굴할수 있다는 모네로(XMR)는 채산성이 좋지 않을까?라는 막연한 기대심리
 - 채굴 풀은 어디로 할까 고민을 해보았는데, 다른데 가입하기도 귀찮고 이미 가입돼있는 한글이라 편한 마이닝풀허브([miningpoolhub](https://miningpoolhub.com/))를 이용하기로 함



## 사전에 알아두면 좋은 내용
 - 마이닝풀 : 리소스(GPU 혹은 그외...)가 작은 장비로 채굴을 해봐야 불가능하니, 작은 리소스들을 워커로 등록 후 리소스를 모아서 채굴을 하고 리워드를 나눠주는 리소스풀(국내 풀중 유명한 곳이 마이닝풀허브)
 - 해시파워(H/s) : 리소스의 해시연산 속도, 마이닝풀허브에서는 워커로 등록한 장비의 해시, 풀의 해시, 전체 블럭체인 네트워크 해시를 대시보드로 보여줌
 - 모네로(XMR) : 블럭체인을 기반으로 한 암호화 화폐, GPU, CPU로 채굴이 가능하고, 암호화 알고리즘은 cryptonight를 이용한다. 전자지갑주소까지 암호화하여 절대 추적이 불가능하다고 함
 - 애프터버너 : Nvidia 계열 그래픽카드 overclock 유틸로, 마이닝알고리즘에 맞게 그래픽카드 성능을 조절가능(power limit, temp limit, core clock, momory clock등) 상태 모니터링도 가능
 - ccminer : 모네로 마이닝 유틸, 상세한 사용설명 [ccminer 설명](http://www.cryptocoin.kr/entry/ccminer-v22-%EC%B6%9C%EC%8B%9C)



### 약간 디테일한 풀 선택의 기준
 - 굳이 마이닝풀허브로 갈 필요는 없음(장점: 한글설명, 해시파워 높은편, 이미가입함, 단점 : 모네로 전송 수수료 높음, 마이너가 많아서 할당량이 적음)
 - 해쉬파워가 높고 마이너가 많은 곳을 갈것인지, 해쉬파워가 낮지만 마이너가 적어서 할당량이 많은 곳을 갈것인지는 당신의 선택
 - 하지만 땡글등 다양한 정보를 확인해 본 결과, 잠깐 잠깐 마이닝 할것 아니면 결국 다 비슷한 채굴량을 보여준다고 함
 - 모네로 풀을 비교해 주는 사이트([moneropools.com](http://moneropools.com))가 있으니 들어가서 자신에게 채굴형태에 맞는 풀을 선택
 - 접속이 안정적이고, 수수료 적고, 최소 인출액이 낮은 풀을 찾는 것이 좋음



### 약간 디테일한 애프터버너(그래픽카드 OC 유틸) 설정
 - 모네로는 마이닝시 메모리 오버로 채산성이 영향을 받는다고 함([땡글참조](https://www.ddengle.com/miningbitcoin/2865333))
 - 내pc 기준(gigabyte gtx 1060 6GB) power limit 80% / core clock -60 MHz / Memory 700MHz 로 지정
 - cpu를 언더로 한 이유는 순전히 전기세가 걱정이 돼서 그렇게 함, 온도도 좀 낮게가기 위함도 있고

### 내 pc / 그래픽카드 스펙
 - Windows 10 x64 
 - Nvidia geforce GTX 1060 6GB x 1
 
## Steps! 
### 1. 필요한 유틸 다운 및 설치
 - [ccminer](https://github.com/tpruvot/ccminer/releases) : 내 그래픽 카드는 CUDA 8.0 이라 V2.2를 받음
 -- 설치는 별다른거 없고 zip 파일만 풀어주면 끝
 - [afterburner](https://www.msi.com/page/afterburner)
 -- zip 파일 풀고, setup파일 실행
 
### 2. 자신이 원하는 풀에 가입하기(마이닝풀허브)
 - 가입(메일 인증이 필요함)
 - 가입 후 자신의 계정으로 로그인>정보수정>OTP활성화 후 google OTP등록을 해야 함   
 ![signup]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/signup_MPH.PNG)   

### 3. 모네로 마이너 추가 
 - 왼쪽 메뉴에 모네로풀로 가서 monero 풀 클릭 > 마이너 클릭   
 ![MPH]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/MPH_monero.png)     

 - 마이너 추가 메뉴에서 마이너 이름, 암호 입력 후 마이너 추가(입력한 마이너 정보가 ccminer 실행 스크립트에 들어감, 마이너 비밀번호는 딱히 중요하지 않으므로 x)  
 - 모니터 항목 on 으로 변경 후 마이너 업데이트 클릭
 - monero 풀 대시보드로 이동(아직 마이너 실행을 안했으므로 속도도 0이고 적립도 0이다. 화면은 내가 나중에 캡쳐해서 숫자가 나와있음)    
 ![woker]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/miner_applying.PNG)      


### 4. afterburner 설정
 - step1에서 설치한 afterburner 실행 후 위에서 말한 대로 설정해 줌(gigabyte gtx 1060 6GB) power limit 80% / core clock -60 MHz / Memory 700MHz 로 지정)  
 ![setting afterburner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/afterburner_setting.PNG)  

### 5. ccminer실행
 - step1에서 압축 푼 ccminer 위치로 이동   
 ![run miner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/before_running_ccminer.PNG)
   
 - notepad로 start.bat 파일을 ccminer-x64 바이너리가 있는 곳에 만들어 아래의 내용을 입력 후 저장한다.(상세 옵션은 README 파일 참조) 
 ```
ccminer-x64 -a cryptonight -o stratum+tcp://asia.cryptonight-hub.miningpoolhub.com:20580 -u 풀계정.마이너이름 -p x
pause
 ```
 - 저장한 start.bat를 실행한다.   
 ![run miner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/running_ccminer.PNG)      
 - 마이닝 시작! - 약 5분뒤 monero 풀 대쉬보드로 이동하면 해시파워와 적립된 코인을 볼수 있다.(대략 511H/s가 나오는데 이더보다 낮아서 이상하다 생각했지만, 채산성은 조금 낫더라...)  
 ![dashboard]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/running_ccminer.PNG)    
 ![dashboard]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-monero-using-ccminer/after_running_ccminer.PNG)
     
## 결론
 - ccminer상에는 대략 66wh라고 나오는데 믿을수가 없으므로 계량기 확인 필요
 - 장비 : gtx 1060 6GB  
 - 애프터 버너 설정 : power limit 80% / core clock -60 MHz / Memory 700MHz
 - 1일 적립 코인 : 약 0.004 XMR / 한달 적립 코인 : 0.12 XMR (이더의 경우 약 0.003ETH/day 그러나 그래픽 카드의 온도가 더 높았음, 비슷한 설정) 
 - 빗썸기준 519,200원(2018/01/04 23시 38분) : 0.12 x 519,200 = 62,304원
 - 결국 전기료 떄고 뭐 떄고 하면... 이더가 낫겠다... 가격도 오르고있고...
 - 끝!
