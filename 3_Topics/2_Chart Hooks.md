helm은 chart release 라이프사이클에서 특정 지점에서 개입할 수 있도록 hook 메커니즘을 제공한다. hook을 다음을 위해 사용할 수 있다:

- 다른 chart가 로드되기 전에 install 중 cm 또는 secret을 로드한다.
- 새로운 chart를 설치하기 전에 db를 백업하기 위한 job을 실행한다. 그리고 데이터를 복원하기 위해 업그레이드 이후 두 번째 job을 실행한다.
- Run a Job before deleting a release to gracefully take a service out of rotation before removing it.

hook은 일반 template과 동일하게 동작하지만 helm이 이를 일반 template이 아닌 다르게 활용할 수 있도록 특별한 k8s annotation을 포함한다.

## The Available Hooks

## Hooks and the Release Lifecycle
기본적으로 chart install에 대한 라이프사이클은 다음과 같다.

1. helm install foo 명령어를 사용해 chart를 release한다.
2. helm library의 instsall API가 호출된다.
3. 몇 가지 유효성을 확인하고 library는 foo template을 렌더링한다.
4. library는 결과 resource를 k8s에 로드한다.
5. library는 클라이언트에게 release 객체(및 기타 데이터)를 반환한다.
6. 클라이언트가 종료된다.

## Writing a Hook

