## Know Your Template Functions
helm은 resource 파일에 Go template를 사용한다. go의 내장 함수 뿐만아니라 다른 함수들도 제공한다.

보안 상 이유로 `env`, `expandenv`를 제외한 [Sprig library](https://masterminds.github.io/sprig/)의 모든 함수를 추가했다.

뿐만 아니라 `include`, `required` template 함수를 추가했다.

- `include`: 다른 template를 가져올 수 있으며 다른 함수의 매개변수로 사용할 수 있다.
- `requied`: template 렌더링 시 필수 값 변수로 명시할 수 있다. 변수에 대해 빈값을 사용자가 사용하면 함수에 파라미터로 전달한 오류 메시지와 함께 template 렌더링은 실패한다.

## Quote Strings, Don't Quote Integers
문자열 데이터 사용 시, 항상 quoting하는 것이 안전하다.

``` yaml
name: {{ .Values.MyName | quote }}
```

정수 값을 사용할 때는 quoting하지 않는 것이 좋다. 그렇지 않을 경우 대부분의 경우 k8s에서 파싱 에러를 야기한다.

This remark does not apply to env variables values which are expected to be string, even if they represent integers:

``` yaml
env:
  - name: HOST
    value: "http://host"
  - name: PORT
    value: "1234"
```

## Using the 'include' Function
Go는 내장 `template` 지시자를 사용해 template에 다른 template을 포함할 수 있다. 하지만 내장 함수는 Go template pipeline에서 사용할 수 없다.

template을 포함하고 추가 동작을 수행하기 위해 helm은 특별한 `include` 함수를 제공한다.

## Using the 'required' function
`required` 함수를 사용해 template 렌더링 시 변수 값을 필수로 선언할 수 있다. 해당 변수 값이 values.yml에서 빈 값이라면 template은 렌더링하지 않고 에러 메시지를 반환한다.

``` yaml
{{ required "A valid foo is required!" .Values.foo }}
```

## Using the 'tpl' Function
`tpl` 함수를 사용해 template 내에서 문자열을 template으로 평가할 수 있다. chart에 template 문자열을 값으로 전달하거나 외부 설정 파일을 렌더링할 때 유용하다. 문법: `{{ tpl TEMPLATE_STRING VALUES }}`

예시:

``` yaml
# values
template: "{{ .Values.name }}"
name: "Tom"

# template
{{ tpl .Values.template . }}

# output
Tom
```

외부 설정 파일 렌더링 예시:
``` yaml
# external configuration file conf/app.conf
firstName={{ .Values.firstName }}
lastName={{ .Values.lastName }}

# values
firstName: Peter
lastName: Parker

# template
{{ tpl (.Files.Get "conf/app.conf") . }}

# output
firstName=Peter
lastName=Parker
```

## Creating Image Pull Secrets
image pull secret은 registry, unsername, password의 조합으로 이루어진다. 이러한 정보는 배포하는 애플리케이션 내부에서 필요할 수 있으며 이러한 정보를 생성하기 위해 base64 인코딩이 필요하다. 아래는 secret resource에서 사용할 docker 설정 파일을 구성하는 helper template에 대한 예시다:

values.yaml
``` yaml
imageCredentials:
  registry: quay.io
  username: someone
  password: sillyness
  email: someone@host.com
  ```

helper template
``` yaml
{{- define "imagePullSecret" }}
{{- with .Values.imageCredentials }}
{{- printf "{\"auths\":{\"%s\":{\"username\":\"%s\",\"password\":\"%s\",\"email\":\"%s\",\"auth\":\"%s\"}}}" .registry .username .password .email (printf "%s:%s" .username .password | b64enc) | b64enc }}
{{- end }}
{{- end }}
```

secret resource manifest
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "imagePullSecret" . }}
```

## Automatically Roll Deployments
cm, secret을 container에서 사용할 경우 po에 대한 rolling이 필요할 수 있다. `helm uprage` 명령어 사용 시 일부 애플리케이션은 재시작이 필요할 수 있지만 deploy `.spec`에 대한 변경 사항이 없어 애플리케이션 po가 재시작되지 않을 수 있다.

이러한 경우 `sha256sum` 함수를 사용해 deploy의 annotation이 업데이트 될 수 있도록 보장할 수 있다.

``` yaml
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
[...]
```

**Note***: 이를 library chart에 추가할 경우 `$. Template.BasePath`를 이용해 원하는 파일에 접근하지 못할 수 있다. 이 때 대신 `{{ include ("mylibchart.configmap") . | sha256sum }}`를 사용한다.

항상 deploy를 rolling하기 위해서는 위와 유사하게 annotation을 사용할 수 있다:

``` yaml
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        rollme: {{ randAlphaNum 5 | quote }}
[...]
```

template 함수를 호출할 때마다 임의 문자열이 생성된다. 즉 여러 resource에서 사용하는 임의 문자열을 동기화해야 하는 경우 모든 관련 resource를 동일 tempalte 파일에 작성해야 한다.

**Note**: 이전에는 --recreate-pods 옵션을 사용해 위 동작을 수행했다. 하지만 helm 3에서 해당 flag는 depreacte다.

## Tell Helm Not To Uninstall a Resource
`helm uninstall` 명령어 사용 시 실제로 uninstall되면 안되는 resource가 있을 수 있다. 이는 annotation을 추가해 방지할 수 있다.

예시:

``` yaml
kind: Secret
metadata:
  annotations:
    "helm.sh/resource-policy": keep
[...]
```

(quoting은 필수다.)

`"helm.sh/resource-policy": keep` annotation은 helm이 resource를 삭제하는 것을 방지한다(`helm uninstall`, `helm upgrade`, `helm rollbak` 명령어 등). 이러한 resource는 helm이 더 이상 관리할 수 없다. uninstall 됐지만 위와 같이 유지되는 resource가 있을 때 `helm install --replace` 명령어를 사용하면 문제가 발생할 수도 있다.

## Using "Partials" and Template Includes
chart에서 재사용 가능한 코드를 생성하기 원할 수 있다.

templates/ 디렉토리에 underscore(_)로 시작하는 파일은 k8s manifest 파일로 간주되지 않는다. 관습에 따라 helper template, partials은 _heplers.tpl 파일에 작성된다.

## Complex Charts with Many Dependencies
대규모 애플리케이션을 위해 한 chart에 여러 하위 chart를 포함할 수 있다.

여러 하위 chart를 포함하는 상위 chart를 구성하는 모범 사례는 상위 chart를 만든 후에 charts/ 디렉토리에 여러 하위 chart를 포함시키는 것이다.

## YAML is a Superset of JSON
YAML은 JSON의 부모 집합이다. 즉, 모든 유효한 JSON 구조는 YAML에서도 유효하다.

공백을 이용한 파싱을 수행하는 YAML 대신 JSON을 사용해 데이터 구조를 표현하는 것이 더 용이할 수 있다.

하지만 위와 같은 경우가 아니라면 YAML을 사용하는 것을 권장한다.

## Be Careful with Generating Random Values
helm에는 랜덤 데이터, 암호화 키를 생성할 수 있는 함수가 있다. 이러한 함수를 사용할 경우 chart upgrade 시 의도치 않게 resource에 대한 재배포를 야기할 수 있음을 주위해야 한다.

## Install or Upgrade a Release with One Command
helm은 install 또는 upgrade를 수행할 수 있는 명령어를 제공한다. `helm upgrade --install` 명령어는 helm이 release가 install 됐는지 여부에 따라 install 또는 uprage를 수행한다.

``` bash
helm upgrade --install <release name> --values <values file> <chart directory>
```