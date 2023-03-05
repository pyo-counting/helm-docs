때때로 template이 아닌 파일을 가져와 template 렌더링을 거치지 않고 내용을 직접 삽입하기 위한 경우도 있다.

이를 위해 helm은 .Files 객체를 제공한다. 관련해 주의 사항은 다음과 같다:

- helm chart에 기본 구성 파일 외 추가 파일을 추가하는 것은 상관 없다 하지만  k8s 오브젝트의 저장 제한이 있기 때문에 chart는 1M보다 작아야 한다.
- 보안상 이유로 일부 파일의 경우 .Files 객체를 통해 접근할 수 없다.
    - templates/ 디렉토리 내 파일은 접근할 수 없다.
    - .helmignore 파일을 통해 제외된 파일은 접근할 수 없다.
    - Files outside of a helm application subchart, including those of the parent, cannot be accessed
- chart는 UNIX의 파일 소유, 권한을 유지하지 않기 때문에 .Files 오브젝트를 통해 접근하는 경우 상관이 없다.

## Basic example

## Path helpers

## Glob patterns

## ConfigMap and Secrets utility functions

## Encoding

## Lines