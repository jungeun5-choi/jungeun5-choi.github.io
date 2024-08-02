---
layout: single
title: "대여용 (대기업)노트북에 리눅스를 설치하는 여정"
categories: [Linux]
tag: [Linux, grub]
---

## 0. 리눅스를 설치하고 싶다...

국내 대기업 노트북에서는 BIOS 자체에 제한을 걸어 OS 설치가 어려운 경우가 있다. 대부분은 부팅 USB를 만든 다음, BIOS에서 부팅 우선순위를 USB로 변경하는 방식으로 쉽게 해결이 된다. 하지만 나는 특수한 상황이라 위의 순서로 해결되지 않았다.

대여용 노트북의 BIOS 설정 제한 때문에 펌웨어를 Legacy로 전환할 수 없었고, UEFI(유이파이)만 지원하는 상황이었다. UEFI는 부팅 시 FAT32 포맷을 인식할 수 없었고, 부팅 USB가 생성될 때 지정된 부트로더는 GRUB이라 NTFS를 인식할 수 없었다. 이 때문에 설치부터 난항을 겪었다.

<br>

## 1. 준비물
1. <b>설치할 Linux OS 선정:</b> 이 글에서는 Rocky Linux 9 사용
2. <b>USB <u>2개</u>:</b> 개당 용량은 Minimal 기준으로 4GB~8GB정도면 충분 (개당 3~5천원 선)
3. <b>설치할 노트북:</b> 이 글에서는 대여용으로 제작된 LG 울트라 시리즈 노트북 사용
4. <b>디스크 파티션 작업:</b> 이 글에서는 노트북을 단독으로 리눅스 머신으로 사용할 예정이었기 때문에 생략


## 2. 부팅 USB 생성
먼저, 포맷 타입을 각각 <u>FAT32</u>, <u>NTFS</u>로 지정하여 한 개씩 생성한다. 이 때, 프로그램은 가장 대중적인 [Rufus](https://rufus.ie/ko/)를 사용했다.

파티션 구성은 설치에 큰 영향을 미치지 않았고, 대상 시스템(Target System)은 UEFI로 지정해준다.

## 3. BIOS 설정에서 부팅 순서 변경
'NTFS' 포맷의 USB를 꽂고 노트북을 부팅 혹은 재부팅한다. 윈도우가 켜지기 전에 [F2] 키를 눌러 BIOS 설정 화면으로 진입한다. <span style='color:gray'>(대여 노트북의 경우, 비밀번호가 걸려있다면 담당자 분께 연락하여 비밀번호를 요청해야한다. 대여용 노트북을 뜯을 순 없으니까...)</span>

그런 다음, 부팅 순서를 Disk가 아니라 'NTFS' 타입의 USB로 설정한다. 부팅 순서를 바꿨다면 재부팅한다. 이 때, 'FAT32' 포맷의 USB를 꽂아주면 재부팅 시 자동으로 인식되기 때문에 편리하다.

## 4. GRUB 화면 진입
재부팅하면 GRUB 화면으로 진입한게 보이지만, 아무리 기다려도 부팅은 되지 않는다.

**GRUB**은 부팅을 돕는 부트로더로 멀티부팅을 지원하여 현재 널리 쓰이고 있는 부트로더 중 하나이다. **하지만 GRUB은 NTFS 포맷의 부팅 USB를 인식하지 못한다....** 즉, GRUB이 부팅 USB로부터 부팅 정보가 담긴 config 파일을 찾지 못해서 부팅을 실패했다는 뜻이된다.

## 5. GRUB에서 config 파일 찾기

부팅 파일은 주로 `/boot` 디렉터리 하위에 있다.

### (1) `ls` 명령어 
파일을 탐색할 때는 GRUB에서도 `ls` 명령어를 사용한다. 최초로 `ls` 명령어를 사용하면 아래와 같은 결과를 볼 수 있다.
```shell
grub> ls
(hd0) (hd0,msdos1) (hd0,msdos2) (hd1,msdos1) (hd1,msdos2)
```

`(hd0)`이 붙은 디렉터리는 가장 먼저 인식시킨 NTFS 포맷 USB에 대한 디렉터리이므로, 아마 여기에는 부팅 파일이 없을 것이다. `(hd1)`부터 찾아본다. <span style='color:gray'>(* GRUB은 USB를 꽂은 순서대로 인식한다.)</span>

```shell
# 이런 식으로 단계적으로 탐색해보면 된다...
grub> ls (hd1,msdos1)/
grub> ls (hd1,msdos1)/efi/
```


나의 경우는 `(hd1,msdos1)` 디렉터리 하위에 부팅에 필요한 config 파일인 `grub.cfg`가 있었다.
```shell
grub> ls (hd1,msdos1)/efi/boot/
./ ../ grub.cfg
```
(* 참고로 이 파일이 아닌 경우도 있으니, 부팅되지 않는다면 다른 파일을 찾아 다시 시도해보자.)

### (2) `set` 명령어
config 파일을 찾았다면, `set` 명령어를 사용해 부팅 파일 경로를 가리키는 항목들을 수정(갱신)해준다. 수정이 필요한 항목은
- `fw_path`
- `prefix`
- `root`

이 세 가지이다.

최초로 `set` 명령어만 입력하면 아래처럼 GRUB 기본값을 확인할 수 있다.
```shell
grub> set
# ... 생략 ...
fw_path=(hd0,msdos1)/boot/
prefix=(hd0,msdos1)/boot/
root=hd0,msdos1
# ... 생략 ...
```

부팅 파일 경로 갱신은 아래처럼 할 수 있다.
```shell
grub> set prefix=(hd1,msdos1)/efi/boot/
grub> set fw_path=(hd1,msdos1)/efi/boot/
grub> set root=hd1,msdos1
```

### (3) `insmod normal`
`insmod normal` 명령어는 일반적으로 커널 이미지를 로드할 때 사용하는 명령어이다.
- **`insmod`:** 'insert module'의 줄임말로, GRUB 환경에서 모듈을 메모리에 로드하는 명령어.
- **`normal`:** 'normal'은 미리 정의된 모듈의 이름인데, 이 모듈은 일반적으로 커널 이미지를 로드하고 부팅 프로세스를 진행하는 데 필요한 기능들을 포함함.

즉, `insmod normal` 명령어는 GRUB에게 `normal` 모듈을 메모리에 불러오라는 뜻이다. 

```shell
gurb> insmod normal
gurb> normal
```

여기까지 실행시키면 리눅스가 정상적으로 시작된다. 


## 명령어 요약

```shell
# 1. 하드 디스크 탐색
grub> ls
(hd0) (hd0,msdos1) (hd0,msdos2) (hd1,msdos1) (hd1,msdos2)
# 이런 식으로 단계적으로 탐색해보면 된다...
grub> ls (hd1,msdos1)/efi/

# cfg 파일을 찾아낸 경우
grub> ls (hd1,msdos1)/efi/boot/
./ ../ grub.cfg

# 2. 부팅 파일 경로 설정
grub> set prefix=(hd1,msdos1)/efi/boot/
grub> set fw_path=(hd1,msdos1)/efi/boot/
grub> set root=hd1,msdos1

# 3. 설정 후, insmod 명령어 실행
gurb> insmod normal
gurb> normal
```