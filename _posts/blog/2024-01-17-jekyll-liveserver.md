---
layout: single
title: "jekyll 블로그, 라이브 서버를 통해 변경 사항 확인하기"
categories: [blog]
tag: [jekyll]
---
*(windows 기준)*

### 1. Ruby 설치
{% include link.html
    url="https://rubyinstaller.org/downloads/"
    title="RubyInstallers"
    description="RubyInstallers for Windows"
%}

> 1. 권장 설치 버전은 버전 앞에 `=>`가 붙어있다고 한다.<br>
> 2. `WITH DEVKIT`으로 설치해야한다.

### 2. cmd에서 `bundler` 설치
실행 전 필요한 패키지들을 설치해준다. 설치는 `cmd` 창에서 진행한다. 만약 `gem` 명령어를 인식하지 못한다면, 설치한 Ruby에 대해 **환경 변수 설정**이 필요하다.

```shell
# jekyll 및 bundler 설치
> gem install jekyll
> gem install bundler
```

### 3. 라이브 서버 실행
라이브 서버는 jekyll 블로그 폴더 루트 경로에서 실행한다. 만약 `Gemfile`을 찾을 수 없다는 문구를 발견한다면, `Gemfile`이 있는 위치로 이동하여 실행한다.

```shell
# 실행 명령어
> bundle exec jekyll serve

# 자동 갱신 옵션 추가
> bundle exec jekyll serve --livereload
```
`_config.yml` 파일을 수정한 경우에는, 자동 갱신 옵션이 말을 듣지 않는 경우가 있다. 라이브 서버를 종료 후, 재실행하면 된다.

### 4. (오류 발생 시) 필요한 패키지들을 Gemfile에 추가 및 설치

실행을 하다보면 아래와 같은 오류가 발생한다.<br>
```
Dependency Error: Yikes! It looks like you don't have jekyll-gist or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. If you've run Jekyll with bundle exec, ensure that you have included the jekyll-gist gem in your Gemfile as well. The full error message from Ruby is: 'cannot load such file -- jekyll-gist' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/!
------------------------------------------------
Jekyll 4.3.3   Please append --trace to the serve command
for any additional information or backtrace.
------------------------------------------------
```

오류를 자세히 읽어보면, 명령어 실행에 무엇이 필요한지 알 수 있다. 위의 오류의 경우에는 jekyll-gist가 필요하다고 알려주고 있다. 위의 오류 메세지를 참고해서, `Gemfile`에 내용을 추가해준다. (보통 `Gemfile`은 jekyll 블로그 폴더의 루트 경로 내에 있다.)

나는 아래의 패키지를 전부 추가하고 나서야 오류가 없어졌다. 다른 PC에서 라이브 서버를 실행할 때는 아래만큼 패키지를 설치할 필요가 없었던 것으로 보아, **상황/환경에 따라 필요한 패키지나, 발생하는 오류의 종류가 달라질 것이라고 추측**한다.
```
source "https://rubygems.org"
gem "webrick", "~> 1.8"
gem "jekyll"
gem "jekyll-include-cache"
gem "jekyll-feed"
gem "jekyll-sitemap"
gem "jekyll-paginate"
gem "jekyll-gist"
gem "tzinfo"
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw]
```

`Gemfile`에 내용을 추가했다면, `bundle install` 명령어도 실행해주어야 한다.

```
> bundle install
```

**Complete!** 라는 문구가 뜨면 설치가 완료된 것이다. [3번](#3-라이브-서버-실행)의 명령어를 실행해보면서, 오류가 없어질 때까지 `Gemfile`을 수정한다. 가끔 `Gemfile.lock` 파일을 삭제 후 `bundle install` 명령어를 재실행 해보면 오류가 해결되는 경우가 있다.


### 5. 라이브 서버 접속
```
Run in verbose mode to see all warnings.
                    ...done in 0.571901 seconds.
```
라는 문구가 뜨면서, 명령어가 문제 없이 실행된 것 같다면 아래의 주소를 통해 라이브 서버에 접근해본다.
```
localhost:4000
```

라이브 서버에서 jekyll 블로그가 라이브 서버에서 실행되는지 확인해본다.