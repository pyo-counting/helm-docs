chart manifest에서 po의 template에 대한 포맷에 대한 best practice를 설명한다.

아래 resource 목록은 PodTemplates를 사용한다:

- deploy
- rc
- rs
- ds
- sts

## Images
container image는 고정된 tag 또는 SHA를 사용해야 한다. latest, head, canary와 같은 tag를 사용하면 안된다.

image를 빠르게 교체할 수 있도록 image는 values.yaml 파일에 정의해야 한다.

``` yaml
image: {{ .Values.redisImage | quote }}
```

image와 tag는 2개의 필드로 구성해야 한다.

``` yaml
image: "{{ .Values.redisImage }}:{{ .Values.redisTag }}"
```

## ImagePullPolicy
helm create 명령어는 기본적으로 imagePullPolicy 필드를 IfNotPresent로 설정한다. 에를 들어 사용자의 deployment.yaml:

``` yaml
imagePullPolicy: {{ .Values.image.pullPolicy }}
```

그리고 values.yaml

``` yaml
image:
  pullPolicy: IfNotPresent
```

유사하게 k8s도 정의되지 않을 경우 imagePullPolicy 필드를 IfNotPresent로 설정한다. 이를 원치 않을 경우 values.yaml 내 원하는 값을 정의하면 된다.

## PodTemplates Should Declare Selectors
PodTemplate는 seletor를 명시해야 한다. 예를 들어:

``` yaml
selector:
  matchLabels:
      app.kubernetes.io/name: MyName
template:
  metadata:
    labels:
      app.kubernetes.io/name: MyName
```

위는 po와 관련 집합에 대해 관계성을 갖도록 하기 때문에 좋은 습관이다.

하지만 이는 deploy와 같은 resource일 경우 더 중요하다. selector가 없으면 전체 label 집합이 po를 선택하는 데 사용되며 버전 또는 릴리즈 날짜와 같이 변경되는 label을 사용할 수 있기 때문이다.