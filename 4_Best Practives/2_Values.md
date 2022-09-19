변수 사용에 대해 설명한다. chart의 values.yaml 파일을 중심으로 권장 사항을 설명한다.

## Naming Conventions
변수 이름은 소문자로 시작해야 하며 단어는 camelcase를 사용해 구분한다.

올바른 예:

``` yaml
chicken: true
chickenNoodleSoup: true
```

부적절한 예:

``` yaml
Chicken: true  # initial caps may conflict with built-ins
chicken-noodle-soup: true # do not use hyphens in the name
```

helm의 내장 변수는 사용자 정의 변수와 구분하기 쉽기 위해 대문자로 시작한다.

## Flat or Nested Values
대부분의 경우 중첩보다 flat 구조이 선호된다. 그 이유는 template 개발자와 사용자에게 더 간단하기 때문이다.

안전을 위해 모든 중첩에 대해 변수를 검사해야 한다:

``` yaml
{{ if .Values.server }}
  {{ default "none" .Values.server.name }}
{{ end }}
```

중첩에 대해 존재 유무를 확인해야 한다. 그러나 flag 구조에서는 이러한 검사가 필요없으므로 template을 더 쉽게 읽고 사용하기 쉽다.

``` yaml
{{ default "none" .Values.serverName }}
```

관련된 변수가 많고 그 중 하나 이상 옵션이 아닌 경우 가독성을 높이기 위해 중첩을 사용할 수 있다.

## Make Types Clear
YAML의 형식 강제 변환 규칙은 때때로 직관적이지 않다. 예를 들어 foo: false는 foo: "false"와 다르다. foo: 12345678과 같은 큰 정수는 경우에 따라 scientific notation으로 변환된다.

타입 변환 오류를 피하는 가장 쉬운 방법은 문자열에 대해서는 명시적으로, 다른 모든 것에 대해서는 암시적으로 하는 것이다. 또는 간단하게 모든 문자열을 quoting하면 된다.

종종 정수 캐스팅 문제를 피하기 위해 정수를 문자열로 저장하고 template에서 {{ int $value }}를 사용해 문자열을 다시 정수로 변환하는 것이 유리할 수 있다.

대부분의 경우 명시적 타입 태그가 존중되므로 foo: !!string 1234는 1234를 문자열로 처리해야 한다. 그러나 YAML 파서는 태그를 사용하므로 한 번의 파싱 후에 데이터 타입이 없어진다.

## Consider How Users Will Use Your Values
변수에 대한 소스는 3가지가 있다.

- chart의 values.yaml 파일
- helm install -f, helm upgrade -f 명령어에 제공되는 변수 파일
- helm install, helm upgrade 명령어에 --set, --set-string flag에 제공되는 변수

변수의 구조를 디자인할 때 사용자가 -f, --set flag를 이용해 사용자가 변수 값을 덮어쓸 수 있다는 것을 고려해야 한다.

--set은 표현에 있어 제한적이므로 values.yaml 파일 작성에 대한 첫 번째 가이드라인은 --set에서 쉽게 덮어쓸 수 있도록 하는 것이다.

위와 같은 이유로 map을 사용하는 것이 더 좋다.

부적절한 예:

``` yaml
servers:
  - name: foo
    port: 80
  - name: bar
    port: 81
```

위 values.yaml파일에 대해 helm <=2.4일 경우 --set을 이용해 덮어쓸 수 없다. helm 2.5부터는 --set servers[0].port=80과 같이 덮어쓸 수 있다. 이러한 표현식은 가능은 하지만 사용자가 파악하기 어렵다.

좋은 예:

``` yaml
servers:
  foo:
    port: 80
  bar:
    port: 81
```

## Document values.yaml
values.yaml 파일 내 프로퍼티는 주석이 있어야 한다. 주석은 프로퍼티의 이름으로 시작해야 하며 설명을 위해 최소 한 문장을 포함해야 한다.

올바른 예:

``` yaml
# the host name for the webserver
serverHost: example
serverPort: 9191
```

부적절한 예:

``` yaml
# serverHost is the host name for the webserver
serverHost: example
# serverPort is the HTTP listener port for the webserver
serverPort: 9191
```

주석은 프로퍼티 이름으로 시작해야 쉽게 찾을 수 있다.