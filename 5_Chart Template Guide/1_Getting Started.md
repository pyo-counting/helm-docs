## Charts
`templates/` 디렉터리는 template 파일이 저장되는 곳이다. helm이 chart를 평가할 때 `templates/` 디렉토리 내 모든 파일을 template rendering engine에 전달한다. 그리고 template에 대한 결과를 수집해 k8s에 전달한다.

**NOTE**: 'templates/' 디렉터리 내 디렉터리를 추가해 여러 template를 구분해 관리하는 것도 가능하다. 이는 doc에서는 확인이 불가하나, artifact hub 내 몇몇 chart 확인 시, 해당 방법을 통해 관리하는 것으로 보인다.

## A Starter Chart
해당 가이드에서는 `helm create mychart` 명령어를 사용해 생성된 mychart helm chart를 사용한다.
### A Quick Glimpse of mychart/templates/

## A First Template

**Tip**: template 이름은 엄격한 네이밍 규칙이 있는 것은 아니다. 하지만 .yaml 접미사를 사용해 YAML 파일을, .tpl 접미사를 사용해 helper를 나타내는 것을 권장한다.

### Adding a Simple Template Call

**Tip**: k8s resource 이름은 DNS 시스템 제한으로 인해 63개 문자로 제한된다.

> template 지시자는 `{{`, `}}` 내에 표현한다.

template 지시자 `{{ .Release.Name }}`은 template에 릴리즈 이름을 주입한다. 이 변수는 namespaced 객체로 생각할 수 있으며, 여기서 .은 각 namespace 요소를 구분한다.

제일 앞의 .은 최상위 namespace를 가리킨다. 따라서 "최상위 namespace로 시작해 Release 객체를 찾은 다음 그 내부에서 Name 객체를 찾는다"로 해석할 수 있다.

--dry-run flag는 resource YAML파일을 테스트할 수 있으며 이는 k8s에 배포되지 않는다.