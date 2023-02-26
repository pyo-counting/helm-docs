chart 내에서 label, annotation을 사용하시 best practice에 대해 설명한다.

## Is it a Label or an Annotation?
metadata 중 아래 조건을 만족할 경우 label로 사용해야 한다:

- k8s가 resource를 식별하는 데 사용된다.
- It is useful to expose to operators for the purpose of querying the system.

예를 들어, `helm.sh/chart: NAME-VERSION` label을 사용함으로써 특정 chart의 모든 인스턴스를 편리하게 찾을 수 있다.

만약 metadata가 쿼리를 위해 사용되는 것이 아니라면, 이는 annotation으로 사용해야 한다.

helm hook은 항상 annotation이다.

## Standard Labels
아래는 helm chart가 사용하는 공통 label이다. helm은 특정 label에 대해 필수로 설정되어야 하는 것은 아니다. REC로 표시된 label은 권장되며 전체적인 일관성을 위해 chart에 배치해야 한다. OPT로 표시된 label은 선택 사항이다. 이들은 관용적이거나 일반적으로 사용되지만 운영 목적으로 자주 사용되지는 않는다.

|Name                           |Status|Description|
|-------------------------------|------|-----------|
|app.kubernetes.io/name         |REC   |전체 애플리케이션을 나타내는 이름. 보통 `{{ template "name" . }}`. 이는 많은 k8s manifest에서 사용하며 helm에 국한되지 않는다.|
|helm.sh/chart                  |REC   |chart 이름과 버전: `{{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}`|
|app.kubernetes.io/managed-by   |REC   |`{{ .Release.Service }}`. helm이 관리하는 모든 것을 찾기 위한 목적|
|app.kubernetes.io/instance     |REC   |`{{ .Release.Name }}`. 동일 애플리케이션의 서로 다른 인스턴스를 구분하기 위한 목적|
|app.kubernetes.io/version      |OPT   |`{{ .Chart.AppVersion }}`. 애플리케이션의 버전|
|app.kubernetes.io/component    |OPT   |애플리케이션의 구성요소가 수행하는 역할을 나타내기 위한 label. 예를 들어 app.kubernetes.io/component: frontend|
|app.kubernetes.io/part-of      |OPT   |여러 chart 또는 소프트웨어 구성요소를 같이 사용해 하나의 애플리케이션을 만드는 경우. 예를 들어 웹 사이트를 생성하기 위한 애플리케이션 소프트웨어 및 db. 지원되는 최상위 애플리케이션으로 설정할 수 있다.|

k8s 문서에서 app.kubernetes.io로 시작하는 k8s 관련 label을 확인할 수 있다.

helm을 이용해 chart 배포 시, 자동 추가되는 annotation, label은 아래와 같다.

- annotation
    - meta.helm.sh/release-name: {{ .Release.Name }}
    - meta.helm.sh/release-namespace: {{ .Release.Namespace }}
- label
    - app.kuberenetes.io/managed-by: {{ .Release.Service }}

추가적으로 helm은 chart release에 대한 버전 관리를 위해 배포 ns 내에 .spec.type=helm.sh/release.v{{ .Release.Revision }}인 secret을 생성하는 것을 확인할 수 있었다. secret의 이름은 다음과 같다.

- sh.helm.release.v2.{{ .Release.Name }}.v{{ .Release.Revision }}
