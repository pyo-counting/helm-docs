chart manifest에서 RBAC resource를 사용하는 best practice를 설명한디.

RBAC resource는 다음과 같다:

- sa (namespaced)
- Role (namespaced)
- ClusterRole (non-namespaced)
- RoleBinding (namespaced)
- ClusterRoleBinding (non-namespaced)

## YAML Configuration
RBAC, sa 설정은 별도의 키로 관리해야 한다. 이들은 별개다. YAML에서 이 두 개념을 분리하면 명확해진다.

``` yaml
rbac:
  # Specifies whether RBAC resources should be created
  create: true

serviceAccount:
  # Specifies whether a ServiceAccount should be created
  create: true
  # The name of the ServiceAccount to use.
  # If not set and create is true, a name is generated using the fullname template
  name:
```

위 구조는 여러 sa가 필요한 더 복잡한 chart로 확장될 수 있다.

``` yaml
someComponent:
  serviceAccount:
    create: true
    name:
anotherComponent:
  serviceAccount:
    create: true
    name:
```

## RBAC Resources Should be Created by Default
rbac.create는 RBAC resource 생성 여부를 나타내기 위해 bollean 값이어야 한다. 기본 값은 true다. RBAC 접근을 사용자가 직접 제어하길 원할 경우 false로 설정하면 된다.

## Using RBAC Resources
serviceAccount.name는 chart에서 생성한 접근 제어가 필요한 resource에서 사용하기 위한 sa의 이름이 설정돼야 한다. serviceAccount.create가 true라면 위 이름을 이용해 sa가 생성된다. 이름이 설정되지 않으면 fullname template을 사용한다. serviceAccount.create가 false라면 생성은 되지 않지만 나중에 생성될 RBAC resource가 올바르게 동작할 수 있도록 동일한 resource에 연결되어야 한다. serviceAccount.create가 false이고 이름을 설정하지 않으면 기본 sa가 사용된다.

아래 helper template은 sa 생성을 위해 사용할 수 있다:

``` yaml
{{/*
Create the name of the service account to use
*/}}
{{- define "mychart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create -}}
    {{ default (include "mychart.fullname" .) .Values.serviceAccount.name }}
{{- else -}}
    {{ default "default" .Values.serviceAccount.name }}
{{- end -}}
{{- end -}}
```