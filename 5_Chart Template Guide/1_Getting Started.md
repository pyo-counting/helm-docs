## Charts
`templates/` 디렉터리는 template 파일이 저장되는 곳이다. helm이 chart를 평가할 때 `template/` 디렉토리 내 모든 파일을 template rendering engine에 전달한다. 그리고 template에 대한 결과를 수집해 k8s에 전달한다.

**NOTE**: 'templates/' 디렉터리 내 추가 디렉터리를 추가해 여러 template를 구분해 관리하는 것도 가능하다. 이는 doc에서는 확인이 불가하나, artifact hub 내 몇몇 chart 확인 시, 해당 방법을 통해 관리하는 것으로 보인다.