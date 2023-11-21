---
title: "Django Debug Toolbar란?"
categories:
  - Django
tags:
  - Django
  - Debug
toc: true
toc_sticky: true
toc_label: "Django 튜토리얼"
toc_icon: "book"
---

## 서론
Django Debug Toolbar는 Django 애플리케이션의 디버깅과 성능 최적화를 위한 도구입니다. [공식 문서](https://django-debug-toolbar.readthedocs.io/en/latest/)를 참고하면 더 많은 정보를 얻을 수 있습니다.

## 기능과 활용
<img  alt="image" src="https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/978a8866-51ec-4a1d-8bb8-323d59d3954f">

Django Debug Toolbar은 다양한 패널을 제공하여 애플리케이션의 다양한 측면을 분석합니다. 특히, SQLPanel은 각 요청에 대한 SQL 쿼리 내용을 확인하는 데 사용됩니다. 이를 통해 데이터베이스와의 상호작용을 세밀하게 살펴볼 수 있습니다. 

**주의사항**
프로덕션 환경에서는 Debug 모드를 활성화하는 것은 권장되지 않습니다. Debug 모드는 보안 문제를 야기할 수 있으며, 쿼리 누적 기능은 메모리를 더 많이 사용할 수 있습니다. 또한, 실제 운영 중에는 디버깅 툴바의 정보를 노출시키면 보안상의 위험이 있을 수 있으므로 주의해야 합니다.

## 결론
Django Debug Toolbar는 개발 중에 쿼리 실행 내역 및 다양한 디버깅 정보를 시각적으로 제공하여 효율적인 개발을 돕는 도구입니다. 그러나, 프로덕션 환경에서는 사용 시 주의가 필요하며, settings.DEBUG를 적절히 관리해야 합니다. 
