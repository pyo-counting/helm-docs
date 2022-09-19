helm template language는 GO 프로그래밍 언어로 구현됐다. 이러한 이유로 template 내 변수에 타입이 있어야 한다. 대부분의 경우 변수는 다음 중 하나의 타입으로 노출된다:

- string: A string of text
- bool: a true or false
- int: An integer value (there are also 8, 16, 32, and 64 bit signed and unsigned variants of this)
- float64: a 64-bit floating point value (there are also 8, 16, and 32 bit varieties of this)
- a byte slice ([]byte), often used to hold (potentially) binary data
- struct: an object with properties and methods
- a slice (indexed list) of one of the previous types
- a string-keyed map (map[string]interface{}) where the value is one of the previous types

Go에는 이외 다른 여러 타입도 있다. 이러한 타입을 변환할 때 `printf "%t"`, `typeOf`, `KindOf` 함수를 이용해 먼저 타입을 확인할 수 있다.