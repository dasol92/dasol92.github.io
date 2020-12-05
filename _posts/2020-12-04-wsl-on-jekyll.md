---
title:  "Jekyll on WSL"
categories: 
  - Jekyll
---

* TOC
{:toc}

### 1. Install WSL
Windows version 2020 에서 WSL2 설치
#### 1.1. Windows 버전 확인
Windows 키 → `winver` 입력해 버전 2020 이상 확인, 아니라면 <Windows 업데이트> 에서 업데이트 진행
#### 1.2. 개발자 모드 설정
Windows 키 → 개발자 설정→ 개발자 모드 '켬'
#### 1.3. Linux용 Windows 하위 시스템 설정
Windows 키 → Windows 기능 켜기/끄기 → 'Linux용 Windows 하위 시스템' 체크
#### 1.4. Ubuntu 설치
Microsoft Store 에서 Ubuntu 설치
> 본 문서에서는 Ubuntu 18.04 버전으로 진행

### 2. Install Packages on WSL
WSL 실행은 Windows 키 → Ubuntu 검색해 실행

Ruby, 빌드 툴 설치
```sh
sudo apt install ruby-full ruby-dev build-essential dh-autoreconf
```
Ruby, Gem 버전 확인
```sh
gem -v
ruby -v
```

Gem 업데이트
```sh
sudo gem update
```


Gem 으로 Jekyll, bundler 설치
```sh
sudo gem install jekyll bundler
```

Jekyll 버전 확인
```sh
$ jekyll -v
jekyll 4.1.1
```


설치가 완료되었다면 마음에 드는 테마를 wsl ubuntu 에 다운로드 받는다. 본인은 [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/) 테마를 선택했다. 

해당 테마의 디렉토리로 들어가서, 테마의 ruby 패키지들을 설치해준다. 
```sh
cd ~/minimal-mistakes-master
bundle
```

해당 테마에는 불필요한 파일들이 있다. 아래에 해당하는 파일 및 디렉토리들을 삭제해준다. 
```
.editorconfig
.gitattributes
.github
/docs
/test
CHANGELOG.md
minimal-mistakes-jekyll.gemspec
README.md
screenshot-layouts.png
screenshot.png
```

블로그 포스팅 작성을 위해 필요한 디렉토리들을 생성해준다. 
```
_posts : 자신이 작성한 포스팅들을 담는 폴더
_drafts : 포스팅들의 드래프트를 담는 공간
```

`Gemfile` 에는 해당 테마의 의존성을 기술해준다. 
```
source "https://rubygems.org"

gem "jekyll", "~> 3.5"
gem "minimal-mistakes-jekyll"
gem "kramdown-parser-gfm"
```

서버 실행
```sh
bundle exec jekyll serve
```