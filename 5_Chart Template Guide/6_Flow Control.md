제어 구문(template에서는 "action"으로 부름)은 template 작성자에게 template 생성 흐름을 제어할 수 있는 기능을 제공한다. helm의 template 언어는 다음과 같은 제어 구문을 제공한다.

- `if / else`: 조건 구문 생성
- `with`: 스코프 명시
- `range`: "for each" 스타일의 루프를 제공

추가적으로 template 세그먼트를 선언하고 사용하기 위해 몇 가지 action을 더 제공한다:

- `define`: template 내에서 새로운 이름 있는 template을 선언
- `template`: 이름 있는 template을 import
- `block`: declares a special kind of fillable template area

## If/Else
기본 구조는 아래와 같다.

``` yaml
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

위 if 문 내 조건식에서는 변수가 아닌 PIPELINE을 사용한다. 제어 구문에서는 단순히 변수 뿐만 아니라 전체 파이프라인을 사용할 수 있다.

변수가 아래와 같은 값일 경우 파이프라인은 false로 판단된다:

- a boolean false
- a numeric zero
- an empty string
- a nil (empty or null)
- an empty collection (map, slice, tuple, dict, array)

## Controlling Whitespace
template 엔진은 `{{`, `}}` 내 모든 공백을 제거하지만 이외 공백은 제거하지 않는다.

YAML은 공백에 의미를 부여하므로 공백을 제어하는 것은 매우 중요하다. helm template에는 이를 위한 몇 가지 기능이 있다.

첫 번째로 template 선언의 중괄호 구문 내 특수문자를 사용해 template 엔진에게 공백을 제거하도록 지시할 수 있다: `{{- `(dash와 공백 추가)은 왼쪽 공백을 삭제하도록 ` -}}`은 오른쪽 공백을 삭제하도록 한다. 개행 문자도 공백으로 인식됨을 인지해야 한다.

> 공백이 추가됨을 인지해야 한다. `{{- 3 }}`은 왼쪽 공백을 삭제하고 3을 출력하지만 `{{ -3 }}`은 -3을 출력한다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" }}
  mug: "true"
  {{- end }}
```

위 예시에서 삭제되는 공백은 *로 표시하면 아래와 같다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}*
**{{- if eq .Values.favorite.drink "coffee" }}
  mug: "true"*
**{{- end }}
```

> template 내 공백 제어와 관련해 자세한 내용은 [Official Go template documentation](https://pkg.go.dev/text/template?utm_source=godoc)을 참고한다.

## Modifying scope using with
`with`는 변수 scoping에 사용된다. .를 이용한 참조는 현재 scope에 대한 참조임을 인지해야 한다. 그러므로 .Values는 현재 scope에서 Values 객체를 호출하는 것이다.

`with` 구조는 `if / else`와 비슷하다.

``` yaml
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

with 구문을 사용해 현재 scope(.)를 다른 객체로 설정할 수 있다.

주의할 점은 이렇게 scope가 변경됐을 때 .을 이용해 상위 객체에 접근할 수 없다. 그렇기 때문에 위 예시는 오류가 발생한다.

``` yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}
  {{- end }}
```

위 예시에서 현재 scope에서 Release 객체를 찾을 수 없기 때문에 오류가 발생한다.

`$`을 이용해 부모 scope에 접근할 수 있다. `$`는 template 실행 시 root scope에 매핑되며 실행 기간 동안 변하지 않는다. 즉 위 예시는 아래와 같이 변경할 수 있다.

``` yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $.Release.Name }}
  {{- end }}
```

## Looping with the range action
다른 언어의 for, foreach 루프와 같이 helm template 에서는 range 구문을 제공한다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  toppings: |-
    {{- range $.Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}    
  {{- end }}
```

> YAML 파일에서 `|-`는 multi-line 문자열을 취한다. k8s 내 cm에서 `.data`에서는 이를 사용해 여러 데이터에 대한 key-value 쌍을 인식한다.

helm template 내에서는 `tuple` 함수를 제공한다. `tuple` 함수는 고정된 크기의 list와 유사한 collection이지만 임의 데이터 타입을 사용한다. 아래는 예시다:

``` yaml
  sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
```

list, typle 외에도 key-value(map 또는 dict)이 잇는 collection을 range에 사용할 수 있다.