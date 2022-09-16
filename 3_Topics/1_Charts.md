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

LICENSE: chart의 license를 포함하는 일반 텍스트 파일이다. 

README.md: Markdown 포맷(README.md)을 사용해야 한다. 일반적으로 아래 내용을 포함한다.

- chart가 제공하는 애플리케이션, 서비스에 대한 설명
- chart를 실행하기 위해 필요한 사항
- values.yaml과 기본 값에 대한 설명
- chart의 설치, 설정과 관련된 다른 정보

artifacthub와 같은 사용자 인터페이스 환경에서 chart에 대한 세부 사항은 READMD.md 내용을 보여준다.

templates/NOTES.txt: chart 설치, release의 상태 조회 시 출력되는 정보를 포함하는 파일. 이 파일은 template으로 간주된다. 보통 사용 참고 사항, 다음 단계, 기타정보를 표시하는 데 사용될 수 있다. 예를 들어 DB 연결, 웹 UI 접근에 대한 방법을 제공할 수 있다. 이 파일은 `helm install`, `helm status` 실행 시 표준 출력(STDOUT)으로 출력된다. 간략한 내용을 포함하면서 자세한 내용은 README에 포함하는 것을 권장한다.

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

## Templates and Values

## Custom Resource Definitions (CRDs)

## Using Helm to Manage Charts

## Chart Repositories
repo는 주로 패키지를 검색하고 확인할 수 있는 메타데이터와 함께 모든 패키지 목록에 대한 index.yml이라는 특수 파일이 있다는 특징이 있다.

클라이언트에서는 `helm repo` 명령어를 통해 관리한다. 하지만 helm은 chart를 원격 서버로 업로드하는 기능을 제공하지 않는다. 그렇게 하면 구현 서버에 상당한 요구 사항이 추가되어 repo 설정에 대한 장벽이 높아지기 때문이다.

## Chart Starter Packs