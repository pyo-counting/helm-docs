Chart.yaml에서 설정 가능한 dependencies의 best practice에 대해 설명한다.

## Versions
가능하다면 1개의 버전을 고정하지 않고 범위를 설정하는 것이 좋다. 권장되는 방식은 패치 버전 매칭이다:

``` yaml
version: ~1.2.3
```

~1.2.3는 >= 1.2.3, < 1.3.0에 해당된다.

### Prerelease versions
위 버전 표기법은 pre-release 버전에는 매칭되지 않는다. 예를 들어 위 버전 매칭은 version: ~1.2.3-1에는 매칭되지 않는다. 아래는 예시는 패치 매칭 뿐만 아니라 pre-release에 대해서도 매칭된다.

``` yaml
version: ~1.2.3-0
```

### Repository URLs
Where possible, use https:// repository URLs, followed by http:// URLs.

repo가 repo index file에 추가한 경우 repo 이름을 URL의 별칭으로 사용할 수 있다. alias: 또는 @ + repo 이름 형식으로 사용하면 된다.

파일 URL(file://...)은 고정된 배포 파이프라인으로 구성된 chart의 특수 케이스에 해당한다.

downloader plugin을 사용할 때 URL scheme은 플러그인에 따라 다르다. chart 사용자는 종속성을 업데이트하거나 구축하기 위해 설치된 scheme를 지원하는 플러그인이 있어야 한다.

helm은 dependency의 repository 필드가 공백이면 종속성 관리 작업을 수행하지 못한다. 이 경우 helm은 해당 charts/ 디렉토리에 종속성 chart가 있는 것으로 가정한다.

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