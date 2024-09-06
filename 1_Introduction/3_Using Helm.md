k8s 클러스터에서 패키지를 관리하기 위해 helm를 사용하는 기초에 대해 설명한다.

## Three Big Concepts
- **Chart**: chart는 helm package다. chart는 k8s cluster 내에서 실행될 resource에 대한 정의를 포함한다. Apt dkpg, Yum RPM 파일처럼 생각하면 된다.
- **Repository**: repository는 chart가 공유되는 공간이다.
- **Release**: release는 k8s cluster 내에서 실행 중인 chart 인스턴스다. 1개의 chart는 동일 cluster 내에서 여러번 설치될 수 있다. 각 설치마다 새로운 release가 생성된다.

즉, Helm은 각 설치 요청에 대해 새로운 Release를 생성함으로써 k8s에 Chart를 설치한다. 원하는 Chart는 Repository에서 검색할 수 있다.

## '`helm search`': Finding Charts
- `helm search hub`: Artifact Hub 내에서 여러 repo를 대상으로 원하는 helm chart를 검색한다.
- `helm search repo`: 로컬 helm 클라이언트에서 직접 추가(`helm repo add`)한 repo에서 검색한다. 이 검색은 로컬 데이터를 기준으로 하기 때문에 인터넷 연결이 필요하지 않다.

`helm search hub` 명령어 사용 시 chart 이름을 명시하지 않으면 사용 가능한 모든 chart를 검색한다.

## `'helm install'`: Installing a Package
`helm install` 명령어 사용 시 --generate-name 플래그를 사용하면 자동으로 release 이름을 부여한다.

install 동안 helm 클라이언트는 생성된 resource, release 상태, 추가 설정에 대한 유효한 정보를 출력한다.

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

helm은 종료되기 전에 모든 resource가 실행되기까지 기다리지 않는다. release의 상태를 확인하기 위해 `helm status` 명령어를 사용하면 된다.

### Customizing the Chart Before Installing
`helm show values` 명령어를 사용해 chart에 설정 가능한 변수 목록을 조회할 수 있다.

`helm install` 명령어 사용 시 아래 두 flag를 사용해 변수를 설정할 수 있다.

- --values(또는 -f): yaml 파일을 명시한다. 여러번 flag를 사용할 수 있으며 가장 오른쪽 파일이 우선 순위가 높다.
- --set: 커맨드라인을 통해 변수를 명시한다.

--set flag가 --values보다 우선순위가 높다. `helm get values <release-name>` 명령어를 사용해 해당 reelase에 --set flag로 설정한 변수를 조회할 수 있다. `helm upgrade` 명령어에 --reset-values flag를 사용해 --set flag로 설정한 변수를 초기화할 수 있다.

### The Format and Limitations of --set
- 기본적으로 --set `<key>=<value>` 형식을 사용해 변수를 설정한다.
- , 구분자를 사용해 --set flag 내 여러 변수를 설정할 수 있다.
- yaml map은 `.`을 사용해 접근하고 sequence는 `[index]`를 사용해 접근한다.
- yaml sequnce는 {value1, value, ...} 형식을 사용해 값을 설정할 수 있다.
- `<key>=null`, yaml sequnce는 `[]`,을 사용해 null 값과 빈값을 설정할 수 있다.
- --set flag를 사용할 때 특수 문자의 경우 \를 사용해 escape 처리해야한다.
- 중첩이 많은 데이터 구조의 경우 --set flag 사용에 어려움이 있을 수 있다. 그렇기 때문에 values.yml 파일 형식을 디자인할 때 --set flag 사용에 대해 고려하는 것이 좋다.

### More Installation Methods
`helm install` 명령어는 여러 소스로부터 chart를 설치할 수 있다:
- chart repo
- 로컬 chart 아카이브(`helm install foo foo-0.0.1.tgz`)
- unpacked chart 디렉토리(`helm install foo path/to/foo`)
- URL (`helm install foo https://example.com/charts/foo-1.2.3.tgz`)

## `'helm upgrade'` and `'helm rollback'`: Upgrading a Release, and Recovering on Failure
`helm rollback [RELEASE] [REVISION]` 명령어를 사용해 이전 release로 롤백할 수 있다.

설치, 업그레이드, 롤백 시 revision 숫자는 1씩 증가한다. 첫 번째 revision 숫자는 항상 1이다. `helm history [RELEASE]` 명령어를 사용해 특정 revision을 조회할 수 있다.

## Helpful Options for Install/Upgrade/Rollback
설치, 업그레이드, 롤백에 대한 helm의 동작을 커스터마이징하기 위해 아래 옵션을 사용할 수 있다.
- --timeout: k8s 명령어가 완료될 때까지 기다리는 시간. 기본 값은 5m0s(Go duration 형식)
- --wait: release가 성공으로 표시하기 위해 모든 po가 ready state, pvc가 바운딩, deploy가 최소 po 개수를 만족(desired - maxUnavailable), svc가 IP를 보유(그리고 loadbalancer일 경우 ingress도 보유)할 때까지 기다린다. 이는 --timeout 값까지 기다린다. timeout에 도달하면 release는 FAILED로 표시된다.
    > **Note**: deploy의 rolling update 전략에 대해 replicas가 1, maxUnavailable이 0으로 설정되지 않을 경우, --wait은 ready 상태의 최소 po를 충족하면 바로 반환된다.
- --no-hooks: 명령어에 대한 hook 실행을 건너뛴다.
- --recreate-pods(upgrade, rollback 명령어에만 유효): This flag willㅌ cause all pods to be recreated (with the exception of pods belonging to deployments). DEPRECATED in Helm 3

## 'helm uninstall': Uninstalling a Release

## 'helm repo': Working with Repositories
chart repo는 자주 변경될 수 있기 때문에 `helm repo update` 명령어를 사용해 로컬 데이터를 업데이트해야 한다.

## Creating Your Own Charts
패키지된 chart는 repo에 배포할 수 있다.

## Conclusion
