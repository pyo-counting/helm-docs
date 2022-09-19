template 함수는 `functionName arg1 arg2 ...`와 같은 형식으로 사용한다.

helm은 60개가 넘는 함수를 지원한다. 이 중 일부는 Go template language에 정의됐으며, 이외 대부분은 Sprig template library에 정의됐다.

> helm template language를 helm 내 전용인 것처럼 보일 수 있지만 실제로는 Go template language, 일부 추가 함수, template에 특정 객체를 노출하는 다양한 wrapper의 조합으로 이루어졌다. Go template은 template에 대해 배울 때 매우 유용하다.

## Pipelines
파이프라인(|)은 UNIX에서의 개념과 유사하다. 파이프라인을 이용해 여러 작업을 순차적으로 수행할 수 있다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
```

위 예시에서는 quote 함수에 대해 quote ARGUMENT 대신 파이프라인을 사용해 quote 함수에 매개변수를 전달했다.

> 순서를 변경하는 것은 template에서 일반적인 관행이다. `quote .val` 형식보다 `.val | quote` 형식을 더 자주 사용한다.

파이프라인은 왼쪽 결과를 오른쪽 함수의 마지막 매개변수로 전달한다.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | repeat 5 | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
```

repeat 함수는 첫 번째 매개변수로 반복회수, 두 번째 매개변수로 반복할 문자열을 입력받는다.

## Using the default function

`default` 함수는 template에서 자주 사용된다: `default DEFAULT_VALUE GIVEN_VALUE`. 이 함수를 사용해 template 내에서 기본 값을 명시할 수 있다.

실제 chart에서 모든 정적 기본 변수 값은 values.yaml 내에 정의돼야 하며 default 명령어를 통해 중복 사용할 필요가 없다. 대신 values.yaml 파일 내에서 선언할 수 없는 계산된 값을 사용하는 변수에 적합하다.

``` yaml
drink: {{ .Values.favorite.drink | default (printf "%s-tea" (include "fullname" .)) }}
```

## Using the lookup function
`lookup` 함수는 동작중인 클러스터 내 resource에 대해 조회할 수 있다: `lookup apiVersion, kind, namespace, name`


|parameter  |type  |
|-----------|------|
|apiVersion |string|
|kind       |string|
|namespace  |string|
|name       |string|

name, namespace은 필수 파라미터가 아니면 ""와 같인 빈 문자열을 이용해 생략할 수 있다.


|Behavior                               |Lookup function|
|---------------------------------------|---------------|
|kubectl get pod mypod -n mynamespace   |lookup "v1" "Pod" "mynamespace" "mypod"|
|kubectl get pods -n mynamespace        |lookup "v1" "Pod" "mynamespace" ""|
|kubectl get pods --all-namespaces      |lookup "v1" "Pod" "" ""|
|kubectl get namespace mynamespace      |lookup "v1" "Namespace" "" "mynamespace"|
|kubectl get namespaces                 |lookup "v1" "Namespace" "" ""|

lookup 함수는 객체를 반환하면 dictionary를 반환한다.

아래는 mynamespace 객체의 annotation을 반환한다.

``` yaml
(lookup "v1" "Namespace" "" "mynamespace").metadata.annotations
```

lookup 함수가 객체 목록을 반환할 떄 items 필드를 이용해 객체 목록에 접근할 수 있다.

``` yaml
{{ range $index, $service := (lookup "v1" "Service" "mynamespace" "").items }}
    {{/* do something with each service */}}
{{ end }}
```

어떤 객체도 찾지 못하면 빈 값이 반환된다. 이는 객체의 존재 유무를 확인하는 데 사용할 수 있다.

lookup 함수는 helm의 k8s 연결을 이용해 k8s에 쿼리한다. API server와 상호작용하는 데 오류가 반환된다면 helm의 template 프로세싱 과정은 실패한다.

helm template, helm install | upgrade | delete | rollback --dry-run 함수는 k8s API server와 연결하지 않기 때문에 lookup 함수에 대해 빈 목록을 반환한다.

## Operators are functions
tempalte에서 q, ne, lt, gt, and, or와 같은 연산자는 모두 함수다. 파이프라인 내에서 `(`, `)`을 이용해 작업을 그룹화할 수 있다.