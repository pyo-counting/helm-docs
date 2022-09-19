chart는 subchart라고 불리는 종속성을 가질 수 있으며 subchart 역시 template, 변수를 가질 수 있다.

subchart와 관련해 몇 가지 중요 사항이 있다.

1. subchart는 "stand-alone"이기 때문에 부모 chart에 의존하면 안된다.
2. 그렇기 때문에 subchart는 부모의 변수에 접근할 수 없다.
3. 부모 chart는 subchart의 변수를 덮어쓸 수 있다.
4. helm은 모든 chart에서 접근할 수 잇는 global 변수의 개념을 갖는다.

## Creating a Subchart

## Adding Values and a Template to the Subchart

## Overriding Values from a Parent Chart
부모 chart에서 subchart에 대한 변수를 설정할 수 있다. 아래는 values.yaml의 예시다(부모 chart는 mysubchart를 자식으로 갖는다):

``` yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

mysubchart:
  dessert: ice cream
```

아래 mysubchart는 자식 subchart로 전달된다. 이는 subchart의 값을 덮어쓴다. subchart에서는 .Values.mysubchart.desert가 아닌 .Values.desrt로 접근한다. tempalte 엔진이 값을 전달할 때 scope를 설정한다.

## Global Chart Values

## Sharing Templates with Subcharts
부모 chart와 subchart는 template를 공유할 수 있다. 모든 chart에 정의된 블록은 다른 chart에서 사용할 수 있다.

``` yaml
{{- define "labels" }}from: mychart{{ end }}
```

위 named chart는 어떤 chart에서도 접근할 수 있다.

chart 개발자는 include 함수, template action을 사용할 수 있다. include의 장점은 template을 동적으로 참조할 수 있다는 것이다.

``` yaml
{{ include $mytemplate }}
```

template action은 오로지 문자열을 통해 참조할 수 있다.

## Avoid Using Blocks
The Go template language provides a block keyword that allows developers to provide a default implementation which is overridden later. In Helm charts, blocks are not the best tool for overriding because if multiple implementations of the same block are provided, the one selected is unpredictable.

The suggestion is to instead use include.