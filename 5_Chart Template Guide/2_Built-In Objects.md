객체는 template 엔진에서 template으로 전달된다. 그리고 코드 내에서 객첼르 전달할 수도 있다(`with`, `range` 구문을 이용해). `tuple` 함수를 이용해서 template 내에서 객체를 생성할 수도 있다.

객체는 단순히 1개의 변수를 갖거나 다른 객체 또는 함수를 포함할 수도 있다.

아래는 최상위 namespace(.)에서 접근 가능한 내장 객체를 설명한다.

- `Release`: release에 대한 정보를 제공한다.
- `Values`: template에 전달되는 values.yaml 파일. 기본적으로 Values는 빈 객체다.
- `Chart`: Chart.yaml 파일의 정보를 포함한다. 
- `Files`: chart와 관련 없는 파일에 대한 접근을 제공한다. template에 접근은 할 수 없다.
- `Capabilities`: k8s 클러스터가 지원하는 기능에 대한 정보를 제공한다.
- `Template`: 현재 실행되는 template의 정보를 포함한다.

내장 변수는 대문자로 시작한다. 이는 Go 언어의 naming 관습이다. 그렇기 떄문에 사용자 생성 변수의 경우 구분을 위해 소문자로 시작하도록 naming한다.