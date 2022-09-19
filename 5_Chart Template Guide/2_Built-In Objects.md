객체는 template 엔진에서 template으로 전달된다. 그리고 코드 내에서 객첼르 전달할 수도 있다(`with`, `range` 구문을 이용해). `tuple` 함수를 이용해서 template 내에서 객체를 생성할 수도 있다.

객체는 단순히 1개의 변수를 갖거나 다른 객체 또는 함수를 포함할 수도 있다.

아래는 최상위 namespace에서 접근 가능한 내장 객체를 설명한다.

- `Release`:
- `Values`:
- `Chart`:
- `Files`:
- `Capabilities`:
- `Template`:

내장 변수는 대문자로 시작한다. 이는 Go 언어의 naming 관습이다. 그렇기 떄문에 사용자 생성 변수의 경우 구분을 위해 소문자로 시작하도록 naming한다.

