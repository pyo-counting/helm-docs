helm은 chart라고 부르는 패키징 포맷을 사용한다. chart는 관련 k8s resource들을 설명하는 파일 집합이다.

chart는 고유한 파일/디렉토리 구조를 갖는 디렉토리다. 디렉토리는 배포 버전이 지정된 아카이브(*.tgz)로 패키징될 수 있다.

chart 배포 없이 다운로드만 원할 경우 `helm pull` 명령어를 사용하면 된다.

## The Chart File Structure
chart는 디렉토리내 파일의 집합으로 관리된다. 디렉토리 이름은 버전 정보가 없는 chart의 이름이다. 아래는 workdress chart의 디렉토리 구조 예시다:

``` bash
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

Helm reserves use of the charts/, crds/, and templates/ directories, and of the listed file names. Other files will be left as they are.

## The Chart.yaml File
`Chart.yml`는 필수 파일이며 아래 필드를 갖는다:

``` yaml
apiVersion: The chart API version (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
type: The type of the chart (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this projects home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
dependencies: # A list of the chart requirements (optional)
  - name: The name of the chart (nginx)
    version: The version of the chart ("1.2.3")
    repository: (optional) The repository URL ("https://example.com/charts") or alias ("@repo-name")
    condition: (optional) A yaml path that resolves to a boolean, used for enabling/disabling charts (e.g. subchart1.enabled )
    tags: # (optional)
      - Tags can be used to group charts for enabling/disabling together
    import-values: # (optional)
      - ImportValues holds the mapping of source values to parent key to be imported. Each item can be a string or pair of child/parent sublist items.
    alias: (optional) Alias to be used for the chart. Useful when you have to add the same chart multiple times
maintainers: # (optional)
  - name: The maintainers name (required for each maintainer)
    email: The maintainers email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). Needn't be SemVer. Quotes recommended.
deprecated: Whether this chart is deprecated (optional, boolean)
annotations:
  example: A list of annotations keyed by name (optional).
```

v3.3.2 버전부터 추가 필드는 허용되지 않는다. 이를 위해 권장하는 방법은 annotation를 사용하는 것이다.

### Charts and Versioning
모든 chart는 버전을 갖는다. 버전은 반드시 [SemVer2](https://semver.org/spec/v2.0.0.html) 표준을 따라야한다. repo 내에서 패키지는 이름과 버전으로 식별된다.

예를들어 Version: 1.2.3 필드를 갖는 nginx chart는 `nginx-1.2.3.tgz` 이름을 갖는다.

Chart.yaml 파일 내 version 필드는 CLI를 포함한 많은 helm 도구에서 사용한다. 패키지를 생성할 때 `helm package` 명령어는 Chart.yaml파일에서 찾은 버전을 패키지 이름의 토큰으로 사용한다. 시스템은 chart 패키지 이름 내 버전이 Chart.yaml 파일 내 버전과 일치한다고 가정하며 이를 충족하지 못할 경우 오류가 발생한다.

### The `apiVersion` Field
helm 3부터 apiVersion 필드는 v2 값을 사용해야 한다.

### The `appVersion` Field
appVersion과 version 필드는 연관이 없다. 이는 애플리케이션의 버전을 나타낸다. 이 필드는 문자열로 인식하기 위해 quote하는 것을 권장한다.

As of Helm v3.5.0, `helm create` wraps the default appVersion field in quotes.

### The `kubeVersion` Field
kubeVersion 옵션 필드는 지원되는 k8s 버전에 대한 제약 조건을 정의한다. helm은 chart install 시 해당 제약 조건의 유효성을 검사하고 cluster가 지원하지 않는 k8s을 사용하는 경우 실패한다.

`공백`은 AND 연션자, `||`은 OR 연산자를 의미한다. `=`, `!=`, `>`, `<`, `>=`, `<=` 연션자도 지원한다. 뿐만 아니라 아래와 같은 연산자도 지원한다

- `-`: 범위를 나타낸다. 1.1 - 2.3.4 는 >= 1.1 <= 2.3.4 와 동일
- `x`, `X`, `*`(와일드카드): 1.2.x 는 >= 1.2.0 < 1.3.0 과 동일
- `~`: patch 버전 업데이트를 허용. ~1.2.3 은 >= 1.2.3 < 1.3.0 과 동일
- `^`: minor 버전 업데이트를 허용. &1.2.3 은 >= 1.2.3 < 2.0.0 과 동일

### Deprecating Charts
chart repo를 통해 chart를 관리할 때 deprecate 표시가 필요할 수 있다. 이는 Chart.yaml 파일 내 deprecated 필드를 사용하면 된다. 최신 버전의 chart가 deprecated되면 해당 chart 자체에 대해 deprecated된 것으로 간주한다. 물론 최신 버전을 다시 배포해 deprecated 상태를 원복할 수 있다.

### Chart Types
type 필드는 chart 타입을 나타낸다. `application`, `library` 값이 있다. application은 기본 값으로 resource 오브젝트를 포함하는 설치 가능한 chart다. 반면 library는 chart builder를 위한 유틸리티 또는 기능을 제공한다. library chart는 설치가 불가하며 일반적으로 resource 오브젝트를 포함하지 않는다는 점에서 application chart와 다르다.

**Note**: application chart는 library chart로 사용할 수 있다. type을 library로 설정하면된다. 그러면 chart가 모든 유틸리티와 기능을 활용할 수 있는 library chart로 렌더링된다. chart의 모든 resource 객체는 렌더링되지 않는다.

## Chart LICENSE, README and NOTES
chart는 설치, 설정, 사용, 라이센스를 설명하는 파일도 포함한다.

**LICENSE**: chart의 license를 포함하는 일반 텍스트 파일이다. 

**README.md**: Markdown 포맷(README.md)을 사용해야 한다. 일반적으로 아래 내용을 포함한다.

- chart가 제공하는 애플리케이션, 서비스에 대한 설명
- chart를 실행하기 위해 필요한 사항
- values.yaml과 기본 값에 대한 설명
- chart의 설치, 설정과 관련된 다른 정보

ArtifactHub와 같은 사용자 인터페이스 환경에서 chart에 대한 세부 사항은 README.md 내용을 보여준다.

**templates/NOTES.txt**: chart 설치, release의 상태 조회 시 출력되는 정보를 포함하는 파일. 이 파일은 template으로 간주된다. 보통 사용 참고 사항, 다음 단계, 기타정보를 표시하는 데 사용될 수 있다. 예를 들어 DB 연결, 웹 UI 접근에 대한 방법을 제공할 수 있다. 이 파일은 `helm install`, `helm status` 실행 시 표준 출력(STDOUT)으로 출력된다. 간략한 내용을 포함하면서 자세한 내용은 README에 포함하는 것을 권장한다.

## Chart Dependencies
chart는 다른 여러 chart를 포함할 수 있다. 이러한 의존성은 Chart.yaml 파일내 dependencies 필드를 통해 동적으로 구성하거나, charts/ 디렉토리에 수동으로 관리할 수 있다.

### Managing Dependencies with the dependencies field
현재 chart에서 필요로하는 의존성 있는 chart는 dependencies 필드에 정의된다.

``` yaml
dependencies:
  - name: apache
    version: 1.2.3
    repository: https://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: https://another.example.com/charts
```

- name: chart 이름
- version: chart 버전
- repository: chart repo 주소. 이 repo 역시 `helm repo add` 명령어를 사용해 추가해야 한다. 주소 대신 이름을 사용할 수도 있다.

``` bash
helm repo add fantastic-charts https://fantastic-charts.storage.googleapis.com
```

``` yaml
dependencies:
  - name: awesomeness
    version: 1.0.0
    repository: "@fantastic-charts"
```

`helm dependency update` 명령어를 사용하면 정의된 chart를 charts/ 디렉토리에 패키지로 다운로드 한다.

### Alias field in dependencies
alias 옵션 필드를 통해 별칭을 추가하면 해당 chart의 별칭을 이름으로 사용한다.

``` yaml
# parentchart/Chart.yaml

dependencies:
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-1
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-2
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
```

위 예시에서 parentchart는 3개의 종속 chart를 갖는다:

``` yaml
subchart
new-subchart-1
new-subchart-2
```

위의 동작을 직접 수행하기 위해서는 동일한 chart에 대해 다른 디렉토리의 이름을 사용해 chart/ 디렉토리에 위치시킨다.

### Tags and Condition fields in dependencies
tags, condition 옵션 필드를 사용해 종속 chart의 활성화 여부를 선택할 수 있다.

- condition: 콤마로 구분되는 1개 이상의 YAML 경로. 최상위 부모의 변수에 해당 경로가 존재하면 boolean 값으로 해석 및 활성화/비활성화 여부를 판단한다. 목록 중 유효한 첫 번째 경로가 평가되고, 경로가 존재하지 않으면 아무 영향이 없다.

- tags: 최상위 부모 변수에서 모든 종속 chart의 tags 값을 사용(boolean)해 해당 chart를 활성화/비활성화 할 수 있다.

``` yaml
# parentchart/Chart.yaml

dependencies:
  - name: subchart1
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart1.enabled,global.subchart1.enabled
    tags:
      - front-end
      - subchart1
  - name: subchart2
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart2.enabled,global.subchart2.enabled
    tags:
      - back-end
      - subchart2
```

``` yaml
# parentchart/values.yaml

subchart1:
  enabled: true
tags:
  front-end: false
  back-end: true
```

subchart1의 경우 front-end tag가 false이지만 subchart.enalbed가 true이기 때문에 활성화된다.

sybchart2의 경우 모든 condition의 경로가 존재하지 않기 때문에 영향력이 없으며,. back-end tag가 true이기 때문에 활성화된다.

#### Using the CLI with Tags and Conditions
--set flag를 통해 tag, condition 값을 변경할 수 있다.

``` bash
helm install --set tags.front-end=true --set subchart2.enabled=false
```

#### Tags and Condition Resolution
- condition은 설정된 값이 유효할 경우 tag보다 우선순위가 높다. 존재하는 첫 번째 condition 경로가 우선이고 뒤의 condition은 무시된다.
- tag는 chart의 tag가 true이면 chart를 활성화한다.
- tag, condition은 최상위 부모 value에 설정돼야 한다.
- tags: 최상위에 정의돼야 한다. 현재 global, 내장 tags:는 지원하지 않는다.

### Importing Child Values via dependencies
경우에 따라 하위 chart의 변수를 상위 chart로 전파해 공통 기본값으로 사용할 수도 있다.

export 포맷을 사용하는 것에 대한 추가 이점은 it will enable future tooling to introspect user-settable values.

부모 chart의 Chart.yaml 파일의 dependencies\[*\].import-values 필드에 하위 chart의 exports 필드의 변수를 나열할 수 있다.

하위 chart의 exports에 포함되지 않은 값을 가져오기 위해서는 child-parent 포맷을 사용한다. 두 형식에 대해 아래에서 설명한다.

#### Using the exports format
하위 chart의 values.yaml 파일의 최상위 루트에 exports 필드를 포함하는 경우, 아래와 같이 상위 chart에서 import할 key를 명시함으로써 하위 chart의 변수를 가져올 수 있다. 아래는 예시다:

``` yaml
# parent's Chart.yaml file
dependencies:
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    import-values:
      - data

# child's values.yaml file
exports:
  data:
    myint: 99
```

dependencies\[*\].import-values 목록에 data key 값을 명시했기 때문에 하위 chart의 exports 필드 목록 중 data key에 포함된 값을 가져온다. 최종적으로 상위 chart에서는 아래와 같이 변수가 포함된다:

``` yaml
# parent's values

myint: 99
```

상위 chart의 dependencies\[*\].import-values에서 명시한 data라는 key를 가진 필드의 내용을 가져오기 때문에 실제 key인 data는 가져오지 않음을 주의해야 한다,

#### Using the child-parent format
하위 chart의 exports 필드에 포함되지 않은 변수를 상위 chart에서 import하기 위해 하위 chart에서 가져올 데이터의 경로와, 상위 chart에서 가져온 데이터의 경로를 모두 명시해야 한다. 아래는 import를 수행하기 전 상위, 하위 chart의 values를 보여준다:
``` yaml
# parent's values.yaml file
myimports:
  myint: 0
  mybool: false
  mystring: "helm rocks!"

# subchart1's values.yaml file
default:
  data:
    myint: 999
    mybool: true
  ```

상위 chart에서 import-values 필드에 아래와 같이 설정할 경우:
``` yaml
# parent's Chart.yaml file
dependencies:
  - name: subchart1
    repository: http://localhost:10191
    version: 0.1.0
    ...
    import-values:
      - child: default.data
        parent: myimports
```

결과는 다음과 같다:
``` yaml
# parent's final values
myimports:
  myint: 999
  mybool: true
  mystring: "helm rocks!"
```


### Managing Dependencies manually via the charts/ directory
종속성에 대한 더 많은 제어가 필요한 경우 chart/ 디렉토리에 해당 chart를 복사해 직접 제어하면 된다. 해당 디렉토리의 경우 unpacked chart가 포함되어야 하며 이름은 _, .로 시작할 수 없다. 이러한 파일은 chart loader에 의해 무시된다.

### Operational aspects of using dependencies
위에서는 chart의 종속성에 대해 설명했다. 이러한 종속성은 helm install, helmm upgrade 명령어에 어떤 영향을 미칠까?

helm은 chart를 install, upgrade 할 때 해당 chart와 모든 종속성이 있는 chart에 대해:

1. 단일 chart인 것처럼 모두 집계한다.
2. resource 타입 -> 이름 우선 순위로 정렬한다.
3. 위 순서로 create, update를 수행한다.

## Templates and Values
모든 template 파일은 chart의 templates/ 폴더 아래에 있는 것으로 간주한다. helm이 chart를 렌더링할 때 template engine에 해당 폴더 내 template을 모두 전달한다.

### Template Files

### Predefined Values
values.yaml 파일 또는 --set flag를 통해 전달되는 변수는 teomplatd에서 .Values 객체를 통해 접근이 가능하다. 물론 template에서 사용 가능한 predefined 변수도 있다.

아래는 predefined 변수로 덮어쓰기가 불가능하다.

- Release.Name: The name of the release (not the chart)
- Release.Namespace: The namespace the chart was released to.
- Release.Service: The service that conducted the release.
- Release.IsUpgrade: This is set to true if the current operation is an upgrade or rollback.
- Release.IsInstall: This is set to true if the current operation is an install.
- Chart: The contents of the Chart.yaml. Thus, the chart version is obtainable as Chart.Version and the maintainers are in Chart.Maintainers.
- Files: A map-like object containing all non-special files in the chart. This will not give you access to templates, but will give you access to additional files that are present (unless they are excluded using .helmignore). Files can be accessed using {{ index .Files "file.name" }} or using the {{.Files.Get name }} function. You can also access the contents of the file as []byte using {{ .Files.GetBytes }}
- Capabilities: A map-like object that contains information about the versions of Kubernetes ({{ .Capabilities.KubeVersion }}) and the supported Kubernetes API versions ({{ .Capabilities.APIVersions.Has "batch/v1" }})


**Note**: 정의되지 않은 부적절한 Chart.yaml 내 필드는 무시된다. 그렇기 때문에 Chart 객체 내에서도 접근이 불가하다.

### Values files
--set, --values(-f) flag를 통해 전달된 변수 및 파일은 기본 chart에 포함된 values.yml 파일과 병합된다. chart의 기본 변수 파일인 values.yaml 파일이 버려지는 것이 아님에 주의해야 한다.

### Scope, Dependencies, and Values
최상위 chart에 사용되는 values.yaml 파일에는 charts/ 디렉토리에 포함된 sub chart에 변수도 정의할 수 있다. 아래 예시는 mysql, apache 종속성을 갖는 WordPress chart의 변수 파일이다:

``` yaml
title: "My WordPress Site" # Sent to the WordPress template

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

최상위 chart에서는 하위 chart의 변수에 접근할 수 있다(예를 들어 .Values.mysql.password). 하지만 반대로 하위 chart에서는 상위 chart의 변수에 접근할 수 없다. 뿐만 아니라 apache chart에도 접근할 수 없다. 하위 chart에서 변수를 사용하기 위해 .Values.mysql.password가 아닌 .Values.password와 같이 사용한다(namespace가 삭제된다).

### Global Values
2.0.0-Alpha.2부터 Helm은 global 변수를 지원한다.

``` yaml
title: "My WordPress Site" # Sent to the WordPress template

global:
  app: MyWordPress

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

모든 chart에서 .Values.global.app의 형태를 통해 global 변수를 사용할 수 있다. 이는 변수 파일이 다음과 같이 다시 생성된다고 이해하면 된다:

``` yaml
title: "My WordPress Site" # Sent to the WordPress template

global:
  app: MyWordPress

mysql:
  global:
    app: MyWordPress
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  global:
    app: MyWordPress
  port: 8080 # Passed to Apache
```

하위 chart에서 global 변수를 정의하는 경우에는 해당 하위 chart의 하위 chart에 global 변수가 공유되지만 상위 chart로는 공유되지 않는다. 하위 chart가 상위 chart에 영향을 줄 수 있는 방법은 없다. 뿐만 아니라 상위 chart의 global 변수가 하위 chart의 것보다 우선 순위가 높다.

### Schema Files
때때로 chart 작성자는 변수에 대한 구조를 정의하고자 한다. 이는 values.schema.josn 파일을 통해 가능하다. schema 파일은 변수에 대한 유효성 검사에 사용된다. 유효성 검사는 아래 명령어에서 수행된다:

- helm install
- helm upgrade
- helm lint
- helm template

schema는 values.yaml 파일이 아닌 최종 .Values 객체에 적용된다는 점을 유의해야 한다.

Furthermore, the final .Values object is checked against all subchart schemas. This means that restrictions on a subchart can't be circumvented by a parent chart. This also works backwards - if a subchart has a requirement that is not met in the subchart's values.yaml file, the parent chart must satisfy those restrictions in order to be valid.

### References
- Go templates
- Extra template functions
- The YAML format
- JSON Schema

## Custom Resource Definitions (CRDs)
k8s는 새로운 k8s object 타입을 정의할 수 있는 기능을 제공한다. CRD를 사용함으로써 k8s 개발자는 새로운 resource 타입을 명시할 수 있다.

Helm 3부터 CRD는 특별한 object 종류로 다뤄진다. helm install 시 가장 먼저 설치되며, 몇 가지 제약 사항이 있다.

CRD yaml 파일은 crds/ 디렉터리에 정의되어야 한다. 여러 CRD는 동일한 파일에 위치해야 한다. helm은 CRD 디렉터리의 모든 파일을 k8s에 로드한다.

CRD 파일은 template을 사용할 수 없으며 모두 평범한 yaml 파일이어야 한다.

helm이 새로운 chart를 설치할 때 k8s API server에 CRD를 로드하고 이용가능할 때까지 기다린다. 그리고 나서 나머지 chart를 렌더링하기 위해 template engine을 실행 및 k8s에 로드한다. 이러한 순서 보장 덕분에 Helm template에서 .Capabilities object를 통해 CRD 정보를 사용할 수 있으며, CRD를 통해 정의된 k8s resource 타입의 객체를 생성할 수 있다.

예를 들어, crds/ 디렉토리에 CronTab이라는 새로운 CRD를 생성하고, template 디렉터리에서 CRD에 대한 새로운 k8s reousrce 타입의 객체를 생성하는 경우:
```
crontabs/
  Chart.yaml
  crds/
    crontab.yaml
  templates/
    mycrontab.yaml

```

crontab.yaml 파일은 tempalte이 포함되지 않은 CRD 정의를 포함해야 한다:
``` yaml
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
```

그리고 mycrontab.yaml template 파일은 새로운 CronTab 객체를 생성한다:
``` yaml
apiVersion: stable.example.com
kind: CronTab
metadata:
  name: {{ .Values.name }}
spec:
   # ...
```

helm은 templates/ 디렉토리 내의 CRD를 먼저 k8s API server에 업로드함으로써 templates/ 디렉터리에 정의된 CronTab resource에 대한 생성을 보장한다.

### Limitations on CRDs
k8s의 object와 다르게 CRD는 global 설치된다. 이러한 이유로 CRD에 대한 관리 시 유의해야 하며 몇 가지 제약 사항도 있다:

- CRD는 재설치되지 않는다. 만약 crds/ 디렉터리의 CRD가 버전과 관계 없이 이미 존재하는 경우 Helm은 설치 / 업그레이드를 수행하지 않는다.
- CRD는 upgrace, rollback 시 설치되지 않는다. Helm은 install 시에만 CRD를 생성한다.
- CRD는 절대 삭제되지 않는다. CRD에 대한 삭제는 k8s 클러스터 내 CRD로 정의된 reosurce의 삭제이다. 그렇기 때문에 helm은 CRD를 삭제하지 않는다.
## Using Helm to Manage Charts

## Chart Repositories
chart repository는 패키지된 chart를 저장 및 제공하는 HTTP 서버다. helm 명령어는 로컬 chart 디렉토리를 관리하는 데 사용되기 때문에 chart에 대한 공유는 chart repository를 사용하는 것을 권장한다.

YAML, tar 파일을 제공하며 GET method에 대한 요청을 처리할 수 있는 HTTP 서버는 repository server로 사용이 가능하다. website 모드가 활성화된 GCS, S3에 대해 helm 팀에서 확인을 완료했다.

repo는 주로 패키지를 검색하고 확인할 수 있는 메타데이터와 함께 모든 패키지 목록에 대한 index.yml이라는 특수 파일이 있다는 특징이 있다.

클라이언트에서는 `helm repo` 명령어를 통해 관리한다. 하지만 helm은 chart를 원격 서버로 업로드하는 기능을 제공하지 않는다. 그렇게 하면 구현 서버에 상당한 요구 사항이 추가되어 repo 설정에 대한 장벽이 높아지기 때문이다.

## Chart Starter Packs
helm create의 --starter flag를 사용해 stater chart를 명시할 수 있다.

stater chart는 일반 chart와 동일하지만 $XDG_DATA_HOME/helm/starters에 위치한다. chart 개발자로서 특별히 디자인된 stater chart를 작성할 수도 있다. 이러한 chart는 아래 내용을 고려해 만들어야 한다:

- Chart.yaml 파일은 생성 시 덮어씌워진다.
- 사용자는 해당 chart의 내용을 변경할 것이기 때문에 사용자를 위한 가이드가 제공되어야 한다.
- starter chart를 template로 사용할 수 있도록 모든 \<CHARTNAME\>은 chart의 이름으로 대체된다.

햔재 $XDG_DATA_HOME/helm/starters에 chart를 추가하는 방법은 직접 손으로 하는 수밖에 없다. 