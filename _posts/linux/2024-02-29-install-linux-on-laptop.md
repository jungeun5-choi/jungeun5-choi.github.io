---
layout: single
title: "대여용 LG 노트북에 Rocky Linux 9 설치하는 글"
categories: [Linux]
tag: [Linux, grub]
published: false
---

## 리눅스를 설치하고 싶다...

국내 대기업 노트북에서는 BIOS 자체에 제한을 걸어 OS 설치가 어려운 경우가 많습니다. 그래도 대부분은 부팅 USB를 만든 다음, BIOS에서 부팅 우선순위를 USB로 변경하는 방식으로 쉽게 해결이 됩니다.

저는 특수한 상황이라 위의 순서로 해결되지 않았습니다. 노트북이 부팅 USB를 인식하지 못해서 설치 자체를 진행할 수 없었는데, 실습에 사용할 노트북은 대여용이라 BIOS 제한정도가 굉장히 높았습니다. BIOS 옵션에서 다른 무언가를 더 시도해볼 수가 없었습니다.

저와 같은 특수한 상황의 분들께 도움이 되었으면 해서 글로 정리해 보았습니다.

<br>

## 준비물
1. 설치할 Linux OS 선정: 저는 Rocky Linux 9을 설치했습니다.
2. USB <u>2개</u>: 개당 용량은 Minimal 기준으로 4GB~8GB정도면 충분합니다. 반드시 2개여야 합니다.
3. 설치할 노트북: 저는 대여용으로 제작된 LG 울트라 시리즈 노트북을 사용했습니다.


## 부팅 USB 생성
포맷 타입 FAT32, NTFS 각 한 개씩 생성합니다.

### BIOS 설정에서 부팅 순서 변경


## grub 화면 진입
grub에 진입은 했지만, usb로부터 부팅 정보가 담긴 config 파일을 찾지 못해 bootloader를 실행하지 못하고 있는 상황입니다.

그래서 저희가 config 파일을 찾아주어야 합니다.

### `ls` 명령어 
이 때 `ls` 명령어를 사용합니다.

### `set` 명령어

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