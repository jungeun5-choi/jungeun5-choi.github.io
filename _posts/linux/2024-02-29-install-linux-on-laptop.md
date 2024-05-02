---
layout: single
title: "대여용 LG 노트북에 Rocky Linux 9 설치하는 글"
categories: [Linux]
tag: [Linux, grub]
published: false
---

## 리눅스를 설치하고 싶다...

국내 대기업 노트북에서는 BIOS 자체에 제한을 걸어 OS 설치가 어려운 경우가 있다. 대부분은 부팅 USB를 만든 다음, BIOS에서 부팅 우선순위를 USB로 변경하는 방식으로 쉽게 해결이 된다. 하지만 나는 특수한 상황이라 위의 순서로 해결되지 않았다.

대여용 노트북의 BIOS 설정 제한 때문에 펌웨어를 Legacy로 전환할 수 없었고, UEFI(유이파이)만 지원하는 상황이었다. UEFI는 부팅 시 FAT32 포맷을 인식할 수 없었고, 부팅 USB가 생성될 때 지정된 부트로더는 GRUB이라 NTFS를 인식할 수 없었다. 이 때문에 설치부터 난항을 겪었다.

<br>

## 준비물
1. <b>설치할 Linux OS 선정:</b> 이 글에서는 Rocky Linux 9 사용
2. <b>USB <u>2개</u>:</b> 개당 용량은 Minimal 기준으로 4GB~8GB정도면 충분 (개당 3~5천원 선)
3. <b>설치할 노트북:</b> 이 글에서는 대여용으로 제작된 LG 울트라 시리즈 노트북 사용
4. <b>디스크 파티션 작업:</b> 이 글에서는 노트북을 단독으로 리눅스 머신으로 사용할 예정이었기 때문에 생략


## 부팅 USB 생성
먼저, 포맷 타입을 각각 <u>FAT32</u>, <u>NTFS</u>로 지정하여 한 개씩 생성한다. 이 때, 프로그램은 가장 대중적인 [Rufus](https://rufus.ie/ko/)를 사용했다.

파티션 구성은 설치에 큰 영향을 미치지 않았고, 대상 시스템(Target System)은 UEFI로 지정해준다.

### BIOS 설정에서 부팅 순서 변경
NTFS 포맷의 USB를 꽂고 노트북을 부팅 혹은 재부팅한다.

윈도우가 켜지기 전, [F2] 키를 눌러 BIOS 설정 화면으로 진입한다. (대여 노트북의 경우, 비밀번호가 걸려있다면 담당자 분께 연락하여 비밀번호를 요청해야한다. 대여용 노트북을 뜯을 순 없으니까...)

부팅 순서를 Disk가 아니라 USB로 설정한다.

## GRUB 화면 진입
부팅 순서를 바꾼 후, 재부팅한다. 이 때 FAT32 포맷의 USB도 꽂아주어 부팅할 때 인식되도록 한다. 그리고 재부팅하면 GRUB 화면으로 진입한게 보이지만, 아무리 기다려도 부팅은 되지 않는다.

GRUB은 부팅을 돕는 부트로더로 멀티부팅을 지원하여 현재 널리 쓰이고 있는 부트로더 중 하나이다. 하지만 GRUB은 NTFS 포맷의 부팅 USB를 인식하지 못한다...

그래서 GRUB 화면에 진입은 했지만, 부팅 USB로부터 부팅 정보가 담긴 config 파일을 찾지 못해서 부팅을 실행하다가 막힌 것이다.

GRUB은 USB를 꽂은 순서대로 인식하기 때문에, NTFS를 먼저 인식했을 것이다. 후발주자로 꽂은 FAT32에서 부팅에 사용할 config 파일을 찾아주기만 하면 된다.

부팅 파일은 주로 `/boot` 디렉터리 하위에 있다.

### `ls` 명령어 
이 때 `ls` 명령어를 사용한다.

최초로 `ls` 명령어를 사용하면 아래와 같은 결과를 볼 수 있다.
```
grub> ls
(hd0) (hd0,msdos1) (hd0,msdos2) (hd1,msdos1) (hd1,msdos2)
```
`(hd0)`이 붙은 디렉터리는 가장 먼저 인식된 NTFS 포맷에 대한 디렉터리이므로, 아마 여기에는 부팅 파일이 없을 것이다. `(hd1)`부터 찾아본다.

나의 경우는 `(hd1,msdos1)`에 부팅에 필요한 config 파일인 `grub.cfg`이 있었다.
```
grub> ls (hd1,msdos1)/efi/boot/
./ ../ grub.cfg
```

### `set` 명령어
찾았다면 `set` 명령어를 사용해 부팅 파일 경로를 가리키는 항목들을 수정해준다.



```shell
grub> set
# ... 생략 ...
fw_path=(hd0,msdos1)/boot/
prefix=(hd0,msdos1)/boot/
root=hd0,msdos1
# ... 생략 ...
```

### `insmod normal`






```
<LG 울트라에 Rocky Linux 9 설치>
1. 부팅 USB 총 두 개 필요
 포맷 타입 FAT32, NTFS 각 한 개씩
2. grub 화면으로 넘어가는 거 확인
3. ls 명령어로 인식된 하드디스크 확인
4. ls (hd1,msdos1)/ 이런 식으로 하드디스크 내부에 파일이 있는지 확인 (이동 x)
 아예 없으면 error 문구를 반환
 특히 /efi나 /Boot가 있어야한다 (대소문자 구분 안 함)
5. 최종 경로에 grub.cfg 파일이 있는 것을 확인
6. set 명령어를 실행해 fw_path, prefix, root 항목 확인 (* 그냥 set만 입력하면 됨)
7. fw_path, prefix, root 항목 수정
    - 예시 (저는 이 경로에  grub.cfg 파일이 있었습니다)
     > set prefix=(hd1,msdos1)/efi/boot/
     > set fw_path=(hd1,msdos1)/efi/boot/
     > set root=hd1,msdos1    
8. insmod normal 명령어 실행
     > insmod normal
     > normal
9. bootloader 화면이 뜨면 완료
```