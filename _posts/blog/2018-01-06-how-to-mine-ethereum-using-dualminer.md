---
layout: post
title: "How to mine ethereum on mining pool hub"
date: 2018-01-06 02:00:00
categories: blog
author: juner
tags: [cryptocurrency, ethereum, mining]
share: true
comments: true
---

# 마이닝 풀 허브에서 이더리움 채굴
 - 잠자는 그래픽 카드여 아오지로 가즈아!!!

## 먼저 출처를 밝힙니다...
 - [마이닝풀허브에서 이더리움/시아코인 채굴(클레이모어 듀얼 이더리움 마이너)](https://coinkr.kr/bbs/board.php?bo_table=coin_mining&wr_id=69)
 - [이더리움 단독채굴(클레이모어 듀얼 이더리움 마이너)](http://www.cryptocoin.kr/entry/%ED%81%B4%EB%A0%88%EC%9D%B4%EB%AA%A8%EC%96%B4-%EC%9D%B4%EB%8D%94%EB%A6%AC%EC%9B%80-%EC%8B%B1%EA%B8%80-%EC%B1%84%EA%B5%B4-%ED%95%98%EA%B8%B0-%EB%A7%88%EC%9D%B4%EB%8B%9D%ED%92%80-%ED%97%88%EB%B8%8C?category=687108)
 - [이더리움 애프터버너 세팅](https://www.ddengle.com/board_FAQ/2717858)  

## 배경
 - 블럭체인 공부 중에 이더리움 풀노드에 관한 호기심이 생김 
 - asic 방어를 한다는 점에서 이더리움이 더욱 매력적임
 - 이더리움 네트워크에 기여하고 싶어 풀 노드를 구성하고 싶었으나 현실은 만만치 않았다...
 - 용돈 벌이도 좀 할겸, 집에 놀고 있는 gtx 1060도 있겠다 마이닝이나 해볼까? 라는 생각이 문뜩 들었다. 
 - 채굴 풀은 어디로 할까 고민을 해보았는데, 한글이라 편한 마이닝풀허브([miningpoolhub](https://miningpoolhub.com/))를 이용하기로 함
 - 이더리움은 이제 작업증명을 pow->pos로 넘어가려고 노력하는 시기(메트로폴리스)라서 pow도 한번 맛좀 보자라고 생각 



## 사전에 알아두면 좋은 내용
 - 마이닝풀 : 리소스(GPU 혹은 그외...)가 작은 장비로 채굴을 해봐야 불가능하니, 작은 리소스들을 워커로 등록 후 리소스를 모아서 채굴을 하고 리워드를 나눠주는 리소스풀(국내 풀중 유명한 곳이 마이닝풀허브)
 - 해시파워(H/s) : 리소스의 해시연산 속도, 마이닝풀허브에서는 워커로 등록한 장비의 해시, 풀의 해시, 전체 블럭체인 네트워크 해시를 대시보드로 보여줌
 - 이더리움(ETH) : 블럭체인을 기반으로 한 암호화 화폐, GPU 채굴이 가능하고, ASIC저항이 있어 GPU가 효율이 좋다. 암호화 알고리즘은 ethash를 이용한다. 튜링완전 언어로 프로그래밍 하여 스마트컨트렉트를 작성할 수 있다. 
 - 애프터버너 : Nvidia 계열 그래픽카드 overclock 유틸로, 마이닝알고리즘에 맞게 그래픽카드 성능을 조절가능(power limit, temp limit, core clock, momory clock등) 상태 모니터링도 가능
 - 클레이모어 듀얼 이더리움 마이너 : 이더리움 마이닝 유틸, [상세한 사용설명](https://www.ddengle.com/miningbitcoin_voted/3097132) 


### 약간 디테일한 풀 선택의 기준
 - 굳이 마이닝풀허브로 갈 필요는 없음, 하지만 이더는 마풀허가 좋다. (장점: 한글설명, 해시파워 높은편, 이미가입함, 단점 : 딱히 이더는 단점이 없는 것 같다)
 - 해쉬파워가 높고 마이너가 많은 곳을 갈것인지, 해쉬파워가 낮지만 마이너가 적어서 할당량이 많은 곳을 갈것인지는 당신의 선택



### 약간 디테일한 애프터버너(그래픽카드 OC 유틸) 설정
 - 이더리움은 마이닝시 메모리 오버로 채산성이 영향을 받는다고 함([땡글참조](https://www.ddengle.com/miningbitcoin/2865333))
 - 내pc 기준(gigabyte gtx 1060 6GB) power limit 80% / core clock -100 MHz / Memory 700MHz 로 지정
 - cpu를 언더로 한 이유는 순전히 전기세가 걱정이 돼서 그렇게 함, 온도도 좀 낮게가기 위함도 있고

### 내 pc / 그래픽카드 스펙
 - Windows 10 x64 
 - Nvidia geforce GTX 1060 6GB x 1
 
## Steps! 
### 1. 필요한 유틸 다운 및 설치
 - [Claymore dual miner](https://bitcointalk.org/index.php?topic=1433925.0) : 10.3 버전이 최신이라 받음
 -- 설치는 별다른거 없고 zip 파일만 풀어주면 끝  
 - [afterburner](https://www.msi.com/page/afterburner)
 -- zip 파일 풀고, setup파일 실행
 
### 2. 자신이 원하는 풀에 가입하기(마이닝풀허브)
 - 가입(메일 인증이 필요함)
 - 가입 후 자신의 계정으로 로그인>정보수정>OTP활성화 후 google OTP등록을 해야 함     
 ![signup]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/signup_MPH.PNG)    


### 3. 이더리움 마이너 추가 
 - 왼쪽 메뉴에 이더리움풀로 가서 ethereum 풀 클릭 > 마이너 클릭    
 ![MPH]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_pool.png)      
 - ethereum 풀 메뉴 아래 마이너 클릭 후
 - 마이너 추가 메뉴에서 마이너 이름, 암호 입력 후 마이너 추가(입력한 마이너 정보가 ccminer 실행 스크립트에 들어감, 마이너 비밀번호는 딱히 중요하지 않으므로 x)  
 - 모니터 항목 on 으로 변경 후 마이너 업데이트 클릭
 - ethereum 풀 대시보드로 이동(아직 마이너 실행을 안했으므로 속도도 0이고 적립도 0이다. 화면은 내가 나중에 캡쳐해서 숫자가 나와있음)      
 ![woker]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_miner_setting.PNG)    


### 4. afterburner 설정
 - step1에서 설치한 afterburner 실행 후 위에서 말한 대로 설정해 줌(gigabyte gtx 1060 6GB) power limit 80% / core clock -100 MHz / Memory 700MHz 로 지정)    
 ![setting afterburner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_afterburner_setting.PNG)

### 5. Claymore dual miner실행
 - step1에서 압축 푼 ccminer 위치로 이동     
 ![run miner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_start_bat.PNG)
   
 - notepad로 start.bat 파일을 ccminer-x64 바이너리가 있는 곳에 만들어 아래의 내용을 입력 후 저장한다.(상세 옵션은 README 파일 참조) 
 ```
 # single mining(only ethereum)
EthDcrMiner64.exe -allpools 1 -epool asia1.ethereum.miningpoolhub.com:20535 -ewal 풀계정.마이너이름 -epsw x -esm 2
pause
 ```
 ```
 # dual mining(ethereum, siacoin)
EthDcrMiner64.exe -epool asia.ethash-hub.miningpoolhub.com:20535 -ewal 풀계정.마이너이름 -eworker 풀계정.마이너이름 -esm 2 -epsw x -dpool stratum+tcp://hub.miningpoolhub.com:20550 -dwal 풀계정.마이너이름 -dpsw x -dcoin sc
 ```
 - 저장한 start.bat를 실행한다.     
 ![run miner]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_run_bat.PNG)    
 - 마이닝 시작! - 약 5분뒤 ethereum 풀 대쉬보드로 이동하면 해시파워와 적립된 코인을 볼수 있다.(대략 23Mh/s정도가 나왔음)    
 ![dashboard]({{ site.url }}/assets/post_images/2018-01-06-how-to-mine-ethereum-using-dualminer/ether_minpoolhub_dash.PNG)


## 결론
 - 듀얼마이너는 ccminer와 다르게 전력 사용량이 안나온다. 계량기 확인 꼭 필요!! 
 - 듀얼로 돌릴경우 시아코인이 생겨서 기분은 좋지만 그래픽카드 온도가 75도까지 올라감
 - 약 23Mh/s의 해쉬 파워가 나오고 5일 채굴해보니 일 평균 0.003ETH가 생긴다. 
- 장비 : gtx 1060 6GB  
 - 애프터 버너 설정 : power limit 80% / core clock -100 MHz / Memory 700MHz
 - 1일 적립 코인 : 약 0.003 ETH / 한달 적립 코인 : 0.09  
 - 빗썸기준 약 1,000,000원(2018/01/02 23시 38분) : 0.09 x 1,000,000 = 9,000원 이나... 요며칠 사이 엄청 올라서
 - 빗썸기준 1,489,500(2018/01/06) : 0.09 x 1,489,500 = 134,055 
 - 게다가 시아코인까지 캐면 전기료 보단 많이 나올듯...
 - 하지만, 지난 6개월간 이더리움 난위도가 많이 올라서 점점 채산성을 떨어질거라 예상됨
 - 궁금한거 다 풀렸다. 끝!
