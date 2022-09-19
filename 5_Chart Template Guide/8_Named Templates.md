여기서는 named template을 선언하고 사용하는 방법에 대해 설명한다. named template(partial, subtemplate이라고도 불림)은 파일 내 정의되며 이름을 갖는다.

templating 작명 시 주의할점: template 이름은 global이다. 동일한 이름의 template을 선언할 경우 마지막으로 로드된 것이 사용된다. subchart 내 template도 상위 template 레벨에서 같이 컴파일되기 때문에 주의해야 한다.

널리 사용되는 작명 규칙 중 하나는 chart 이름을 접두사로 사용하는 것이다: `{{ define "mychart.labels" }}`. chart 이름을 접두사로 사용하면 동일한 이름의 template을 구현하는 두 개의 다른 chart로 인해 발생할 수 있는 충돌을 피할 수 있다.

## Partials and _ files
helm template language은 다른 곳에서 이름으로 접근할 수 있는 named template을 생성할 수 있다.

아래는 template 작명 시 규칙이다:

- templates/ 디렉터리 내 대부분의 파일은 k8s manifest가 포함된 것처럼 다뤄진다.
- NOTES.txt 파일은 예외다
- _로 시작하는 파일은 manifest 내용이 없는 파일로 가정한다. 이 파일은 k8s 객체 정의를 위해 렌더링되지 않으며 다른 chart template에서 사용할 수 있다.

이러한 파일은 partials, helpers를 저장하는 데 사용된다.

## Declaring and using templates with define and template
define action은 template 파일 내에서 named template를 생성한다. 구문은 다음과 같다:

``` yaml
{{- define "MY.NAME" }}
  # body of template here
{{- end }}
```

예시:

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

When the template engine reads this file, it will store away the reference to mychart.labels until template "mychart.labels" is called. Then it will render that template inline. So the result will look like this:

**Note**: define은 template으로 호출되지 않는 한 출력을 생성하지 않는다.

일반적으로 위와 같은 template을 partials 파일(일반적으로 _helpers.tpl)에 저장한다.

define 함수는 관련 설명을 주석(`{{/* ... */}}`)을 통해 설명해야 한다.

## Setting the scope of a template
위 named template 내에서 객체를 통해 변수의 값을 이용해본다:

``` yaml
{{/* Generate basic labels */}}
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```

위 named template를 사용하면 아래 오류가 발생한다.

``` bash
$ helm install --dry-run moldy-jaguar ./mychart
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: [unknown object type "nil" in ConfigMap.metadata.labels.chart, unknown object type "nil" in ConfigMap.metadata.labels.version]
```

렌더링 결과를 확인하기 위해 --disable-openapi-validation flag를 추가한다. 결과는 다음과 같다:

``` yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: moldy-jaguar-configmap
  labels:
    generator: helm
    date: 2021-03-06
    chart:
    version:
```

template action 사용 시 어떠한 scope도 전달하지 않았기 때문에 접근이 불가능한 것이다. 이를 위해 아래와 같이 변경하면 된다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
```

## The include function
template action은 함수가 아니기 때문에 결과 값을 다른 함수의 매개변수로 전달할 수 없다.

이를 해결하기 위해 helm은 template 내용을 파이프라인의 다른 함수로 전달할 수 있는 대안을 제공한다: `include` 함수

> helm template 내에서 `template` 대신 `include`의 사용을 선호한다. 왜냐하면 출력에 대한 포맷을 변경하기 쉽기 때문이다.