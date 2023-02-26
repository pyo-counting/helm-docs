## The Purpose of Helm
helm은 chart라고 불리는 k8s package를 관리하는 도구이다. helm을 통해:

- 새로운 chart를 생성할 수 있다.
- chart를 *.tgz 아카이브 파일로 패키지할 수 있다.
- chart가 저장되는 repo와 상호작용할 수 있다.
- k8s cluster에 chart를 설치, 삭제할 수 있다.
- helm을 통해 설치된 chart의 릴리즈 사이클을 관리할 수 있다.

helm은 중요한 3가지 개념을 갖는다:

1. chart는 k8s 애플리케이션 인스턴스를 생성하기 위해 필요한 정보다.
2. config는 릴리즈 가능한 객체를 만들기 위한 chart에 병합할 수 있는 설정 정보가 포함된다.
3. release는 특정 config와 결합된 동작 중인 chart 인스턴스다.

## Components
helm은 두 부분으로 구현된 실행파일이다:

helm client는 end user를 위한 command-line client다. client의 역할은 다음과 같다:

- 로컬 chart 개발
- repo 관리
- release 관리
- helm library와 상호작용
   - 설치될 chart 전송
   - release에 대한 업그레이드, 삭제 요청

helm library은 모든 helm 작업을 실행하기 위한 로직를 제공한다. k8s API 서버와 통신하며 아래 기능을 제공한다:

- chart와 config을 결합해 release 빌드
- k8s에 chart를 설치
- k8s와 상호작용함으로써 chart를 업그레이드, 삭제

## Implementation
helm client, library는 Go 언어로 작성된다.

library는 k8s client library를 사용해 k8s와 통신한다. Currently, that library uses REST+JSON. It stores information in Secrets located inside of Kubernetes. It does not need its own database.