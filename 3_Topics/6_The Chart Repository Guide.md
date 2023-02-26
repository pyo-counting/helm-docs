Helm chart repository를 생성하고 사용하는 방법을 설명한다. chart repository는 패키지된 chart를 저장하고 공유할 수 있는 공간이다.

distributed community helm repository는 Artifact Hub에 존재한다. 물론 helm 명령어를 통해 개인 chart repository를 생성할 수 있다.

## Prerequisites

## Create a chart repository
chart repository는 index.yaml 파일과 (옵션)일부 패키지 chart를 저장하는 HTTP 서버다. chart를 공유할 준비가 됐을 때 chart repository에 업로드하는 것이 바람직하다.

Helm 2.2.0부터 repository에 대한 client-side SSL 인증이 지원된다. 다른 인증 프로토콜은 플러그인으로 사용할 수 있다.

chart repository는 YAML 파일, tar 파일을 제공할 수 있으며 GET method 요청에 대한 응답할 수 있는 HTTP 서버의 조건만 만족하면 되기 때문에 chart repository를 호스팅하는 여러 방법이 존재한다. 예를 들어 GCS bucket, AWS S3 bucket, GitHub 페이지를 사용하거나 자체 웹 서버를 사용할 수도 있다.

### The chart repository structure

### The index file

## Hosting Chart Repositories
chart repository를 제공하는 몇 가지 방법을 살펴본다.

### Google Cloud Storage

### Cloudsmith

### JFrog Artifactory

### GitHub Pages example

### Ordinary web servers

### ChartMuseum Repository Server

### GitLab Package Registry
GitLab의 프로젝트 별 Package Registry에 Helm chart를 게시할 수 있다. 자세한 내용은 [GitLab 관련 페이지](https://docs.gitlab.com/ee/user/packages/helm_repository/)를 참고한다.

## Managing Chart Repositories