## Three Big Concepts
- **Chart**: chart는 helm package다. chart는 k8s cluster 내에서 실행될 resource에 대한 정의를 포함한다. Apt dkpg, Yum RPM 파일처럼 생각하면 된다.
- **Repository**: repository는 chart가 공유되는 공간이다.
- **Release**: release는 k8s cluster 내에서 실행 중인 chart 인스턴스다. 1개의 chart는 동일 cluster 내에서 여러번 설치될 수 있다. 각 설치마다 새로운 release가 생성된다.

즉, Helm은 각 설치 요청에 대해 새로운 Release를 생성함으로써 k8s에 Chart를 설치한다. 원하는 Chart는 Repository에서 검색할 수 있다.

## '`helm search`': Finding Charts
- `helm search hub`: Artifact Hub 내에서 원하는 helm chart를 검색한다.
- `helm search repo`: 로컬 helm 환경에서 직접 추가(`helm repo add`)한 repo에서 검색한다. 이 검색은 로컬 데이터를 기준으로 하기 때문에 인터넷 연결이 필요하지 않다.

## `'helm install'`: Installing a Package
helm은 아래 순서대로 resource를 설치한다:

- Namespace
- NetworkPolicy
- ResourceQuota
- LimitRange
- PodSecurityPolicy
- PodDisruptionBudget
- ServiceAccount
- Secret
- SecretList
- ConfigMap
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CustomResourceDefinition
- ClusterRole
- ClusterRoleList
- ClusterRoleBinding
- ClusterRoleBindingList
- Role
- RoleList
- RoleBinding
- RoleBindingList
- Service
- DaemonSet
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- HorizontalPodAutoscaler
- StatefulSet
- Job
- CronJob
- Ingress
- APIService

helm은 종료되기 전에 모든 resource가 실행되기까지 기다리지 않는다.

### Customizing the Chart Before Installing
`helm show values` 명령어를 사용해 chart에 설정 가능한 변수 목록을 조회할 수 있다.

--set flag가 --values보다 우선순위가 높다. `helm get values <release-name>` 명령어를 사용해 해당 reelase에 --set flag로 설정한 변수를 조회할 수 있다. `helm upgrade` 명령어에 --reset-values flag를 사용해 --set flag로 설정한 변수를 초기화할 수 있다.

### The Format and Limitations of --set
중첩이 많은 데이터 구조의 경우 --set flag 사용에 어려움이 있을 수 있다. 그렇기 때문에 values.yml 파일 형식을 디자인할 때 --set flag 사용에 대해 고려하는 것이 좋다.

### More Installation Methods
`helm install` 명령어는 여러 소스로부터 chart를 설치할 수 있다:
- chart repo
- 로컬 chart 아카이브(`helm install foo foo-0.0.1.tgz`)
- unpacked chart 디렉토리(`helm install foo path/to/foo`)
- URL (`helm install foo https://example.com/charts/foo-1.2.3.tgz`)

## `'helm upgrade'` and `'helm rollback'`: Upgrading a Release, and Recovering on Failure
설치, 업그레이드, 롤백 시 revision 숫자는 1씩 증가한다. 첫 번째 revision 숫자는 항상 1이다. `helm history [RELEASE]` 명령어를 사용해 특정 revision을 조회할 수 있다.