Helm 3는 패키지 배포를 위해 OCI를 지원한다. chart 패키지는 OCI-based registry에 저장, 공유 가능하다.

## Enabling OCI Support

## Running a registry

## Commands for working with registries
### The `registry` subcommand
`helm registry` 명령어를 사용해 로그인(`helm registry login`), 로그아웃(`helm registry logout`) 가능하다.

### The `push` subcommand
`helm push` 명령어는 `helm package` 명령어를 통해 생성된 .tgz 파일에 대해서만 사용가능하다.

OCI registry에 업로드 시, `oci://` 접두사를 사용해야하며 이름과 tag를 포함하면 안된다.

생성한 provenance file(`.prov`) 파일이 .tgz 파일과 같이 있을 떄, `helm push` 명령어에 의해 자동으로 업로드 된다. 명령어의 결과로 helm chart manifest에 layer가 생성된다.

### Other subcommands
아래 명령어 목록은 oci:// 프로토콜에 대해 사용 가능하다.

- helm push
- helm pull
- helm show
- helm template
- helm install
- helm upgrade

`helm push` 명령어를 제외한 명령어들은 이름을 명시해야 한다.

## Specifying dependencies

## Helm chart manifest