---
title: 블로그 Google Analytics 연동하여 사용자 수 파악하기
author: D.G.Lee
date: 2021-09-30 15:00:00 +0900
categories: [Blog-management]
tags: [Blog-management]
math: true
mermaid: true
---

## Overview

GA(Google Analytics) 는 웹 페이지 또는 모바일 어플리케이션 사용자의 접근 기록이나, 행동 데이터들을 수집하고 분석하는 툴입니다.
실적을 계산하거나 웹 사이트/모바일 어플리케이션을 개선시키는 등의 다양한 용도로 활용되지만 저는 단순히 제 블로그의 접근하는 분들이 얼마나 될까? 라는 궁금증을 해결하기위해서 활용해보았습니다. ㅎㅎ...

## Google Analytics 생성

Google 계정으로 로그인하여 Google Analytics 페이지에서 측정 시작을 통해 초기 설정을 진행할 수 있습니다.
- [https://analytics.google.com](https://analytics.google.com) (Google Analyitcs)

![Google-Analytics-measure-1](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-measure-1.png)

![Google-Analytics-measure-2](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-measure-2.png)

![Google-Analytics-measure-3](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-measure-3.png)

![Google-Analytics-measure-3-advanced](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-measure-3-advanced.png)

![Google-Analytics-measure-4](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-measure-4.png)

위 스텝을 성공적으로 진행하면 계정(piou987), Universal Analytics 속성(wsscc2021.github.io), Google Analytics 4 속성(wsscc2021.github.io - GA4)이 생성됩니다.

## _config.yml 파일 수정

웹페이지 접속 시 Google Analytics로 데이터를 보내기 위해서는 별도의 HTML+JS를 작성해야하지만, 
제가 사용하는 테마에는(`Jekyll-theme-chirpy`) 기능이 이미 구현되어 있어서 `_config.yml` 파일에 `Analytics ID`만 입력하면 사용할 수 있었습니다.  

Google Analytics 4 속성의 웹 스트림 ID를 복사하여 `_config.yml`파일에 붙여넣습니다.

![Google-Analytics-Check-StreamID-1](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-Check-StreamID-1.png)

![Google-Analytics-Check-StreamID-2](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-Check-StreamID-2.png)

```yaml
google_analytics:
  id: 'G-2Y7BLPLM54'              # fill in your Google Analytics ID
  # Google Analytics pageviews report settings
  pv:
    proxy_endpoint:   # fill in the Google Analytics superProxy endpoint of Google App Engine
    cache_path:       # the local PV cache data, friendly to visitors from GFW region
```


## Google Analytics Developer Page

만약 테마에 기능이 구현되어 있지 않다면 Google Analytics의 Developer 페이지를 참조하면 좋을 것 같습니다.

- [https://developers.google.com/analytics/devguides/collection/gtagjs](https://developers.google.com/analytics/devguides/collection/gtagjs)

## 동작 확인

변경한 내용을 반영한 뒤 블로그에 접근해보면, Google Analytics에 사용자 수가 집계되는 것을 볼 수 있습니다.

![Google-Analytics-Report](/assets/img/posts/2021-09-30-블로그 Google Analytics 연동하여 사용자 수 파악하기/Google-Analytics-Report.png)

## Page View

GCP(Google Cloud Platform)의 App Engine을 사용하여 Google Analytics 의 Report를 노출시키고 이를 기반으로 Page view 기능도 활용할 수 있습니다.
하지만 나중에 사용자 수가 많아져서 App Engine 비용이 발생하게 될까 싶어 활용하지 않으려 합니다.
무엇보다 Google Analytics에서 잘 분석하면 얻을 수 있는 정보라서... 크게 쓸모가 없다고 생각했습니다.