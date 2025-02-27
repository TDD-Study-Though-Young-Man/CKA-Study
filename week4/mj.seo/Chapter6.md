# 4️⃣ Week 4 (Section 6)

- `Drain Node`
    - 클러스터의 특정 노드에서 모든 포드를 안전하게 제거하고 노드를 비우는 과정을 말한다.
    - 노드를 유지보수하거나 업그레이드할 때 사용된다.
    
    ### kubectl drain 동작 과정
    
    1. `파드 제거 준비` : 노드에 있는 모든 파드를 다른 노드로 이동시키기 위해 노드가 **스케줄링 대상**에서 제외된다.
    2. `파드 종료` : 파드가 안전하게 종료되고 다른 노드에서 재시작될 수 있도록 `Graceful termination`이 적용된다.
    3. `파드 재배치` : 제거된 파드가 다른 노드에서 **스케줄링**되며, **상태**를 **유지**하는 **파드**의 경우 `PersistenceVolumeClaims`와 같은 리소스를 **다른 노드에서 사용**할 수 있도록 한다.
    4. `노드 비우기 완료` : 모든 파드가 **제거**되면 노드가 완전히 비워진다.
    
    ### 사용 예시
    
    ```bash
    kubectl drain <노드 이름> --ignore-daemonsets --delete-local-data
    ```
    
    - `-ignore-daemonsets`: **DaemonSet**으로 관리되는 파드는 비우지 않는다. **DaemonSet**은 **노드**에서 지속적으로 실행되어야 하는 **파드**이기 때문에 무시된다.
    - `-delete-local-data`: 로컬 데이터를 가진 **파드**를 **강제**로 **삭제**합니다. 이는 일반적으로 **로컬 저장소**를 사용하는 경우에 **필요**합니다.

- `Uncordon Node`
    - 이전에 `cordon` 상태로 설정된 노드를 다시 **스케줄링** 가능한 상태로 되돌리는 작업이다.
    - `kubectl uncordon` 명령어를 사용하여 노드를 `uncordon` 할 수 있다.
    
    ### 사용 예시
    
    ```bash
    kubectl uncordon <노드 이름>
    ```
    
    - 이 `명령어`를 실행하면 지정한 `노드`가 다시 `스케줄링 대상`이 되어 클러스터의 `스케줄러`가 새로운 **파드**를 이 **노드**에 **할당**할 수 있게 된다.

### 사용할 수 있는 상황

- **`업그레이드 후 복구`** : **노드**의 **소프트웨어**나 **하드웨어**를 업그레이드한 후, `노드`를 다시 `클러스터`의 일부로 만들고 싶을 때 사용합니다.
- **`문제 해결 후 복구`** : **노드**에서 발생한 **문제**를 해결한 후, `노드`를 `정상 상태`로 되돌려 다시 `사용`하고자 할 때 사용합니다.

- `Cordon Node`
    - `특정 노드`에서 새로운 파드가 `스케줄링`되지 않도록 설정하는 작업을 의미한다.
    - `노드`를 **유지 보수**하거나 **점검**하기 위해 **사용**된다.

### 사용 예시

```bash
kubectl cordon <노드이름>
```

- 이 `명령어`를 실행하면 지정한 `노드`는 `새로운 파드`를 스케줄링하지 않게 된다.
- 이미 해당 `노드`에서 실행 중인 `파드`에는 영향을 미치지 않으며, 새로운 `파드`만 스케줄링되지 않도록 한다.

### 사용할 수 있는 상황

- **`유지 보수 전 준비`** : `노드`를 **업그레이드**하거나 **하드웨어**를 `점검`하기 전, 새로운 `파드`가 `노드`에 스케줄링되는 것을 `방지`하기 위해 `사용`
- **`문제 해결 전 준비`** : `노드`에서 문제가 발생했을 때, `문제`를 해결하는 동안 **새로운 파드**가 `노드`에 스케줄링되지 않도록 하기 위해 `사용`

### Cordon vs Drain

- **`cordon`** : 노드를 `SchedulingDisabled` 상태로 설정하여 새로운 파드가 `스케줄링`되지 않도록 한다. 기존 파드는 계속해서 실행
- **`drain`** : 노드를 비우기 위해 기존의 모든 파드를 **다른 노드**로 `이동`시킵니다. 이 과정에서 `파드`는 안전하게 종료되고 `다른 노드`에서 **재시작**

- `Kubernetes Version`
    - **Major**
        - 주요 `변경 사항`을 나타내며, `이전 버전`과 호환되지 않는 `큰 변화`가 있을 때 증가
        - 예를 들어, `v1.0.0`에서 `v2.0.0`으로의 변경은 `Major` 버전 업그레이드를 의미
    - **Minor**
        - `새로운 기능`이 추가되거나 향상될 때 `증가`하지만, `이전 버전`과의 **호환성은 유지**
        - 예를 들어, `v1.1.0`은 `v1.0.0`과 호환되지만 **새로운 기능이 포함**
    - **Patch**
        - `버그 수정`이나 `보안 패치`와 같은 작은 변경 사항을 나타내며, 기능 추가 없이 `오류`를 `수정`
        - 예를 들어, `v1.0.1`은 `v1.0.0`에서 발견된 버그를 수정한 버전이다.
    - **Tag Alpha**
        - 초기 `테스트 단계`의 버전을 나타내며, 기능이 `완전`하지 않거나 `안정적`이지 않을 수 있다.
        - 예를 들어, `v1.1.0-alpha.1`은 `v1.1.0`의 **첫 번째 알파 버전이다.**
    - **Tag Beta**
        - `베타 테스트 단계`의 버전을 나타내며, 대부분의 `기능`이 `포함`
        - `상대적`으로 안정적이지만 여전히 **테스트가 필요**합니다.
        - 예를 들어, `v1.1.0-beta.1`은 `v1.1.0`의 **첫 번째 베타 버전이다.**
- `Cluster Upgrade` - https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
    - `클러스터`의 **구성 요소(마스터 노드 & 워커 노드)**를 `최신 버전`으로 `업데이트`하여 새로운 기능, 보안 패치, 버그 수정 등을 `적용`하는 `과정`이다.

### Upgrade 과정

- `준비 단계`
    - **버전 확인** : 현재 사용 중인 `Kubernetes` 버전을 확인하고 `업그레이드`하려는 대상 버전의 변경 사항 및 `호환성`을 `검토`한다.
    - **백업** : `클러스터` 구성 및 데이터(예: etcd 데이터베이스)를 `백업`한다. `업그레이드` 과정에서 문제가 발생할 경우를 대비해 `복구 가능한 상태`를 **유지한다.**
    - **업그레이드 경로 검토** : `Kubernetes`는 특정 업그레이드 경로를 따른다. `Major` 버전은 한 번에 하나씩, `Minor` 및 `Patch` 버전은 여러 단계를 건너뛰지 않도록 한다.

- `마스터 노드 업그레이드`
    - **업그레이드 계획** : `마스터 노드`의 업그레이드 순서를 계획한다. `고가용성 클러스터`에서는 마스터 노드를 순차적으로 업그레이드한다.
    - **API 서버 업그레이드** : `kube-apiserver`를 업그레이드한다. 이는 `클러스터`의 주요 구성 요소로, 다른 `마스터 노드` 구성 요소들이 이를 따른다.
    - **컨트롤러 매니저 및 스케줄러 업그레이드** : `kube-controller-manager`와 `kube-scheduler`를 업그레이드한다.
    - **etcd 업그레이드** : 클러스터 데이터 저장소인 `etcd`를 업그레이드합니다. 이 단계는 매우 `신중`하게 `수행`되어야 합니다.
    - **클러스터 상태 확인** : 마스터 노드 업그레이드 후 `클러스터 상태`를 확인하고 모든 구성 요소가 정상 작동하는지 확인한다.
    
- `워커 노드 업그레이드`
    - **노드 cordon** : `업그레이드`하려는 워커 노드를 `cordon` 상태로 설정하여 새로운 파드가 스케줄링되지 않도록 한다.
        
        ```bash
        kubectl cordon <노드이름>
        ```
        
    - **Pod 드레인**: 해당 `노드`의 `파드`를 다른 `노드`로 `이동`시킨다.
        
        ```bash
        kubectl drain <노드이름> --ignore-daemonsets --delete-local-data
        ```
        
    - **노드 업그레이드** : `kubelet` 및 `kube-proxy`를 업그레이드한다. `운영 체제` 및 `기타 소프트웨어`를 **업데이트**할 수도 있습니다.
    - **노드 uncordon** : `업그레이드`가 완료된 후 노드를 `uncordon`하여 다시 `파드`를 `스케줄링`할 수 있도록 한다.
        
        ```bash
        kubectl uncordon <노드이름>
        ```
        

- `클러스터 기능 확인`
    - **테스트 및 검증** : `클러스터`와 `애플리케이션`이 정상적으로 `작동`하는지 확인한다. 필수적인 `기능 테스트`를 `수행`한다.
    - **모니터링** : `업그레이드` 후 클러스터 성능 및 안정성을 `모니터링`합니다.

- `kubeadm`
    - **설명** : `kubeadm`은 `Kubernetes` 클러스터를 설치하고 관리하기 위한 도구이다. `kubeadm`은 이러한 `클러스터`를 설정하는 과정을 단순화하는데 사용된다.
    - **장점**
        - **사용 용이성**: 복잡한 설정을 단순화하여 `초보자`도 쉽게 클러스터를 `구축`할 수 있다.
        - **보안**: 기본적인 보안 기능을 제공하여 안전한 `클러스터` 운영이 가능하다.
        - **유연성**: 다양한 환경에서 `Kubernetes` 클러스터를 구축할 수 있다.

### ETCD 백업

- `Snapshot 생성`
    
    ```bash
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
    --ca-cert=<trusted-ca-file> \
    --cert=<cert-file> \
    --key=<key-file> \
    snapshot save <backup-file-location>
    ```
    

- `Snapshot File을 활용해 Restore`
    
    ```bash
    ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> \
    snapshot restore <snapshot_file>
    ```
