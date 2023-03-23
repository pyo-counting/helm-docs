# Helm
## Helm 학습
버전: v3.11.1

추가 정리 필요 내용

- chart hook
- chart test
- library chart
- chart provenance and integrity
- chart repository
- OCI-based chart registry
- advanced technique
- k8s rbac
- k8s CRD
- helm plugin

### 요약
- helm chart의 기본 구조를 나타내는 test/ 디렉토리 하위 구조는 다음과 같다.
    ```
    test/
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
- Helm은 각 설치 요청에 대해 새로운 Release를 생성함으로써 k8s에 Chart를 설치한다. 원하는 Chart는 archive 되어(*.tgz)로 Repository에 배포된다(Artifact Hub는 여러 repo들에 속한 chart를 관리한다).
- helm은 go의 template을 사용한다. go의 내장 함수, Spring library(env, expandenv 제외), include, required 함수를 사용할 수 있다.
- template action의 단점을 보완하기 위해 include 함수를 사용한다.
    - template action의 경우 결과 값을 파이프라이닝 할 수 없으며, template action의 인자로 변수를 사용할 수 없다.
- cm, secrets를 업데이트할 때 해당 리소스를 사용하는 po에 대한 재배포가 필요할 수 있다. 이를 위해 helm chart 내에서 편법을 사용해 po가 재배포될 수 있도록 할 수 있다.
- chart 이름은 소문자, 숫자로 구성된다. 이름 내 단어는 dash(-)로 구분된다. 대문자, underscore(_)도 chart 이름에 사용할 수 있지만 권장 사항은 아니다.
- template 파일 이름은 camelcase가 아닌 dash 표기법을 사용한다.
- templates/ 디렉토리에 underscore(_)로 시작하는 파일은 k8s manifest 파일로 간주되지 않는다. 관습에 따라 helper template, partials은 _heplers.tpl 파일(named template을 정의)에 작성된다. .yaml 접미사를 사용해 YAML 파일, .tpl 접미사를 사용해 helper를 나타내는 것을 권장한다.
- 최상위 chart의 values.yaml 파일에는 하위 chart의 변수의 값도 설정할 수 있다. global 변수도 설정이 가능하며 정의된 해당 chart와 하위 chart에서 `{{ .Values.global.변수이름 }}`와 같이 접근이 가능하다.
- helm 변수에 대한 우선순위는 values.yaml < 부모 chart의 values.yaml 파일 < --values < --set
- values.yaml 파일에는 각 변수에 대한 설명이 포함된 주석이 있어야 한다. 주석은 변수의 이름으로 시작해야 한다.
- YAML 주석은 helm install --debug 명령어 사용 시 조회가 되지만 template 주석은 조회가 되지 않으며 단지 template에 대한 설명을 위한 기능이다.
- values.yaml 파일 내 변수에 대해 \<key\>=null, yaml sequnce는 [],을 사용해 null 값과 빈값을 설정할 수 있다.
- chart value에 대해 명시적으로 모든 문자열은 quoting하는 것을 권장한다.
- 사용자 정의 변수 이름은 소문자로 시작해야 하며 단어는 camelcase를 사용해 구분한다. 내장 변수는 구분을 위해 대문자로 시작한다.
- `{{ define }}`을 이용해 생성된 template은 하위 chart에서도 접근이 가능하기 때문에 이름에 namespace를 가져야 한다. 동일한 이름의 template이 선언될 경우 마지막으로 로드된 template이 유효하다. subchart의 template도 상위 chart와 같이 렌더링되기 때문에 유의해야 한다.
- `{{ define }}`을 이용해 정의된 template에 대해서는 `{{/* */}}` 구문을 사용해 해당 named teplate에 대한 설명을 붙이는 것이 관습이다.
- helm create 명령어는 기본적으로 imagePullPolicy 필드를 IfNotPresent로 설정한다.
- template 파일 이름은 k8s resource의 종류를 파악할 수 있도록 해야 한다.
- 대부분의 변수는 values.yaml 파일에 기본값이 있으므로 `default` 함수를 사용할 필요가 없지만 values.yaml 파일에 정의될 수 없는 계산을 통해 값이 결정되는 경우에 대해서 유용하다.
- helm template, helm install | upgrade | delete | rollback --dry-run 명령어는 k8s API server와 연결하지 않기 때문에 `lookup` 함수에 대해 빈 목록을 반환한다.
- values.yaml 파일에는 여러 ---, ...을 사용해 여러 values.yaml 파일을 명시할 수 있지만 첫 번째 파일만 사용한다. template에도 사용 가능하지만 디버깅이 어렵기 때문에 권장하지 않는다.
- template 디버깅 방법
    - helm lint: best practice를 따르는지 확인하는 데 사용되는 도구
    - helm template --debug: template을 로컬에서 테스트 렌더링
    - helm install --dry-run --debug: 서버에서 template을 렌더링한 다음 결과 manifest 파일을 반환
    - helm install --disable-openapi-validation:
    - helm get manifest: 서버에 설치된 template을 조회
    - YAML 파싱이 실패했지만 생성된 내용을 확인하기 위한 방법은 문제가 발생하는 구문을 주석처리한 다음 helm install --dry-run --debug으로 다시 실행해보는 것이다:
- .helmignore 파일은 helm chart .tgz 패키징시 포함하고 싶지 않은 파일을 명시하는 데 사용할 수 있다.
- chart 내 templates/NOTES.txt 파일은 EADME.md 파일과 다르게 chart 설치 및 정보 조회 시, template으로 변환 및 줄력된다.
- templates/ 디렉터리 내 디렉터리를 추가해 여러 template를 구분해 관리하는 것도 가능하다.
- tempalte에서 eq, ne, lt, gt, and, or와 같은 연산자는 모두 함수로 구현된다. 파이프라인 내에서 `(`, `)`을 이용해 연산을 그룹화할 수 있다.
- YAML은 크게 collection, scalar 타입을 제공한다. collection 타입의 경우 map과 sequnce가 있다. 자세한 내용은 5_Chart Template Guide - 15_Appendix YAML Techniques.md 파일을 참고한다.
- template 내에서 변수는 다른 객체에 대한 이름있는 참조다. 예를 들어 name이라는 변수에 .Values 객체를 할당하고자 할 경우 `$name := .Values`와 같이 사용할 수 있다. name이라는 변수는 `{{ $name }}`과 같이 사용할 수 있다. 제어 구문 내에서 선언한 변수는 해당 구문 내에서만 유효하며, 제어 구문 밖에 정의된 변수는 template 파일에 정의된 것으로 다른 template에서도 접근이 가능하다. `$`은 root scope를 가리키는 변함 없는  값이다. `$`는 template 실행 시 root scope에 매핑되며 실행 기간 동안 변하지 않는다.
- template에서 제어 구문은 action이라고 부른다. `if / else if / else`, `with`, `range` action을 제공하며 추가적으로 template과 관련해 `define`, `template`, `block` action을 제공한다.
    - `if / else if / else` 구문은 아래의 경우에 대해 false로 판단하며 이외 모든 경우에 대해 true로 판단한다.
        - a boolean false
        - a numeric zero
        - an empty string
        - a nil (empty or null)
        - an empty collection (map, slice, tuple, dict, array)
    - `with` 구문을 사용해 해당 구문 내에서 변수 스코프를 변경할 수 있다. template 내에서는 기본적으로 root 스코프이므로 Values, Release와 같은 객체에 접근이 가능하지만 with 구문을 사용해 이를 변경할 수 있는 것이다. 스코프가 변경됐더라도 $를 사용해 root scope에 접근할 수 있다.
    - `range` 구문은 for-each 구문이다. range 구문에는 순회할 변수가 명시되는데, 이 때 변수의 index, key, value를 변수에 할당할 수 있다.
- `{{ }}` template 지시자의 경우 앞 또는 뒤  공백 제어를 위해 -문자를 사용할 수 있다. 이 때 개행 문자 역시 공백으로 간주되며 `{{- 3 }}`의 경우 왼쪽 공백을 모두 삭제, 오른쪽 공배는 삭제하지 않으며 3을 출력한다.
- 
    
### 명령어
- Global flag:
    - `--debug`: 자세한 출력
- `helm search`: ArtifactHub 또는 로컬에 추가된 repo를 대상으로 chart를 검색한다.
    - `hub`: Artifact Hub에서 chart를 검색한다.
    - `repo`: repo에서 chart를 검색한다.
- `helm install`: chart를 배포한다.
    - `--dry-run`: chart 설치를 시뮬레이션한다. k8s Open API Scheme에 렌더링된 template을 전송 및 유효성을 검증하며, k8s 객체를 생성하는 것은 아니다.
    - `--disable-openapi-validation`: 설치 프로세스 중 렌더링된 template을 k8s Open API Schema에 대해 유효성 검증을 하지 않는다.
- `helm get`:
    - `manifest`:
- `helm uninstall`: release를 삭제한다.
    - `--keep-history`: release history를 보존한다. 이 후 rollback이 가능하다.
- `helm pull`: chart install 없이 패키지만 다운로드 한다.
    - `--untar`: true로 설정할 경우, 다운로드 이후 untar로 수행한다.
- `helm upgrade`: release를 업그레이드한다.
    - `--install`: 업그레이드 대상 release가 없을 경우 helm install 명령어를 수행한다.
    - `--reset-values`: helm install 명령어에서 --set flag를 사용해 설정한 변수를 초기화한다.
    - `--timeout`: helm install/rollback 명령어에서도 사용. k8s 명령어가 완료될 때까지 helm client가 기다리는 시간(기본값: 5m)
    - `--wait`: helm install/rollback 명령어에서도 사용. release가 성공으로 표시하기 위해 모든 po가 ready state, pvc가 바운딩, deploy가 최소 po 개수를 만족(desired - maxUnavailable), svc가 IP를 보유(그리고 loadbalancer일 경우 ingress도 보유)할 때까지 기다린다. 이는 --timeout 값까지 기다린다. timeout에 도달하면 release는 FAILED로 표시된다.
- `helm status`: release의 상태를 조회한다.
- `helm lint`: 
- `helm template`: 로컬 환경에서 chart를 렌더링하고 결과를 출력한다.
- `helm repo`: repo 관리 명령어
    - `update`: 로컬 환경에 repo에 대한 데이터를 업데이트한다.