helm install, helm upgrade의 경우 사용자를 위한 정보를 출력할 수 있다. 이러한 정보는 template을 사용해 작성할 수 있다.

이러한 정보는 templates/NOTES.txt 파일에 작성하면 된다. 이 파일은 일반 텍스트 파일이지만 template 파일처럼 처리되기 때문에 일반 template 함수 및 객체를 사용할 수 있다.

아래는 예시다:

``` yaml
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}
```

NOTES.txt 파일을 사용해 새로 설치된 chart를 사용하는 방법에 대한 자세한 정보를 사용자에게 제공할 수 있다. 필수는 아니지만 해당 파일을 제공하는 것이 좋다.