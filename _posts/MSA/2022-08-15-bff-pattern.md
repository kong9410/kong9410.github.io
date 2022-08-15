---
title: BFF(Backend For Frontend) 패턴
tags: msa
layout: post
description: 여러 유형의 클라이언트의 API Gateway는 어떻게 처리하는가?
---

PC뿐 아니라 다양한 장비를 사용하기 때문에 다양한 클라이언트를 고려해야 한다. 다양한 클라이언트를 위해서는 특화된 처리를 위한 API 조합이나 처리가 필요하다. 이를 해결하기 위한 방법으로 BFF(Backend For Frontend) 패턴이 있다.

![Backend for frontend - BFF example](https://res.cloudinary.com/practicaldev/image/fetch/s--vAxUNLcm--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lcxzmd5seiiwadqtvjwg.png)

BFF 패턴은 API 게이트웨이와 같은 진입점을 하나로 두지 않고 프론트엔드 유형에 따라 각각 두는 패턴이다. 프론트엔드를 위한 백엔드라는 의미로 BFF(Backend For Frontend) 라고 부른다. 웹을 위한 API 게이트웨이, 모바일을 위한 API 게이트웨이 등 클라이언트 종류에 따라 최적화된 처리를 수행할 수 있게 구성할 수 있다. 이로써 모바일을 위한 API만 선택해서 제공하거나 웹을 위한 API만 적절하게 제공할 수 있다. 또한 각 프론트엔드에 대한 처리만 수행하는 BFF를 두고 이후에 통합적인 API 게이트웨이를 둠으로써 공통적인 인증/인가, 로깅 등의 처리를 통제하는 구조로 구성할 수도 있다.

## 원문

[도메인 주도 설계로 시작하는 마이크로서비스 개발 - YES24](http://www.yes24.com/Product/Goods/98880996)