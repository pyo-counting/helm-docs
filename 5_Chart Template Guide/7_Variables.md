이전에 아래 코드가 실패됨을 확인했다.

``` yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}
  {{- end }}
```

왜냐하면 scope가 바뀐 with 구문 내에서 현재 접근이 불가능하기 때문이다. scoping 이슈와 관련해 해결를 위한 한 가지 방법은 현재 scope에 관계 없이 접근할 수 있는 변수애 객체를 할당하는 것이다.

helm template 에서 변수는 다른 객체에 대한 이름있는 참조다: `$name`. 변수는 `:=` 할당 연산자를 통해 할당된다. 위 예시를 아래와 같이 변수를 정의해 바꿀 수 있다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

with 구문 밖에서 정의한 변수 relname은 with 구문 내에서 `$name`을 이용해 접근할 수 있다.

변수는 range 루프 내에서 유용하다. 인덱스와 값을 조회하는 데 사용될 수 있다:

``` yaml
  toppings: |-
    {{- range $index, $topping := .Values.pizzaToppings }}
      {{ $index }}: {{ $topping }}
    {{- end }}    
```

위 코드 결과는 아래와 같다:

``` yaml
  toppings: |-
      0: mushrooms
      1: cheese
      2: peppers
      3: onions
```

key와 값을 조회하는 데 사용될 수도 있다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

변수는 "global"하지 않다. 변수는 선언된 구문 내에서 유효하다. 위에서 $relname은 template의 최상위에 정의됐기 때문에 전체 template에서 유효하다. 하지만 $key, $val은 range 구문 내에서만 유효하다.

예외적으로 `$`은 root context를 가리키는 유일한 변수로 어떤 구문에서 사용하더라도 동일하게 동작한다.

``` yaml
{{- range .Values.tlsSecrets }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .name }}
  labels:
    # Many helm templates would use `.` below, but that will not work,
    # however `$` will work here
    app.kubernetes.io/name: {{ template "fullname" $ }}
    # I cannot reference .Chart.Name, but I can do $.Chart.Name
    helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}"
    app.kubernetes.io/instance: "{{ $.Release.Name }}"
    # Value from appVersion in Chart.yaml
    app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
    app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
type: kubernetes.io/tls
data:
  tls.crt: {{ .certificate }}
  tls.key: {{ .key }}
---
{{- end }}
```