## Know Your Template Functions
helm은 resource 파일에 Go template를 사용한다. go의 내장 함수 뿐만아니라 다른 함수들도 제공한다.

보안 상 이유로 `env`, `expandenv`를 제외한 [Sprig library](https://masterminds.github.io/sprig/)의 모든 함수를 추가했다.

뿐만 아니라 `include`, `required` template 함수를 추가했다.

- `include`: 다른 template를 가져올 수 있으며 다른 함수의 매개변수로 사용할 수 있다.
- `requied`: template 렌더링 시 필수 값 변수로 명시할 수 있다. 변수에 대해 빈값을 사용자가 사용하면 함수에 전달된 오류 메시지와 함께 template 렌더링은 실패한다.

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
Go는 내장 template 지시자를 사용해 template에 다른 template을 포함할 수 있다. 하지만 내장 함수는 Go template pipeline에서 사용할 수 없다.

template을 포함하고 추가 동작을 수행하기 위해 helm은 특별한 include 함수를 제공한다.

## Using the 'required' function

## Using the 'tpl' Function

## Creating Image Pull Secrets

## Automatically Roll Deployments

## Tell Helm Not To Uninstall a Resource

## Using "Partials" and Template Includes

## Complex Charts with Many Dependencies

## YAML is a Superset of JSON

## Be Careful with Generating Random Values

## Install or Upgrade a Release with One Command