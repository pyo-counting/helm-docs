# Helm
## Helm 학습

### 요약
- Helm은 각 설치 요청에 대해 새로운 Release를 생성함으로써 k8s에 Chart를 설치한다. 원하는 Chart는 패키지 형태로(*.tgz)로 Repository에 배포된다.
- helm은 go의 template을 사용한다. go의 내장 함수, Spring library(env, expandenv 제외), include, required 함수를 사용할 수 있다.
- template 함수의 단점을 보완하기 위해 include함수를 사용한다.
- cm, secrets를 업데이트할 때 해당 리소스를 사용하는 po에 대한 재배포가 필요할 수 있다. 이를 위해 helm chart 내에서 편법을 사용해 po가 재배포될 수 있도록 할 수 있다.
- templates/ 디렉토리에 underscore(_)로 시작하는 파일은 k8s manifest 파일로 간주되지 않는다. 관습에 따라 helper template, partials은 _heplers.tpl 파일에 작성된다.
- 최상위 chart의 values.yaml 파일에는 하위 chart의 변수의 값도 설정할 수 있다.
- global 변수도 설정이 가능하며 정의된 chart와 하위 chart에서 접근이 가능하다.
- chart value에 대해 명시적으로 모든 문자열은 quoting한다.
- 사용자 정의 변수 이름은 소문자로 시작해야 하며 단어는 camelcase를 사용해 구분한다. 내장 변수는 구분을 위해 대문자로 시작한다.
- chart 이름은 소문자, 숫자로 구성된다. 이름 내 단어는 dash(-)로 구분된다.
- template 파일 이름은 camelcase가 아닌 dash 표기법을 사용한다.
- YAML 주석은 helm install --debug 명령어 사용 시 조회가 되지만 template 주석은 조회가 되지 않으며 단지 template에 대한 설명을 위한 기능이다.
- helm create 명령어는 기본적으로 imagePullPolicy 필드를 IfNotPresent로 설정한다.

### 명령어
- `helm uninstall`: release를 삭제한다.
    - `--keep-history`: release history를 보존한다. 이 후 rollback이 가능하다.
- `helm pull`: chart install 없이 패키지만 다운로드 한다.
    - `--untar`: true로 설정할 경우, 다운로드 이후 untar로 수행한다.
- `helm upgrade`: release를 업그레이드한다.
    - `--install`: 업그레이드 대상 release가 없을 경우 helm install 명령어를 수행한다.