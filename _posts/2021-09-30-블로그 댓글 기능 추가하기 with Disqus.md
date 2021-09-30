---
title: 블로그 댓글 기능 추가하기 with Disqus
author: D.G.Lee
date: 2021-09-30 11:00:00 +0900
categories: [Blog-management]
tags: [Blog-management]
math: true
mermaid: true
---

## Overview
Jekyll 로 만든 "정적"웹페이지는 댓글 추가/삭제/수정과 같은 동적인 작업을 수행하지 못합니다. 때문에 댓글 서비스를 별도로 구축해야하는 데, 직접 구축하기 보다는 완성된 서비스를 제공해주는 Disqus를 활용하는 것이 간편합니다. 이번 글에서는 제 블로그에 Disqus를 적용했던 내용을 적으려 합니다.
- Jekyll + Github.io
- Jekyll-theme-chirpy
- Disqus

## Disqus Site 생성
댓글 기능을 위해 Disqus Site를 생성해야 합니다. Disqus 홈페이지에 로그인 한 뒤 Get Started를 통해 간단하게 사이트를 추가할 수 있습니다.
- https://disqus.com/ (Disqus)

![Disqus-GetStarted.png](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-GetStarted.png)

![Disqus-SelectSite.png](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-SelectSite.png)

![Disqus-CreateNewSite-1](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-1.png)

![Disqus-CreateNewSite-2](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-2.png)

![Disqus-CreateNewSite-3](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-3.png)

![Disqus-CreateNewSite-4](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-4.png)

![Disqus-CreateNewSite-5](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-5.png)

![Disqus-CreateNewSite-6](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-CreateNewSite-6.png)

## _config.yml 파일 수정

Disqus 서비스를 웹페이지에서 사용하기 위해서는 별도의 HTML+JS를 작성해야하지만, 
제가 사용하는 테마에는(`Jekyll-theme-chirpy`) 기능이 이미 구현되어 있어서 `_config.yml` 파일에 `Shortname`만 입력하면 사용할 수 있었습니다.  
추가된 Disqus Site의 정보를 확인하여 `Shortname`을 복사하고 `_config.yml` 의 disqus 설정값에 붙여넣습니다.

![Disqus-Edit-Settings-1](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-Edit-Settings-1.png)

![Disqus-Edit-Settings-2](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-Edit-Settings-2.png)

`_config.yml`
```yaml
disqus:
  comments: true  # boolean type, the global switch for posts comments.
  shortname: 'wsscc2021-github-io'    # Fill with your Disqus shortname. › https://help.disqus.com/en/articles/1717111-what-s-a-shortname
```

## 적용된 내용 확인

위와 같이 설정한 뒤 블로그 게시글의 맨 마지막으로 넘어가 확인해보면, 아래와 같이 Disqus 서비스가 제공해주는 댓글 기능을 활용할 수 있는 상태가 되어 있습니다. 

![Disqus-Check-on-blog-post](/assets/img/posts/2021-09-30-블로그 댓글 기능 추가하기 with Disqus/Disqus-Check-on-blog-post.png)

## Disqus embed code

만약 다른 Theme를 사용하고 있어서 Disqus 기능이 구현이 안되어 있고 직접 코드를 작성해야한다면 Disqus Help 페이지를 참조하면 좋을 것 같습니다.
- [https://help.disqus.com/en/articles/1717112-universal-embed-code](https://help.disqus.com/en/articles/1717112-universal-embed-code)