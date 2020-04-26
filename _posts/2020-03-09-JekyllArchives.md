---
title: "jekyll-archives plugin으로 Github Page 카테고리별 게시글 구성하기"
categories:
  - tools
author_profile: true
read_time: true

last_modified_at: 2020-03-09T18:27:00+09:00

classes: wide
toc: false
---





## 0. 삽질 이야기 + Prerequisites

예전에는 네이버 블로그에 가끔(아주 가끔...) 포스팅 했었는데 다시 꾸준히 공부하자고 결심하면서 Github Page가 더 깔끔해보여서 바꾸게 되었다. Github Page에 대해서는 [이 글(Github Page 소개)](https://devinlife.com/howto github pages/github-blog-intro/) 를 참고해주세요. 테마가 다양하게 제공되는데 나는 minimal-mistakes를 골랐다. 심플하기도 하고 원래 뭘 고를지 고민될 때에는 다들 많이 쓰는 쪽으로 대세에 편승하는 쪽이 속이 편하다.

[Viewpoint: 블로그 생성](https://hide1202.github.io/blogs/create-blog/) 을 참고해서 블로그를 생성했고 여기서는 Github page에서 카테고리별 게시글을 볼 수 있는 페이지와 해당 페이지로 이어지는 메뉴바를 만드는 것을 다룰 예정이다.  이 내용만 쓰는 건 내가 여기서 헤맸기 때문... Github Page 생성하고 Jekyll 설치하고 하는 것들은 여기저기 잘 되어있는 글들이 많으니까 구글 선생님께 물어보면 될 것 같다.

카테고리별 게시글을 보여주는 페이지를 만드려면 jekyll-archives를 설치해야 한다. 그런데 Github Page는 [지킬 공식 문서](https://jekyllrb.com/docs/plugins/installation/) 에 따르면 --safe 옵션을 붙여서 페이지를 생성하기 때문에 플러그인들이 적용되지 않는다. (이걸 모르고 푸시해놓고 어... 왜 카테고리 전부 다 보여주지... 하면서 헤매고 있었다ㅠㅠ)



#### Prerequisites

- [Minimal-Mistakes Github](https://github.com/mmistakes/minimal-mistakes) 클론해서 Github Page 생성하기
- Ruby와 Jekyll 설치하기



## 1. 카테고리 별 메뉴바 생성하기

Github Page 폴더에 있는 /_data/navigation.yml을 수정한다. 나의 경우에는 C# 공부, 프밍 관련 책, 지금 쓰는 jekyll-archives 같은 도구와 기타 등등 애매한 분류의 글을 넣을 Blog 4개의 카테고리를 만들었다.

```yaml
main:
  - title: "About"
    url: /about/
  - title: "Category"
    url: /categories/
  - title: "Blog"
    url: /categories/blog/
  - title: "C Sharp"
    url: /categories/csharp/
  - title: "Books"
    url: /categories/books/
  - title: "Tools"
    url: /categories/tools/
```

메뉴를 만들었다면 이제 메뉴바의 메뉴를 눌렀을 때 보여질 페이지도 만들어야 한다.

우선 전체 카테고리의 글을 보여주는 Category에 해당하는 페이지는 아래의 /_pages/category-archive.md이다. `layout`을 categories로 설정해주고, 연결되는 링크인 `permalink` 도 그에 맞게 써준다. `author_profile`은 전체 _config.yml에서 설정한 author 정보를 해당 페이지에서도 보여줄 것인지를 선택하는 것이다.

```
---
title: "Posts by Category"
layout: categories
permalink: /categories/
author_profile: true
---
```

특정 카테고리만 보여주려면 `taxonomy`라는 항목을 추가해야 한다. 아래는 내가 만든 blog.md 파일이다. `permalink`는 /categories/blog와 이어주고, `taxonomy`는 해당 페이지에서 보여주고 싶은 카테고리 이름을 적으면 된다.

```blog.md
---
title: "Personal Blog"
layout: categories
permalink: /categories/blog/
author_profile: true
taxonomy: blog
---
```



## 2. jekyll-archives 설치하기

[jekyll-archives](https://github.com/jekyll/jekyll-archives) 는 Github Page의 글들을 날짜, 태그, 카테고리에 의해 자동으로 분류해주는 플러그인이다.

이 플러그인을 설치하기 위해서는 _config.yml과 Gemfile을 수정하고 플러그인을 다운받아 수정하면 된다.



/_config.yml 파일의 plugins 부분에서 다음과 같이 jekyll-archives를 추가한다.

```yml
# Plugins (previously gems:)
plugins:
  - jekyll-paginate
  - jekyll-sitemap
  - jekyll-gist
  - jekyll-feed
  - jekyll-include-cache
  - jekyll-archives

# mimic GitHub Pages with --safe
whitelist:
  - jekyll-paginate
  - jekyll-sitemap
  - jekyll-gist
  - jekyll-feed
  - jekyll-include-cache
  - jekyll-archives
```
Gemfile의 plugin group에도 jekyll-archives를 추가한다.

```yaml
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-algolia"
  gem "jekyll-include-cache"
  gem "jekyll-archives"
end
```

이후 WSL bash shell에서 `bundle install`을 실행하면 플러그인이 설치된다.

이렇게 하면 로컬에서 돌렸을 때에는 각 페이지마다 해당 카테고리의 게시글을 볼 수 있다. Github Page에서는 아니지만....



## 3. 로컬에서 페이지를 빌드하고 마스터 브랜치로 푸시하기

이 부분은 [방구석엔지니어: Github 블로그에서 Jekyll 플러그인 사용하기](https://eunvanz.github.io/jekyll/2018/01/29/github-블로그에서-Jekyll-플러그인-사용하기/) 이 분 글을 참고해서 진행했다.

간략하게만 설명하자면 master와 source branch를 만들어 기본 브랜치를 source branch로 하고, 모든 작업은 source branch에서 실행한다. 입력 후 source branch 내용을 빌드하고, Rakefile을 수정해 `rake publish` 명령어를 실행하면 빌드한 source branch 내용을 master branch로 자동으로 푸시할 수 있게 하는 것이다.





이렇게 카테고리 설정은 끝! 한 번 설정해 놨으니 이제 마크다운에 익숙해지고 열심히 공부하고 열심히 정리해야겠다.



## Reference

[devinlife: Github 블로그 메뉴 구성하기](https://devinlife.com/howto github pages/blog-menu/)

[devinlife: 블로그 카테고리와 태그 목록 구성하기](https://devinlife.com/howto github pages/category-tag/)

[방구석엔지니어: Github 블로그에서 Jekyll 플러그인 사용하기](https://eunvanz.github.io/jekyll/2018/01/29/github-블로그에서-Jekyll-플러그인-사용하기/)

