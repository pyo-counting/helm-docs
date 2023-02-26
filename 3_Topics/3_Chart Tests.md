## Example Test
chart는 여러 k8s resource에 대한 여러 object를 포함한다. chart 작성자는 install 시 실제로 chart가 예상한 것처럼 정상 동작하는지 확인하기 위해 test를 작성하길 원할 수도 있다. 이를 통해 chart 이용자 역시 chart의 동작 방식을 이해할 수 있다.

test는 templates/ 디렉터리에 k8s Job resource 타입이다. container는 test가 성공적으로 수행되었음을 나타내기 위해 exit code 0을 반환해야 한다. test를 위한 job 객체는 helm test hook인 `helm.sh/hook: test` annotation을 포함해야 한다.
## Steps to Run a Test Suite on a Release

## Notes
- tempaltes/ 디렉터리 내에 원하는 만큼의 테스트가 정의된 yaml 파일을 생성할 수 있다.
- test와 template을 구분하기 위해 \<chart-name\>/templates/tests 디렉터리에 test 관련 파일을 정의할 수도 있다.
- test는 Helm hook이기 때문에 `helm.sh/hook-weight`, `helm.sh/hook-delete-policy`와 같은 annotation이 resource에 정의되어야 한다.