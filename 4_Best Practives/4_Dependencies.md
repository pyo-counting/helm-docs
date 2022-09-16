Chart.yaml에 명시된 dependencies에 대한 best practice에 대해 설명한다.

## Versions

### Prerelease versions

### Repository URLs

## Conditions and Tags
condition, tag는 optional한 dependencies에 추가되어야 한다.

condition의 기본 형식은 다음과 같다:

``` yaml
condition: somechart.enabled
```

somchart는 dependency의 이름이다.

여러 subchart가 함께 optional 기능을 제공하는 경우 동일한 tag를 공유해야 한다.

예를 들어 nginx, memcached가 함께 chart의 기본 애플리캐이션에 대한 성능 최적화를 제공하고 해당 기능이 활성화 되기 위해 둘 다 있어야 하는 경우 다음과 같은 tag가 있어야 한다:

``` yaml
tags:
  - webaccelerator
```

사용자는 1개의 tag를 통해 2개의 subchart를 활성화/비활성화 할 수 있다.