---
layout: post
title: "Quick guide for sed on linux"
date: 2011-05-30 19:00:00
categories: blog
author: juner
tags: [linux]
comments: true
---

[This post is moved from old blog](http://blog.naver.com/juner84/100129369041)
# Quick guide for sed on linux

## linux sed 사용법
<br>
sed로 텍스트 편집하기
<p>sed (stream editor) : 그래픽 인터페이스가 없는 명령행 유틸리티임 그래서 많은 편집 작업을 일괄적! 으로 처리하기에 아주! 적합한 도구이다.
<p>sed는 파일을 한행씩 처리하고 현재 작업중인 라인을 패턴스페이스란 곳에 저장함
</br>

## 사용 문법
```bash
sed 'command' [filename]

sed s/regular_expression/replacement_String/flags input_file
```

### ex)
```bash
sed s/file_nmae/file_name/ > filename.txt
```

## sed 명령어들
- d : 행 삭제
- h : 패턴스페이스 내용을 홀드스페이스로 복사
- H : 패턴스페이스 내용을 홀드스파이스로 추가
- g : 홀드스페이스 내용을 패턴스페이스에 복사
- G : 홀드스페이스 내용을 패턴스페이스에 추가
- p : 행을 출력함
- n : 다음입력
- q : sed 종료
- r : 파일로 부터 행을 읽어 온다
- ! : 선택된 행을 제외한 나머지 전체 행에 명령어를 적용
- s : 문자열을 치환함

### s - 검색 & 치환
s는 검색과 치환을 수행한다는 뜻이다. 슬래시(/)로 검색하고 치환할 정규 표현식을 지정한다.
```bash
ex) sed s/filename/filename.txt/ filename.txt
#filename.txt에 있는 filename이란 스트링을 filename.txt로 치환한다.  
```
### \ -역슬래시
역슬래시(\\)는 이스케이프 문자라고 부르는데 (\\) 다음에 오는 문자는 정규표현식으로 해석하지 않는다.  
결국 스트링에 특수기호가 포함되어 있을때 \를 사용한다.
```bash
ex) sed s/\$FL/\$FILELIST/ filename.txt
```

### 한행에 여러번 나오는 문자열 교체하기 (flag : g)
sed는 행단위 편집기로, 행을 한번에 하나씩 메모리로 읽은 후 한단위로 처리한다.

sed를 실행할 때는 이 사실을 명심해야한다. 모든 명령행 옵션도 이러한 설계 철학에 기반을 두기 때문이다.

기본적으로 행마다 sed 명령을 새롭게 적용한다고 이해하면 되겠다
그래서 한줄에 같은 단어가 2개 이상 있다면 하나만 바뀌고 끝난다.

 - filename : prj.txt
 - project : aaa.prj, project : aaa
 - project : ok

위에 내용에서 project를 project_name로 바꾸고 싶다면
```bash
sed s/project/project_name/ prj.txt
```

그러나 두번째 둘의 콤마(,)뒤의 project는 바뀌지 않은것을 볼수 있을 것이다.

그래서 다음과 같이 g flag(global)를 이용한다.
```
sed s/project/project_name/g prj.txt
```

### 선행검색
s앞에 /string/을 사용하면 선행검색이 가능하다

```bash
ex) sed /okplayers/s/players :/artist :/ playerlists.txt
# okplayers가 있는 줄을 찾아 players :를 artist :로 바꾸어라
```

### 콜론(:)으로 끝나는 문자열 모두 변경하기
정규표현식을 이용한 sed 활용법

```bash
filename:$FLN
system "echo project:$project"
system "echo version:$version"
```

위의 문자에서 :으로 끝나는 문자열을 모두 변경하여 보자!!!
```bash
sed s/[a-z]*:/value:/g system.txt
```

변경된 내용은 다음과 같다
```bash
value:$FLN
system "echo value:$project"
system "echo value:$version"
```

그러나!! 이것보다 더 좋은 것이 있었으니,

일정 형식의 string에 앞에 어떤 단어를 추가 하고 싶을 때

기존의 filename:, project:, version: 앞에 new_를 추가 하고 싶으면 앰퍼센트를(&) 사용하면된다.

### 앰퍼센트(&)
위의 내용에서 각각의 콜론 앞 스트링에 new_를 붙이고 싶다면
```bash
filename:$FLN
system "echo project:$project"
system "echo version:$version"
```

```bash
sed s/[a-z]*:/new_\&/g system.txt
```

```bash
new_filename:$FLN
system "echo new_project:$project"
system "echo new_version:$version" 이렇게 됨 ㅎㅎ
```


<출처>

[원본 Link](http://blog.naver.com/PostView.nhn?blogId=heewon0117&logNo=120088332695&viewDate=&currentPage=1&listtype=0)
