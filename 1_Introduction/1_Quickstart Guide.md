## Prerequisites

## Install Helm

## Initialize a Helm Chart Repository
Artifact Hub에서 helm chart를 검색 및 조회할 수 있다. 그리고 해당 helm chart를 설치할 수 있는 repository 주소를 제공한다. 해당 repository를 `helm repo add` 명령어로 추가 할 수 있다. 각 repo는 해당 helm chart뿐만 아니라 다른 chart를 포함할 수도 있다.

## Install an Example Chart

## Learn About Releases

## Uninstall a Release
`helm uninstall` 명령어 사용 시 --keep-history flag를 사용하면 release 히스토리가 보존된다(release의 상태는 uninstalled가 됨). 삭제를 위해 --keep-history flag 없이 `helm unisntall` 명령어를 사용하면 된다. release에 대한 정보는 `helm status` 명령어를 통해 조회 가능하다.

helm은 release를 uninstall한 후에도 추적하기 때문에 history를 조회 및 rollback(`helm rollback`)할 수 있다.