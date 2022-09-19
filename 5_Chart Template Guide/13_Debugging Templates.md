렌더링된 template은 k8s API server로 전송되며 YAML 파일 포맷이 아닌 이유로 오류가 발생할 수 있으므로 template 디버깅은 까다로울 수 있다.

디버그를 도와줄 수 있는 몇 가지 명령어가 있다.

- helm lint: best practice를 따르는지 확인하는 데 사용되는 도구
- helm template --debug: template을 로컬에서 테스트 렌더링
- helm install --dry-run --debug: 서버에서 template을 렌더링한 다음 결과 manifest 파일을 반환
- helm install --disable-openapi-validation:
- helm get manifest: 서버에 설치된 template을 조회

YAML 파싱이 실패했지만 생성된 내용을 확인하기 위한 방법은 문제가 발생하는 구문을 주석처리한 다음 helm install --dry-run --debug으로 다시 실행해보는 것이다:

``` yaml
apiVersion: v2
# some: problem section
# {{ .Values.foo | quote }}
```

주석에 대한 렌더링도 진행된 결과를 확인할 수 있다:

``` yaml
apiVersion: v2
# some: problem section
#  "bar"
```

위와 같이 파싱 에러 없이 오류 구문에 대한 결과를 확인할 수 있다.