# Helm
## Helm 학습

### 요약
- cm, secrets를 업데이트할 때 해당 리소스를 사용하는 po에 대한 재배포가 필요할 수 있다. 이를 위해 helm chart 내에서 편법을 사용해 po가 재배포될 수 있도록 할 수 있다.
- templates/ 디렉토리에 underscore(_)로 시작하는 파일은 k8s manifest 파일로 간주되지 않는다. 관습에 따라 helper template, partials은 _heplers.tpl 파일에 작성된다.
