library chart는 다른 helm chart에서 사용할 수 있도록 공유 가능한 chart의 한 유형이다. library chart를 사용해 사용자는 여러 chart 간에 재사용할 수 있는 코드를 공유함으로써 DRY(Don't repeat yourself)를 수행할 수 있다.

library chart는 Helm 2 이후 chart 관리자들이 사용하는 공통 helper chart를 공식적으로 인식하기 위해 helm 3부터 도입됐다. library chart를 통해:

- library chartr와 application chart와 명시적으로 구분이 가능하다.
- library chart에 대한 설치를 방지하기 위한 로직을 제공한다.
- release artifact를 포함할 수 있는 library chart에서 template을 렌더링하지 않는다.
- Allow for dependent charts to use the importer's context

chart 관리자는 library chart를 정의할 수 있게 됐으며, helm이 일관된 방식으로 chart를 다룰 수 있다. 언제든지 chart 타팁을 가능함으로써 application chart의 정의를 공유할 수 있습니다.

## Create a Simple Library Chart

## Use the Simple Library Chart

## Library Chart Benefits

## The Common Helm Helper Chart
