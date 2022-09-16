## Structure of templates/

- templates/ 디렉토리는 아래 구조를 가져야한다:
- template 파일의 확장자는 YAML 형식을 출력하는 경우 확장자가 .yaml이어야 한다. .tpl 확장자는 형식이 지정된 내용을 생성하지 않는 template 파일에 사용할 수 있다.
- template 파일 이름은 camelcase가 아닌 dash 표기법(예를 들어 my-example-configmap.yaml)을 사용해야 한다.
- 각 resource 정의는 자체 template 파일을 가져야 한다.
- template 파일 이름은 resource의 종류를 파악할 수 있도록 해야 한다. 예를 들어 foo-pod.yaml, bar-svc.yaml

## Names of Defined Templates
defined template( {{ define }}을 이용해 생성된 template)은 전역적으로 접근 가능하다. 즉, chart와 모든 sub-chart에서 접근이 가능하다.

이러한 이유로 모든 defined template 이름은 네임스페이스를 갖도록 해야 한다.

올바른 예:

``` yaml
{{- define "nginx.fullname" }}
{{/* ... */}}
{{ end -}}
```

나쁜 에:

``` yaml
{{- define "fullname" -}}
{{/* ... */}}
{{ end -}}
```

It is highly recommended that new charts are created via helm create command as the template names are automatically defined as per this best practice.

## Formatting Templates
template은 공백 문자 2개로 들여쓰기해야 한다.

template 지시자는 여는 괄호 뒤에 공백 문자를 있어야 하며 닫는 괄호 앞에 공백 문자가 있어야 한다.

올바른 예:

``` yaml
{{ .foo }}
{{ print "foo" }}
{{- print "bar" -}}
```

잘못된 예:

``` yaml
{{.foo}}
{{print "foo"}}
{{-print "bar"-}}
```

template은 가능한 한 공백을 줄여야 한다.

``` yaml
foo:
  {{- range .Values.items }}
  {{ . }}
  {{ end -}}
```

블록은 template 코드의 흐름을 나타내기 위해 들여쓸 수 있다.

``` yaml
{{ if $foo -}}
  {{- with .Bar }}Hello{{ end -}}
{{- end -}}
```

그러나 YAML은 공백 지향 언어로 코드 들여쓰기가 해당 규칙을 지키는 것이 불가능한 경우가 많다.

## Whitespace in Generated Templates
생성된 template의 공백을 최소로 유지하는 것이 좋다. 특히 다수의 빈 줄이 서로 인접하도록 생성되는 것은 좋지 않다. 그러나 때때로(특히 논리적 섹션 사이) 빈 줄은 괜찮다.

가장 좋은 예:

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example
  labels:
    first: first
    second: second
```

좋은 예:

``` yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: example

  labels:
    first: first
    second: second
```

좋지 않은 예:

``` yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: example





  labels:
    first: first

    second: second
```

## Comments (YAML Comments vs. Template Comments)
YAML, helm template 모두 주석이 있다.

YAML 주석:

``` yaml
# This is a comment
type: sprocket
```

template 주석:

``` yaml
{{- /*
This is a comment.
*/}}
type: frobnitz
```

template 주석은 template 기능을 설명해야 한다.

``` yaml
{{- /*
mychart.shortname provides a 6 char truncated version of the release name.
*/}}
{{ define "mychart.shortname" -}}
{{ .Release.Name | trunc 6 }}
{{- end -}}
```

template YAML 내에서 주석은 helm 사용자가 디버깅 하는데 도움이 될 수 있도록 사용한다.

``` yaml
# This may cause problems if the value is more than 100Gi
memory: {{ .Values.maxMem | quote }}
```

위 주석은 helm install --debug에 의해 볼 수 있지만 {{- /* */}} 내 주석은 그렇지 않다.

## Use of JSON in Templates and Template Output