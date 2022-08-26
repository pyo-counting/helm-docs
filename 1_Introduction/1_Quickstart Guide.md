## Uninstall a Release
`helm uninstall` 명령어 사용 시 --keep-history flag를 사용하면 release 히스토리가 보존된다(release의 상태는 uninstalled가 됨). 삭제를 위해 --keep-history flag 없이 `helm unisntall` 명령어를 사용하면 된다. release에 대한 정보는 `helm status` 명령어를 통해 조회 가능하다.

helm은 release를 uninstall한 후에도 추적하기 때문에 history를 조회 및 rollback(`helm rollback`)할 수 있다.