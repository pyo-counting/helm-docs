## Chart Names
chart 이름은 소문자, 숫자여야 한다. 이름 내 단어는 dash(-)로 구분된다:

예시:

```
drupal
nginx-lego
aws-cluster-autoscaler
```

대문자, underscore(_)도 chart 이름에 사용할 수 있지만 권장 사항은 아니다.

## Version Numbers
helm은 가능한 경우 SemVer 2를 사용해 버전 번호를 사용한다(docker image tag는 반드시 SemVer를 따를 필요는 없으므로 규칙에 대한 예외로 간주된다).

SemVer 버전이 k8s label에 저장될 때 label은 + 기호를 값으로 허용하지 않기 때문에 일반적으로 + 문자를 _ 문자로 변경한다.

## Formatting YAML
YAML 파일은 2개 공백 문자를 사용해 들여쓰기해야 한다.

## Usage of the Words Helm and Chart
Helm, helm 단어를 사용하는 데 몇 가지 규칙이 존재한다.

- Helm은 프로젝트 전체를 나타낸다.
- helm은 클라이언트 명령어를 나타낸다.
- chart라는 용어는 고유 명사가 아니므로 대문자로 사용할 필요가 없다.
- Chart.yaml은 파일 이름이 대소문자를 구분하므로 대문자로 표시해야 한다.

확실하지 않은 경우 Helm을 사용한다.
