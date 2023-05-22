`Values` 내장 객체를 사용해 chart에 전달된 변수에 접근 가능하다. chart에 전달하는 곳은 여러가지다:

- chart 내 values.yaml 파일
- subchart일 경우 부모 chart의 values.yaml
- helm install, helm upgrade 명령어의 -f flag로 전달된 파일
- helm install, helm upgrade 명령어의 --set flag로 전달된 변수

위는 우선순위를 나타내기도 한다. 현재 chart의 values.yaml 파일 내 변수가 기본 값이며, 부모 chart의 values.yaml 파일에 의해 변수들이 재정의될 수 있으며, 사용자가 chart 생성 또는 업데이트 시 전달하는 파일 또는 파라미터에 의해 다시 재정의 될 수도 있다.

While structuring data this way is possible, the recommendation is that you keep your values trees shallow, favoring flatness. When we look at assigning values to subcharts, we'll see how values are named using a tree structure.

## Deleting a default key
변수의 기본 값에서 key를 삭제해야 할 경우 `null`로 재정의할 수 있다. 이 경우 helm은 key를 삭제한다.

아래는 예시다:

``` yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  initialDelaySeconds: 120
```

livenessProbe의 httpGet 대신 exec를 사용하고자 할 경우 --set livenessProbe.httpGet=null flag를 이용해 httpGet을 삭제할 수 있다.

``` bash
helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null
```

아래는 결과다.

``` yaml
livenessProbe:
  exec:
    command:
    - cat
    - docroot/CHANGELOG.txt
  initialDelaySeconds: 120
```
