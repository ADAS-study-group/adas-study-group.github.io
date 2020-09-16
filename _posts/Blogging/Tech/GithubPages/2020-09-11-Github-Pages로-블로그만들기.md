---
title: Github Blog 만들기 
author: Jay Lee
date: 2020-09-11 14:10:00 +0800
categories: [Blogging, Tech, GithubPages]
tags: [GithubPages]
---

## How it works
Github 에서는 해당 계정 Username 이름으로 연동된 web page를 제공해둔다.

예를들면 필자의 Github User ID 가 jayleekr이고,
해당 계정의 Repository에 https://github.com/jayleekr/jayleekr.github.io 와 같이 새로운 블로그 용 Repository를 만들면
https://jayleekr.github.io/ 라는 주소로 웹페이지가 생성한 블로그용 Repository와 연동이 된다.

이 말은 해당 저장소를 가지고 있는 Github측(Remote)에서 백엔드 웹서버를 위 규칙에 의거하여 자동으로 생성해 준다는 의미이다. 

Front-end를 만들어 보자

## Front-end

Front-end 프레임웍이야 정말 종류가 많지만 좀 더 빠르게 만들기 위해 가장 유행하는 방법을 이용하여 만들어 보겠다.

이름하야 바로 **Jekyll** 인데, Github 공동 설립자인 Tom Preston-Werner 가 Ruby-on-rails 언어로 작성하였으며 MIT 라이센스를 따르는 정적 사이트 생성기이다.

(Github 공동 설립자가 만들어서 Github 와 기본 연동이 되는 건가)

## Create Repository 

도입부에서 언급했듯이 Github User name 에 맞게 생성해주면된다.

e.g.
User ID : jayleekr
Repository URL : https://github.com/jayleekr/jayleekr.github.io
Blog URL : https://github.com/jayleekr

저장소를 만들었으면 Local 에서 작업하여 Remote 에 Push 하는 형태로 작업할 계획이니, 블로그 저장소를 Local에 Clone 하자 

## Create Blog Site

Jekyll 을 이용하여 사이트를 생성하는 방식은 크게 두가지이다.
1. Local 에 Ruby, Bundler, 기타 gem Library들을 설치하여 Blog 를 생성하여 Remote에 Push하는 방법  
2. 애초에 jekyll 공식 저장소에서 Fork 해올때 블로그 화 할수있는 Repository 이름 형태로 땡겨오는 방법

> (필자는 2번 방식으로 Fork 해와서 기본적인 것들만 사용하다가, 
몇가지 애로사항으로 인하여 결국 2번방식으로 땡겨온 저장소를 Local에서 Debug 해가며 진행해야 했어서 1,2번 둘다 사용하고 있다)

### Install miscellaneous 

참조 : https://poiemaweb.com/jekyll-basics

### 특정 theme 설치
필자는 jekyll-theme-chirpy 라는 jekyll 테마를 설치할 예정이다.
~~이를 위해서는 jekyll installer 가 읽어드리는 설정파일들이 필요한데, 이러한 과정들이 번거로워 아예 해당 테마의 github source를 submodule 로 저장소에 추가하였다.~~
* 참조 : https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88

~~아래와 같은 커맨드로 submodule 로 저장소의 docs 라는 디렉토리를 jekyll의 chroot로 사용하도록 하면 끝이다.~~
```sh
$ git submodule add https://github.com/cotes2020/jekyll-theme-chirpy.git docs
$ cat .gitmodules
[submodule "docs"]
	path = docs
	url = https://github.com/cotes2020/jekyll-theme-chirpy.git
```
~~위와 같이 jekyll theme 을 가져오면 해당 디렉토리에는 ruby on rails 에서 사용할 gemfile과 기본적인 설정파일들 및 Markdown으로 작성된 문서들이 복사 된다.~~

처음에는 submodule로 설계했으나, submodule에 블로그를 포스팅

### Init Jekyll
위 예제 처럼 docs 디렉토리에 jekyll-chirpy 테마가 설치된 스켈레톤들이 설치됐다고 가정하자.
아래와 같이 실행하면, 해당 설정파일로 만든 Web Page 가 로컬호스트에서 동작한다.
```sh
$ bundle install
$ bundle exec jekyll serve
Configuration file: /home/jayleekr/00_Projects/06_ADAS/docs/_config.yml
            Source: /home/jayleekr/00_Projects/06_ADAS/docs
       Destination: /home/jayleekr/00_Projects/06_ADAS/docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.487 seconds.
                    Auto-regeneration may not work on some Windows versions.
                    Please see: https://github.com/Microsoft/BashOnWindows/issues/216
                    If it does not work, please upgrade Bash on Windows or run Jekyll with --no-watch.
 Auto-regeneration: enabled for '/home/jayleekr/00_Projects/06_ADAS/docs'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop. 
```
