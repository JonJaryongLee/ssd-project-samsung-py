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
2일차 오후 4:30 공개 예정  
