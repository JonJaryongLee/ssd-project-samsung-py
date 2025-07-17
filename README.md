# SSD 프로젝트

코치 이름: 이자룡  

## 프로젝트 개요

SSD 검증 애플리케이션   
- SSD 는 실제 하드웨어가 아닌 SW 로 구현한다.  
- Test Shell 프로그램을 제작하여 SSD 동작을 테스트할 수 있다.  
- 다양한 Test Script 를 작성한다.  

## 프로젝트 구성
1. SSD (ssd.py)
   - 하드웨어 대신, 소프트웨어로 구현
2. Test Shell (shell.py)
   - SSD 를 테스트하는 프로그램

### SSD (ssd.py)
1. 0 부터 99 까지 `ssd_nand.txt` 에 저장 가능  
2. 맨 처음 `ssd_nand.txt` 는 0 부터 99 까지 모두 0x00000000 이 기록되어있음  
3. 각 인덱스 (LBA) 엔 16진수가 저장됨 (0x00000000 - 0xFFFFFFFF)
4. R (READ) / W (WRITE) 명령 수행 후 프로그램 즉시 종료  
5. R 이든 W 이든, `ssd_nand.txt` 전체 데이터를 일단 읽어온다.  
6. R 일 경우: `ssd_output.txt` 에 값 기록
7. W 일 경우: `ssd_nand.txt` 에 "해당 부분을 변경한 새로운 전체 값" 기록 후 `ssd_output.txt` 는 빈 파일로 변경.  
8. 즉, 내부에서 입출력하지 않고, 순수하게 파일로만 입출력한다.  
9. 잘못된 동작일 경우 (인덱스 범위를 벗어나는 등), `ssd_output.txt` 에는 `ERROR` 가 기록된다.  
10. 3 ~ 4 일차에 R / W 이외의 기능이 추가될 수 있다.  


**동작 예시**  

```powershell
python .\ssd.py W 3 0x1298CDEF
```
- `ssd_nand.txt` 3번 인덱스 (LBA) 에 0x1298CDEF 저장  
- `ssd_output.txt` 는 빈 파일로 변경  

```powershell
python .\ssd.py R 3
```
- `ssd_nand.txt` 의 3번 인덱스 (LBA) 에 기록된 값을 가져옴  
- 해당 값을 `ssd_output.txt` 에 기록  


**`ssd_output.txt` 는 항상 가장 마지막 실행한 결과만 한 줄로 가지고 있어야 함**  

### Test Shell (shell.py)
- SSD 를 테스트할 수 있는 프로그램  
- 검증팀에서 `ssd.py` 를 직접 실행해서 R/W 하지 않고, `shell.py` 을 사용해 각종 테스트 진행한다고 가정  

- 사용 가능 명령어 종류
  - read
  - write
  - exit : 프로그램 종료
  - help : 프로그램 사용법
    - 제작자 명시 (팀장/팀원)
    - 각 명령어마다 사용법 기입
  - fullwrite : 전체 인덱스 (LBA) write
  - fullread : 전체 인덱스 (LBA) read
  - Test Script 실행 명령 : Test Script 부분에서 상세 설명  
  - 없는 명령어 : INVALID COMMAND 출력  

- 편의를 위해 `ssd.py` 에 fullwrite 와 fullread 를 구현하면 안된다.  

**동작 예시**

```powershell
python .\shell.py

Shell> write 3 0xAAAABBBB
[Write] Done

Shell> read 0
[Read] LBA 00 : 0x00000000

Shell> read 3
[Read] LBA 03 : 0xAAAABBBB

Shell> read 999
[Read] ERROR

Shell> fullwrite 0xFFFFFFFF
[Full Write] Done

Shell> fullread
[Full Read]
LBA 00 : 0xFFFFFFFF
LBA 01 : 0xFFFFFFFF
...
LBA 99 : 0xFFFFFFFF
```

### Test Script
총 세 가지 테스트 실행 명령어를 Test Shell 에 명령어로 추가한다.  
- 테스트가 성공했다면 PASS 출력  
- 테스트가 진행 도중 실패했다면 즉시 FAIL 출력  

1. `1_FullWriteAndReadCompare`
   - `1_` 라고만 입력해도 실행 가능  
   - 0 ~ 4번 LBA까지 다섯개의 동일한 랜덤 값으로 write 명령어 수행  
   - 0 ~ 4번 LBA까지 실제 저장된 값과 맞는지 확인  
   - 5 ~ 9번 LBA까지 다섯개의 동일하지만 0 ~ 4번과 다른 랜덤값으로 write 명령어 수행  
   - 5 ~ 9번 LBA까지 실제 저장된 값과 맞는지 확인  
   - 위와 같은 규칙으로 전체 영역에 대해 반복  

2. `2_PartialLBAWrite`
   - `2_` 라고만 입력해도 실행 가능
   - 30회 반복  
     - 4번 LBA에 랜덤값을 적는다.  
     - 0번 LBA에 같은 값을 적는다.  
     - 3번 LBA에 같은 값을 적는다.  
     - 1번 LBA에 같은 값을 적는다.  
     - 2번 LBA에 같은 값을 적는다.  
     - LBA 0 ~ 4번 모두 같은지 확인  

3. `3_WriteReadAging`
    - `3_` 라고만 입력해도 실행 가능  
    - 200회 반복  
      - 0번 LBA에 랜덤 값을 적는다.  
      - 99번 LBA에 같은 값을 적는다.  
      - LBA 0번과 99번이 같은지 확인  

## 기타 안내사항

### 팀 관련
1. 팀 이름 설정  
2. 팀에서 사용할 규칙 정하기 (컨벤션, 그라운드룰, PR, Merge, Commit 룰 등...)
3. 팀 역할 분배
4. 팀장은 추가 점수 부여  
5. 팀장은 Best 팀원 2명 선택 가능하며, Best 팀원은 추가 점수 부여 (팀장이 자기 자신 선택하면 안됨)  
6. 팀장을 제외한 코드리뷰어를 1명 이상 지정 필수  
   - 팀원들이 개발한 코드는 코드리뷰어가 리뷰  
   - 코드리뷰어가 개발한 코드는 팀장이 리뷰  
   - 시간 간격을 협의하셔서, 정해진 시간별로 코드리뷰 담당자를 변경

### 깃 관련
우선 리포지토리 만들고 Discord 의 github_url 채널에 링크부터 공유 해주세요.  

### 테스트 관련
- 1 ~ 2 일차에서 mock 을 한번 이상 사용
   - 구현한 SSD 성능이 느리기에 mock 을 사용해 해결 가능
- 1 ~ 2 일차에선 TDD 필수, 3 ~ 4 일차에선 필수 아님
   - 3 ~ 4 일차는 Unit Test + 리팩토링만 요구됨

### 기능 추가 관련
- 3 ~ 4 일차엔 기능 추가 요청 예정
- 기능이 추가된다 가정하고 개발할 것

### 발표 관련
1. 반드시 팀장이 하지 않아도 됨  
2. 발표는 리팩토링 before / after 가 중심이 되므로, 리팩토링 과정마다 commit 하여 발표할 자료를 미리미리 만들어 둘 것  
3. 따라서, 처음엔 리팩토링을 신경 쓰지 않고 개발하고, 리팩토링을 해나가는 과정을 모아두는것이 유리함 (개발과 리팩토링을 동시에 하지 말 것)  
4. 디자인 패턴 적용 시 추가점수 부여되니, 해당 부분을 발표자료에 포함할 것  
   - Singleton, Command Pattern ...  

### PPT 관련
- 발표 PPT 제목에 팀 이름과 조 번호를 적어주세요.  
- 템플릿은 Discord 교안공유 채널에 `발표양식.pptx` 로 올려두었습니다.  
- 필수로 들어가야 하는 내용
   - 조원 소개 및 역할
   - 기능 구현 소개
   - TDD 활용 예시
   - Mocking 활용 예시
   - 리팩토링을 통한 클린코드 전후 결과 비교
   - 소감
---


## 3 ~ 4 일차 안내

### Erase
삭제 기능 추가  

1. SSD
   -  `python .\ssd.py E [LBA] [SIZE]`
      -  ex) `python .\ssd.py E 0 10` - 0번을 포함해, 10 사이즈만큼 삭제 (9까지)
   -  실제론 삭제가 아니라, 0x00000000 이 된다.
   -  **1 <= SIZE <= 10**
   -  예외 처리
      -  사용법에 올바르지 않는 명령 사용 (파라미터 갯수, 타입 등)
      -  LBA 가 0 ~ 99 범위에서 벗어나거나, SIZE 가 1 미만 10 초과 
      -  SIZE 만큼 더했을 때 LBA 가 99 를 초과

2. Test Shell
   -  `Shell> erase [LBA] [SIZE]`
      -  ex) `Shell> erase 0 10` - 0번을 포함해, 10 사이즈만큼 삭제 (9까지)
      -  **단! SSD 애플리케이션과는 다르게 SIZE 의 범위는 10 초과일 수 있다.**
         -  ex) `Shell> erase 0 50` - 0번을 포함해, 50 사이즈만큼 삭제
      -  예외 처리
         -  사용법에 올바르지 않는 명령 사용 (파라미터 갯수, 타입 등)
         -  LBA 가 0 ~ 99 범위에서 벗어나거나, SIZE 가 1 미만 100 초과  
         -  SIZE 만큼 더했을 때 LBA 가 99 를 초과
            -  ex) `Shell> erase 90 20`
   -  `Shell> erase_range [ST_LBA] [EN_LBA]`
         -  ex1) `Shell> erase_range 20 22` - 20번부터 22번까지 삭제
         -  ex2) `Shell> erase_range 0 99` - 0번부터 99번 LBA 까지 전체 삭제
      -  예외 처리
         -  사용법에 올바르지 않는 명령 사용 (파라미터 갯수, 타입 등)
         -  LBA 가 0 ~ 99 범위에서 벗어남
         -  erase_range 에서 [ST_LBA] 가 [EN_LBA] 보다 클 때
            -  ex) `Shell> erase_range 99 0`
   - `Shell> 4_EraseAndWriteAging`
     - `4_` 라고만 입력해도 실행 가능
     - 0 ~ 2번 LBA 삭제
     - Loop 30회
       - 2번 LBA 랜덤 값
       - 2번 LBA 다른 랜덤 값
       - 2 ~ 4번 LBA 삭제
       - 4번 LBA 랜덤 값
       - 4번 LBA 다른 랜덤 값
       - 4 ~ 6번 LBA 삭제
       - 6번 LBA 랜덤 값
       - 6번 LBA 다른 랜덤 값
       - 6 ~ 8번 LBA 삭제
       - ...
       - 96 ~ 98번 LBA 삭제
       - 98번 LBA 랜덤 값
       - 98번 LBA 다른 랜덤 값
       - 98 ~ **99**번 LBA 삭제 (인덱스 주의)

### Logger
Test Shell 에 Logger 클래스 추가  


1. 로그 파일은 다음 형태로 기록된다. (로그 포맷 반드시 맞출 것)  

```
[YY.MM.DD HH:mm] ClassName.methodName()         : log message.
```

예시는 다음과 같다.  
```
[25.07.08 17:04] Sender.sendMessageWithData()   : send and wait sync message.
[25.07.08 17:04] Message.checkMessage()         : FAIL - Response message didn’t arrive.
[25.07.08 17:04] Shell.release()                : Bye bye.
```

2. 다음 메서드를 사용하면 로그가 현재 날짜 / 시간 기준 로그 파일에 기록된다.  
`logger.print("ClassName.methodName()", "log message.")`  
즉, Logger 클래스를 하나 만들어야 한다.  

3. 로그 파일 이름은 다음 규칙을 따른다.  
   - 최초 로그 파일 이름: `latest.log`  
   - `latest.log` 파일의 용량이 10kb (약 영문 10,000 자) 를 초과하면, 파일 이름을 `until_250708_17h_12m_11s.log` 형태로 변경
   - 새로운 `latest.log` 파일에 로그 기록
   - `until_` 파일이 두 개가 되면, 둘 중 더 오래된 로그의 확장자를 `.log` 에서 `.zip` 으로 변경 (실제 압축하는 과정은 없지만, 압축을 가정)

예시는 다음과 같다.  
- 최초 : `latest.log`
- 25년 7월 08일 17시 12분 52초에 `latest.log` 10kb 초과: 파일 이름 `until_250708_17h_12m_52s.log` 로 변경 후, 새로운 `latest.log` 생성
- 25년 7월 10일 09시 00분 00초에 `latest.log` 10kb 초과
  - 파일 이름 `until_250710_09h_00m_00s.log` 로 변경 후, 새로운 `latest.log` 생성
  - `until_250708_17h_12m_52s.log` 의 확장자를 `.zip` 으로 변경 => `until_250708_17h_12m_52s.zip`

### Runner
테스트 셸을 실행하는 방법은 다음과 같았다.  

```powershell
python .\shell.py
```

옵션을 추가하겠다.  

```powershell
python .\shell.py .\path\to\shell_script.txt
```

`shell_script.txt` 엔 다음 내용이 적혀있다.  

```
1_FullWriteAndReadCompare
2_PartialLBAWrite
3_WriteReadAging
4_EraseAndWriteAging
```

즉, 지금까지 Test Shell 에 직접 `1_` , `2_` 라고만 실행했던 것을, `shell_script.txt` 에 적힌 목록대로 실행하는 기능을 만들어보자.  


출력은 다음과 같다.  

```
1_FullWriteAndReadCompare  ___   Run...Pass
2_PartialLBAWrite          ___   Run...Pass
3_WriteReadAging           ___   Run...FAIL!
```

- Run... 이 먼저 출력되고, 해당 테스트가 끝날때까지 기다린 후, Pass / FAIL! 을 옆에 출력한다.  
- 위 경우, `3_` 실행 중 실패한 경우이며, `4_` 를 실행하지 않고 즉시 중단한다.  



### Command Buffer
SSD 애플리케이션의 성능을 올리기 위해, 무조건 명령어를 실행하지 않고, buffer를 활용해 효율적으로 명령어를 수행해보자.  

- 실제로 프로젝트를 진행하다보면, 이 기능을 구현하면 오히려 실행 속도가 저하되는 현상을 볼 수 있다. `ssd_nand.txt` 라는 파일이 SSD 를 대신하고, 다섯개의 buffer는 다섯개의 파일로 관리되기 때문이다.  
- 우리 목적은, `ssd_nand.txt` 가 실제 SSD 라고 가정할 때, 해당 파일의 변경을 최소화하는 것이다. 실제 SSD 는 자주 변경할수록 수명이 떨어지고 성능이 저하되기 때문이다.  


`.\buffer` 폴더에 다음 다섯개 파일을 생성한다.  
`1_empty` , `2_empty` , `3_empty` , `4_empty` , `5_empty`  

- 다섯 개 파일은 확장자가 없다.  
- 다섯 개 파일의 내용은 존재하지 않는다. 오로지 파일 이름만 활용한다.  


**예시1**  
비효율적인 버퍼 관리를 먼저 보자.  
다음 다섯개 파일이 존재한다면?  
`1_W_20_0xABCDABCD` , `2_W_20_0x12341234` , `3_E_20_1` , `4_empty` , `5_empty`  

아래 명령이 buffer에 기록되어 있는 것  
1. 20번 LBA 에 0xABCDABCD 기록
2. 20번 LBA 에 0x12341234 기록
3. 20번 LBA 부터 사이즈 1만큼 삭제 (즉, 20번만 삭제)

생각해보면, 맨 마지막것만 실행하면 된다. 왜냐면 어차피 E 명령어로 삭제될 것이기 때문이다.  

따라서, **명령어가 들어오는 순서대로 buffer를 단순히 기록하면 안된다.**  

1. 최초: `1_empty` , `2_empty` , `3_empty` , `4_empty` , `5_empty`  
2. 1번 buffer 기록: `1_W_20_0xABCDABCD` , `2_empty` , `3_empty` , `4_empty` , `5_empty`  
3. 2번 buffer 기록: `1_W_20_0xABCDABCD` , `2_W_20_0x12341234` , `3_empty` , `4_empty` , `5_empty`  
   
이것은 비효율적이다. 2번 buffer의 명령에 의해, 1번 buffer는 아무 의미 없는 명령이 되었다. 따라서, 순서 3번은 다음과 같이 되어야 한다.  

   3. 2번 buffer 기록: `1_W_20_0x12341234` , `2_empty` , `3_empty` , `4_empty` , `5_empty`  

그리고 `E_20_1` 명령이 들어올 차례가 되었는데, 마찬가지로 `1_W_20_0x12341234` 는 효율을 위해서 삭제되어야 한다.  

최종적으로, 다음과 같은 다섯 개 파일이 존재한다.  
`1_E_20_1` , `2_empty` , `3_empty` , `4_empty` , `5_empty`  


**예시2**  
다음 다섯개 파일이 존재한다.  
`1_W_50_0xAAAABBBB` , `2_W_20_0xABABCCCC` , `3_empty` , `4_empty` , `5_empty`

이 상황에서, 다음 명령어가 들어온다 가정하자.  

```powershell
python .\ssd.py R 50
```

그럼 과연 평소대로 `ssd_nand.txt` 파일을 실행해서, 50번에 있는 값을 찾아서, `ssd_output.txt` 에 해당 값을 기록하는것이 효율적일까?  
그럴 필요 없다. 왜냐면 buffer 에 의하면 50번 LBA 엔 0xAAAABBBB 가 기록된 것이 확실하기 때문에, 그 값을 가져와서 `ssd_output.txt` 에 기록하면 된다.  
즉, **가능한 buffer에 먼저 접근** 하고, `ssd_nand.txt` 에 대한 접근을 최소화해야만 한다.  


**예시3**  
다음 다섯개 파일이 존재한다.  
`1_W_20_0xABCDABCD` , `2_E_10_4` , `3_empty` , `4_empty` , `5_empty`  


이 상황에서, 다음 명령이 3번 buffer에 들어올 차례가 되었다.  
`3_E_12_3`  


buffer 의 목적은 **명령어를 줄이는 것** 이기 때문에, 3번 buffer 에 넣는 대신 2번 buffer 와 3번 buffer 를 **새로운 명령어로** 통합할수도 있다.  
`1_W_20_0xABCDABCD` , `2_E_10_5` , `3_empty` , `4_empty` , `5_empty`  

- 단, E 명령의 size 는 10 을 넘을 수 없음에 유의하자.  


**Flush 기능 추가**  
SSD 애플리케이션과 Test Shell 애플리케이션에 하나의 기능 (F / flush) 을 더 추가한다.  

F 는 Flush 기능이며, buffer 에 있는 **모든 명령어를 수행**하고, buffer 전체를 비우는 역할을 한다.  
- 비운다? 각각의 파일명을 `1_empty` , `2_empty` ... `5_empty` 로 변경하는것을 뜻한다.    

- 사용법
  - SSD 애플리케이션
    - `python .\ssd.py F`
  - Test Shell 애플리케이션
    - `Shell> flush`
  
Q. 잠깐, Flush 가 실행되기 전엔 `ssd_nand.txt` 에 실제 기록되지 않는 것이 아닌가? 그렇다면 이제 프로그램은 Flush 를 실행해야만 변경이 반영되는 것인가?  
A. 실제론 그렇다. 그러나, 다음 상황을 생각해보자.

**ssd_nand.txt**  
```
00  0x99998888
01  0xAAAABBBB
02  0xFFFFFFFF
...
99 0x00000000
```

**buffer files**
```
1_W_01_0x12341234 2_empty  3_empty  4_empty  5_empty
```

위 상황을 잘 보면, 실제 `ssd_nand.txt` 의 01 에 기록된 값은 0xAAAABBBB 이지만, 결국 프로그램은 buffer를 먼저 참고할 것이기 때문에 0x12341234 가 저장되어 있다고 **간주할 수 있다.** 이는 Flush 를 통해 buffer가 비워지고 최종 반영될 뿐이다.  


**세부사항**  
- buffer 파일은 항상 5개다.  
- Read 는 Buffer 에 기록할 필요가 없다. `ssd_nand.txt` 를 변경하지 않는 작업이기 때문이다.  
- 다섯 개 buffer 가 꽉 찬 상태에서 새로운 명령어가 들어오면, Flush 명령어 입력할 필요 없이 자동으로 flush 수행 후, 새로운 명령어를 buffer 에 채운다.  
- 목적을 항상 생각하자. buffer의 존재 이유는 `ssd_nand.txt` 의 쓰기 접근을 최소화하는 것이다. 즉, W / E 명령을 최대한 줄이는 방법을 생각해야 한다. 명령어가 늘어나면 오히려 잘못된 것이다.  
- buffer 는 모든 요구사항 중 가장 어려운 부분이다. 효율적으로 buffer를 관리하는 알고리즘을 각 팀별로 생각해내야만 한다. 최대한 다양한 상황을 고려해보자.  

---
# 발표 안내사항
- ~ 14:30
  - 강사 PC 에 PPT 다운로드
  - 깃 마지막 커밋 (PPT 파일도 깃 리포지토리에 첨부)
  - Discord 시연 세팅 (음성채널 화면공유)
  - 각 팀장은 Best 팀원 두 명 강사에게 알려주기
- 14:40 ~ 15:00   : A팀
- 15:00 ~ 15:20   : C팀
- 15:20 ~ 15:35   : 15분 휴식
- 15:35 ~ 15:55   : D팀
- 15:55 ~ 16:15   : B팀
- 16:15 ~ 16:30   : 15분 휴식
- 16:30 ~         : 사내 현업활동 가이드