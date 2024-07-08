# 2️⃣ Week 2 (Section 3)
- `Scheduler`
    - **설명**
        
        `Kubernetes Scheduler`는 Kubernetes 클러스터에서 새로 생성된 Pod를 실행하기 위해 적절한 **Node**를 선택하는 역할을 담당하는 `컴포넌트` 다.
        
        `Kubernetes`에서 Pod는 애플리케이션의 컨테이너와 그 실행 환경을 정의하는 단위이며, `Scheduler`는 이러한 Pod가 클러스터 내의 **어떤 Node에서 실행될지를 결정**하는 중요한 역할을 한다.
        
    
    - **주요 기능**
        - `Pod 대기열 관리` : Scheduler는 아직 스케줄링되지 않은 **새로 생성된 Pod들을 모니터링**하고, 이들을 대기열에 추가한다.
        - `적합한 Node 선택` : Pod를 실행할 적합한 Node를 선택한다.
        - `바인딩` : 가장 높은 점수를 받은 `Node`를 선택한 후, `Scheduler`는 해당 Pod를 그 **Node에 바인딩 한다.**

- `No Scheduler & NodeName`
    - **설명**
    
    `Kubernetes` 클러스터에서 `Scheduler`를 사용하지 않고 `NodeName` 필드를 이용해 직접 Pod를 특정 Node에 배포하는 방법을 사용할 수 있다.
    
     `NodeName`을 설정하여 Pod를 특정 Node에 직접 할당하는 것은 `Kubernetes Scheduler`를 우회하는 방식이다.
    
    - `NodeName`을 이용하여 **Pod 배포하기**
        - `Pod` 정의 파일에서 `nodeName` 필드를 설정한다.
            
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: my-pod
            spec:
              containers:
              - name: my-container
                image: my-image
              nodeName: my-node
            ```
            
        
    - `장점`
        - `직접 제어` : 특정 `Pod`를 특정 `Node`에 **반드시 배포해야 할 경우 유용**하다.
        - `간단함` : `Scheduler`의 복잡한 설정 없이 간단하게 특정 `Node`에 `Pod`를 배포할 수 있다.
        
    - `단점`
        - `수동관리` : `Scheduler`가 자동으로 최적의 `Node`를 선택하지 않으므로, 사용자가 수동으로 `Node`를 선택해야 합니다. 이는 **클러스터 관리의 복잡성을 증가시킨다.**
        - `리소스 효율성 저하` : `자동 스케줄링`에 의해 클러스터 전체의 **리소스 사용률을 최적화**하는 기능을 사용할 수 없기 때문에, `리소스 사용`의 `비효율성`이 **발생**할 수 있다.
        - `내결함성 부족` : 특정 `Node`에 `Pod`를 고정하면, 해당 `Node`에 문제가 발생했을 때 다른 `Node`로의 **자동 재배치가 이루어지지 않아** 가용성이 떨어집니다.

- `Binding Object`
    - **설명**
        
        `Kubernetes`에서 **Binding Object**는 특정 `Pod`를 특정 `Node`에 명시적으로 `바인딩`하는 데 사용되는 리소스이다.
        
        `Binding Object`는 **Scheduler**를 **우회**하여 **수동으로 스케줄링**을 구현할 때 유용하다. 이는 일반적으로 `클러스터 관리자`가 **특정 Pod가 특정 Node에서 실행되도록 강제**할 필요가 있을 때 사용된다.
        
    - **Binding Object**의 구성 요소
        
        `Binding Object`는 주로 다음과 같은 필드로 구성된다.
        
        - **apiVersion**: Kubernetes API 버전, 보통 "v1"을 사용
        - **kind**: 리소스 유형으로, `"Binding"`으로 설정
        - **metadata**: `Binding Object`의 `metadata`로, 주로 이름과 네임스페이스를 포함
        - **target**: `Pod`를 바인딩할 `Node`를 지정합니다. 이 필드는 `apiVersion`, `kind`, `name` 필드로 구성
        
        ```yaml
        apiVersion: v1
        kind: Binding
        metadata:
          name: my-pod
          namespace: default
        target:
          apiVersion: v1
          kind: Node
          name: my-node
        ```
        
    
    - `Binding Object` 사용 방법
        1. **Pod 생성**: 먼저 스케줄링되지 않은 상태로 `Pod`를 생성한다. 이는 `Pod` 정의에서 `nodeName` 필드를 생략하여 생성한다.
        2. **Binding Object 생성**: `Pod`가 생성된 후, `Binding Object` 를 만들어 해당 Pod를 특정 Node에 **바인딩한**다.
        3. **API 서버에 적용**: `Binding Object`를 API 서버에 적용하여 바인딩을 완료한다. 이는 `kubectl apply -f binding.yaml` 명령어를 사용하여 수행할 수 있습니다.

- `Label & Selector`
    - **설명**
        
        `Kubernetes`에서 **Label**과 **Selector**는 `클러스터 리소스`를 구성하고 관리하는 데 중요한 역할을 하는 `메커니즘`이다.
        
        `Label`은 `리소스`에 붙이는 **키-값 쌍**이며, `Selector`는 **특정 라벨을 기반**으로 `리소스`를 선택하는 데 사용된다.. 이를 통해 사용자는 `유연`하고 `강력한 방식`으로 **리소스를 조직하고 제어**할 수 있다.
        
    - `Label`
        - `Label`은 `Kubernetes` 객체(예: **Pod, Service, Node** 등)에 첨부할 수 있는 `키-값 쌍`이다. `Label`은 객체를 식별하고 그룹화하는 데 사용되며, **특정 목적이나 속성**을 나타낸다.
        
        - `라벨의 주요 특징`
            - **키-값 쌍**: 라벨은 `key: value` 형태를 가진다. 예를 들어, `environment: production`, `app: frontend` 등의 형태를 가질 수 있다.
            - **다양한 사용**: 여러 목적을 위해 동일 객체에 여러 라벨을 추가할 수 있다.
            - **선택적**: 라벨은 선택 사항이며, 객체에 붙이거나 붙이지 않을 수 있다.
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: my-pod
          labels:
            app: my-app
            environment: production
        spec:
          containers:
          - name: my-container
            image: my-image
        
        ```
        
    
    - `Selector`
        
        `Selector`는 `Label`을 기반으로 **객체를 선택하고 필터링하는 데 사용된다.**
        
        `Kubernetes`는 `Selector`를 사용하여 `특정 라벨`을 가진 객체를 검색하고 작업을 수행할 수 있다. 
        
        `Selector`에는 주로 두 가지 유형이 있다.  `동등(equal) 셀렉터`와 `집합-based 셀렉터` 이다.
        
    
    - `동등 (Equal) Selector`
        - **동등 Selector**는 특정 라벨 키와 값이 **정확히 일치하는 객체를 선택**한다.
        
        ```yaml
        selector:
          matchLabels:
            app: my-app
            environment: production
        ```
        
    
    - `집합-based Selector`
        - `집합-based Selector` 는 `라벨 키`가 **특정 집합 조건을 만족하는 객체를 선택한다.**
            
            이는 `matchExpressions`를 사용하여 정의된다.
            
        
        ```yaml
        selector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - my-app
            - another-app
          - key: environment
            operator: NotIn
            values:
            - staging
        ```
        
        이 예시는 `app 라벨`이 `my-app` 또는 `another-app` 중 하나이며, `environment` 라벨이 
        
        `staging`이 아닌 **모든 객체를 선택**한다.
        

- `Taint & Tolerations`
    - `Taint`
        - `Taint`는 `Node`에 적용되어 특정 조건이 만족되지 않으면 해당 `Node`에 `Pod`가 스케줄링되지 않도록 하는 **제약 조건이다.**
            
            `Taint`는 `Node`에 **"오염"을 시키는 역할**을 하며, 이를 통해 특정 `Node`에서 특정 `Pod`의 실행을 **방지**할 수 있다.
            
        
    - `Taint`의 **구성 요소**
        - **키(Key)**: Taint의 식별자
        - **값(Value)**: 선택 사항이며, 추가적인 정보를 제공
        - **효과(Effect)**: Taint의 작동 방식을 정의
            - `NoSchedule`: 이 `Taint`를 허용하지 않는 `Pod`는 **스케줄링되지 않는다.**
            - `PreferNoSchedule`: 이 `Taint`를 허용하지 않는 `Pod`가 가능한 한 스케줄링되지 않도록 한다.
            - `NoExecute`: 이 Taint를 허용하지 않는 Pod는 `Node`에서 **즉시 제거**된다.
    
    - `Taint` 적용 방법
        
        ```yaml
        # node1에 key=value라는 Taint를 추가하고 NoSchedule 효과를 적용한다.
        # key=value Toleration이 없는 Pod는 이 Node에 스케줄링되지 않는다.
        kubectl taint nodes node1 key=value:NoSchedule
        ```
        
    
    - `Toleartion`
        - `Toleration`은 `Pod`에 적용되어 특정 `Taint`를 허용할 수 있게 한다.
            
            `Toleration`은 `Taint`에 대응하여 `Pod`가 `Taint`가 있는 `Node`에 스케줄링될 수 있도록 
            
            **허용하는 역할**을 합니다.
            
        
    - `Toleration`의 구성 요소
        - **키(Key)**: `Toleration`이 대응하는 Taint의 **식별자**
        - **값(Value)**: 선택 사항이며, `Taint`의 **값을 지정**
        - **효과(Effect)**: `Toleration`이 대응하는 Taint의 **효과를 정의**
        - **Operator**: `Toleration`의 동작 방식을 정의
            - `Equal`:  **Taint**의 키와 값이 `일치`하는 경우 허용
            - `Exists`: **Taint**의 `키만 일치`하면 허용
    
    - `Toleration` 적용 예시
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: my-pod
        spec:
          tolerations:
          - key: "key"
            operator: "Equal"
            value: "value"
            effect: "NoSchedule"
          containers:
          - name: my-container
            image: my-image
        ```
        
        위 예시는 `key=value`라는 `Taint`를 가진 `Node`에 `my-pod`가 **스케줄링 될 수 있도록 허용**한다.
        

- `Node Selector`
    - `설명`
    
    `Node Selector`는 `Kubernetes`에서 **Pod**가 특정 **Node**에 `스케줄링`되도록 하는 방법 중 하나이다.
    
    이는 `Pod`가 실행될 `Node`를 선택할 때 **라벨을 기반으로 필터링하는 간단하고 직관적인 방법이다.**
    
    `Node Selector`를 사용하면 `Pod`가 특정 라벨을 가진 `Node`에서만 `실행`되도록 할 수 있다.
    
    - **Node Selector의 동작 방식**
        
        `Node Selector`는 `Pod` 스펙에서 `nodeSelector` 필드를 사용하여 정의된다.
        
        이 필드는 `키-값 쌍`의 형태로 `Node 라벨`을 지정하며, 지정된 라벨을 가진 `Node`에만 `Pod`가 
        
        **스케줄링**된다.
        
    
    - **Node Selector 구성 예시**
        1. `Node 라벨 설정` : Node에 라벨을 설정
        
        ```bash
        kubectl label nodes node1 disktype=ssd
        ```
        
        1. `Pod 스펙`에서 **Node Selector** 사용 : Pod 스펙에서 `nodeSelector` 필드를 사용하여 라벨을 지정
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: my-pod
        spec:
          containers:
          - name: my-container
            image: my-image
          nodeSelector:
            disktype: ssd
        ```
        
        위의 예시에서는 `disktype=ssd` 라벨을 가진 `Node`에서만 `my-pod`가 실행된다.
        
    
    - `Node Selector`의 단점
        - `단순한 필터링` : **Node Selector**는 단순한 **키-값 매칭**만 지원하므로, 보다 `복잡한 스케줄링` 요구사항을 충족하기는 어렵다.
        - `유연성 부족` : 복잡한 조건이나 **여러 라벨 조건**을 동시에 적용할 수 없다.

- `Node Affinity`
    - **설명**
        
        `Node Affinity`는 `Kubernetes`에서 `Pod`가 특정 조건을 만족하는 `Node`에 스케줄링되도록 하는 강력하고 유연한 메커니즘이다. 
        
        `Node Affinity`는 `Node Selector`보다 더 **복잡한 스케줄링 요구 사항을 처리**할 수 있으며, 여러 조건을 `논리적`으로 `결합`하여 사용할 수 있다.
        

- Node Affinity의 구성 요소

`Node Affinity`는 주로 세 가지 `유형`으로 나눌 수 있다.

1. **RequiredDuringSchedulingIgnoredDuringExecution**
2. **PreferredDuringSchedulingIgnoredDuringExecution**
3. **RequiredDuringSchedulingRequiredDuringExecution**

이 세 유형은 각각 `필수 조건` 또는 `선호 조건`을 나타낸다.

### 1. RequiredDuringSchedulingIgnoredDuringExecution

이 유형의 `Node Affinity`는 `Pod`가 스케줄링될 때 **반드시 만족해야 하는 조건을 정의한다.** 

이 조건이 만족되지 않으면 `Pod`는 해당 `Node`에 스케줄링되지 않는다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd

```

위 예시에서는 `disktype=ssd` 라벨을 가진 Node에만 `my-pod`가 스케줄링된다.

### 2. PreferredDuringSchedulingIgnoredDuringExecution

이 유형의 `Node Affinity`는 `Pod`가 `스케줄링`될 때 선호하지만 **필수는 아닌 조건**을 정의한다. 

여러 `Node`가 조건을 만족하는 경우, 선호 조건을 만족하는 `Node`에 더 높은 우선순위로 **스케줄링**됩니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
            - key: environment
              operator: In
              values:
              - production

```

위 예시에서는 `environment=production` 라벨을 가진 `Node`가 선호되지만, 반드시 필요한 것은 아니다.

### 3. RequiredDuringSchedulingRequiredDuringExecution

이 유형의 `Node Affinity`는 `Pod`가 `스케줄링`될 때와 `실행 중`에도 반드시 만족해야 하는 조건을 정의한다.

조건이 만족되지 않으면 `Pod`는 해당 `Node`에 **스케줄링**되지 않으며, 실행 중에 조건이 변경되면 `Pod`는 해당 `Node`에서 **제거된다.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
  affinity:
    nodeAffinity:
      requiredDuringSchedulingRequiredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd

```

위 예시에서는 `disktype=ssd` 라벨을 가진 `Node`에서만 `my-pod`가 실행되며, **실행 중**에도 이 조건을 계속 

`만족`해야 한다.

- `Node Affinity` 연산자

`Node Affinity`는 다양한 연산자를 사용하여 조건을 설정할 수 있다.

- **`In`** : 지정된 값 중 하나와 일치하는 경우
- **`NotIn`** : 지정된 값 중 하나와 일치하지 않는 경우
- **`Exists`** : 라벨 키가 존재하는 경우
- **`DoesNotExist`** : 라벨 키가 존재하지 않는 경우
- **`Gt`** : 라벨 값이 지정된 값보다 큰 경우
- **`Lt`** : 라벨 값이 지정된 값보다 작은 경우

- `Node Affinity` 예시

아래는 `disktype=ssd` 라벨을 필수 조건으로, 

`environment=production` 라벨을 선호 조건으로 설정한 Pod 정의다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
            - key: environment
              operator: In
              values:
              - production

```

- `Node Affinity`의 사용 사례
    - **리소스 제약이 있는 워크로드**: `특정 하드웨어 자원`(GPU, SSD 등)이 필요한 워크로드를 해당 자원을 가진 `Node`에 스케줄링.
    - **환경 분리**: 개발, 테스트, 프로덕션 환경을 `Node 라벨`로 구분하고, 각 환경에 맞는 **워크로드를 스케줄링**.
    - **데이터 로컬리티**: 데이터베이스와 같은 `애플리케이션`이 데이터가 `로컬`로 저장된 `Node`에서 실행되도록 보장.
    
- `Node Affinity`와 `Taint/Toleration` 비교
    
    `Node Affinity`와 **Taint/Toleration**은 모두 `Pod`의 `스케줄링`을 제어하는 `메커니즘`이지만, 사용 목적과 방식이 다르다.
    
    - **`Node Affinity`** : 특정 조건을 만족하는 `Node`에 `Pod`를 스케줄링하는 것을 `선호`하거나 `필수`로 설정.
    - **`Taint/Toleration`** : 특정 `Node`에 `Pod`가 스케줄링되지 않도록 제약을 두고, 특정 조건을 만족하는 `Pod`만 예외적으로 허용.

- `결론`
    
    `Node Affinity`는 `Kubernetes`에서 `Pod`를 특정 `Node`에 스케줄링하는 데 강력한 유연성을 제공한다.
    
    `다양한 조건`과 `연산자`를 사용하여 **복잡한 스케줄링 요구 사항을 충족**할 수 있으며, 필수 조건과 선호 조건을 통해 `스케줄링 정책`을 **세밀하게 조정**할 수 있다.
    
    이를 통해 `클러스터 리소스`를 효율적으로 관리하고, `애플리케이션`의 성능과 안정성을 높일 수 있다.
    

- `Resource Requests`
    - **설명**
    
    `Kubernetes`에서 `리소스 요청(Resource Request)`은 `Pod`가 특정 `Node`에 **스케줄링**될 때 필요한 최소한의 **리소스 양**을 지정하는 **설정이다.** 
    
    이는 **CPU**와 **메모리**와 같은 리소스를 **효율적으로 관리**하고, 각 `Pod`가 적절한 `Node`에 `배치`되도록 
    
    보장한다. `리소스 요청`은 `Pod 스펙`의 **컨테이너 섹션에서 정의된다.**
    

- `Resource Request`의 구성 요소
    
    주요 리소스 요청은 **CPU**와 **메모리**에 대해 정의된다.
    
    - **CPU** : `1 CPU`는 클러스터의 전체 `CPU 코어` 중 하나를 의미한다.
        
         0.5 CPU는 1/2 코어를 의미하고, `0.1 CPU`는 **1/10 코어**를 의미한다.
        
    - 메모리 : 메모리는 바이트 단위로 요청된다.
        
        일반적으로 `Mi (Mebibytes)`나 `Gi (Gibibytes)` 단위를 사용한다.
        

- `Resource Request` 설정 예시
    
    다음은 `리소스 요청`을 설정하는 예시다.
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      containers:
      - name: my-container
        image: my-image
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
    
    ```
    
    위 예시에서는 `my-container`가 최소 **64MiB의 메모리**와 **250m (0.25) CPU**를 요청하고 있다.
    

- `Resource Request`의 사용 사례
    - **리소스 보장**: `리소스 요청`을 설정하면, `Pod`가 최소한의 리소스를 `보장`받을 수 있다.
        
        이는 `클러스터`의 `안정성`을 높이고, `리소스 경합`을 줄이는 데 `도움`이 된다.
        
    - **적절한 스케줄링**: `스케줄러`는 `리소스 요청`을 기반으로 적절한 `Node`를 선택하여 `Pod`를 배치한다.
        
        이는 `리소스`가 충분히 있는 `Node`에 `Pod`가 배치되도록 `보장`한다.
        
    - **오버커밋 방지**: `리소스 요청`을 통해 특정 리소스 이상을 할당하지 않도록 `제한`할 수 있어,
        
        `Node`의 리소스 `오버커밋`을 `방지`할 수 있다.
        
    
- `Resource Limit`과의 관계
    
    `리소스 요청`과 함께 `리소스 제한`(Resource Limit)을 설정할 수도 있다. 
    
    `리소스 제한`은 컨테이너가 사용할 수 있는 `리소스`의 최대값을 정의한다. `리소스 요청`은 최소 보장값, 리소스 제한은 `최대 사용값`을 설정한다.
    

- `리소스 요청`과 `제한 설정` 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

```

위 예시에서는 `my-container`가 최소 64MiB의 메모리와 250m CPU를 요청하며, 최대 128MiB의 메모리와 500m CPU를 사용할 수 있다.

- `Daemon Sets`
    - `설명`
    
    `DaemonSet`은 `Kubernetes`에서 모든 (또는 선택된) `Node`에서 특정 `Pod`가 하나씩 실행되도록 보장하는 `컨트롤러`이다.
    
    `DaemonSet`은 클러스터의 각 `Node`에서 데몬(백그라운드 서비스) 역할을 하는 `Pod`를 배포하고 관리하는 데 사용된다.
    

- `DaemonSet` 의 주요 기능
    - **Node 당 하나의 Pod** : `DaemonSet`은 클러스터의 각 `Node`마다 하나의 `Pod`가 실행되도록 보장 한다. `Node`가 클러스터에 추가되면, `DaemonSet`은 자동으로 해당 `Node`에도 `Pod`를 생성한다.
    - **자동 업데이트**: `DaemonSet`에 정의된 `Pod` 템플릿이 변경되면, 모든 관련 `Pod`가 자동으로 업데이트됩니다.
    - **특정 Node 선택**: `NodeSelector`, `Node Affinity`, `Taints`와 `Tolerations` 등을 사용하여 특정 `Node`에만 `DaemonSet`을 배포할 수 있습니다.
    
- `DaemonSet` 사용 사례
    - **Node 모니터링** : 각 `Node`에서 **모니터링 에이전트**(Prometheus Node Exporter 등)를 실행하여 `시스템 상태`를 `수집`하고 `보고`한다.
    - **로그 수집** : 각 `Node`에서 **로그 수집기**(Fluentd, Logstash 등)를 실행하여 `애플리케이션 로그`를
        
        중앙으로 수집한다.
        
    - **네트워크 플러그인** : 각 `Node`에서 **네트워크 플러그인(CNI 플러그인)**을 실행하여 클러스터 네트워크를 구성한다.

- `DaemonSet` 예시
    
    다음은 각 `Node`에서 로그 수집기를 실행하는 `DaemonSet`의 예시이다.
    
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: log-collector
      namespace: kube-system
    spec:
      selector:
        matchLabels:
          name: log-collector
      template:
        metadata:
          labels:
            name: log-collector
        spec:
          containers:
          - name: log-collector
            image: fluentd:latest
            resources:
              requests:
                cpu: 100m
                memory: 200Mi
              limits:
                cpu: 200m
                memory: 400Mi
          nodeSelector:
            logging: "true"
    
    ```
    
    위 예시에서 DaemonSet은 `logging: "true"` 라벨을 가진 
    
    `Node`마다 `Fluentd` 로그 수집기를 실행한다.
    

- `DaemonSet`과 `ReplicaSet` 비교
    - **`ReplicaSet`** : **지정된 수의 Pod 복제본을 유지**합니다. `Pod`의 수를 수동으로 조정할 수 있으며, 특정 `Node`에 배포되지 않는다.
    - **`DaemonSet`** : 모든 (또는 선택된) `Node`에 하나의 `Pod`를 실행합니다. `Node`가 추가되거나 제거될 때 자동으로 `Pod`를 `생성`하거나 `삭제`한다.
    
- `DaemonSet`과 `Taints` 및 `Tolerations`
    
    `DaemonSet`은 `Taints`와 `Tolerations`를 사용하여 특정 `Node`에만 배포할 수 있다. 
    
    예를 들어, 다음과 같이 특정 `Node`에 `Taint`를 설정한 후 `DaemonSet`에서 이를 `Tolerate` 하도록 설정할 수 있다.
    
- `Node`에 `Taint` 설정

```bash
kubectl taint nodes node1 dedicated=log-collector:NoSchedule

```

- `DaemonSet`에서 `Toleration` 설정:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "log-collector"
        effect: "NoSchedule"
      containers:
      - name: log-collector
        image: fluentd:latest

```

위 예시에서 `DaemonSe`t은 `dedicated=log-collector:NoSchedule` **Taint를 가진 Node에도 배포된다.**

- `Static Pods`
    - **설명**
    
    `Static Pod`는 `kubelet`에 의해 **직접 관리되는 Pod**이다.
    
    `Kubernetes API`서버에 의해 관리되지 않고, `kubelet`이 실행 중인 각 `Node`에 있는 특정 파일 
    
    경로에서 정의된다. 이 파일이 수정되면, `kubelet`은 해당 `Pod`를 생성, 업데이트 또는 삭제한다.
    
    `Static Pods`는 주로 클러스터를 `부팅`하거나 중요한 시스템 서비스를 `실행`하는 데 사용된다.
    

- `Static Pods`의 주요 특징
    - **kubelet에 의해 관리됨**: `Static Pods는 kubelet`이 직접 관리한다.
        
        `kubelet`은 해당 파일을 읽고 **Pod를 생성**하거나 `삭제`한다.
        
    - **API 서버에 등록됨**: `kubelet`은 **Static Pod**를 실행할 때, 해당 `Pod`를 `Kubernetes API` 서버에
        
        자동으로 등록한다. 이를 통해 다른 `Kubernetes` 도구들이 해당 `Pod`의 상태를 볼 수 있다.
        
    - **파일 기반 정의**: `Static Pod`는 특정 파일 경로에서 정의되며, 일반적으로 `/etc/kubernetes/manifests` 디렉토리에 위치한다.

- `Static Pods`의 사용 사례
    - **클러스터 부트스트랩**: `클러스터 초기화` 과정에서 **중요한 시스템 구성 요소**를 실행할 때 사용할 수 있다.
        
        예를 들어, `API 서버`, `etcd` 등을 **Static Pod로 설정**할 수 있다.
        
    - **중요 시스템 서비스**: `클러스터` 내에서 항상 실행되어야 하는 중요한 서비스를 `Static Pod`로 `설정`한다.
    
- `Static Pods` 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-pod-example
  namespace: default
spec:
  containers:
  - name: static-container
    image: busybox
    command: ['sh', '-c', 'echo Hello, Kubernetes! && sleep 3600']

```

이 YAML 파일을 `/etc/kubernetes/manifests/static-pod-example.yaml` 경로에 저장하면, 

`kubelet`은 해당 파일을 읽고 Pod를 생성한다.

- `Static Pods` 설정 방법
    1. **Static Pod 정의 파일 생성** : Static Pod에 대한 YAML 정의 파일을 작성한다.
    2. **파일 저장** : 정의 파일을 `kubelet`이 읽을 수 있는 디렉토리에 저장한다. 기본적으로 `/etc/kubernetes/manifests` 디렉토리를 사용한다.
    3. **kubelet 설정 확인** : `kubelet` 설정에서 `-pod-manifest-path` 옵션이 올바르게 설정되었는지 확인 하고, 옵션은 `kubelet`이 `Static Pod` 정의 파일을 읽을 디렉토리를 지정해주면 된다.
        
        ```bash
        kubelet --pod-manifest-path=/etc/kubernetes/manifests
        
        ```
        

- `Static Pods`와 `DaemonSet` 비교
    - **Static Pods** : `kubelet`에 의해 직접 관리되고, `파일 기반`으로 정의된다. `Kubernetes API` 서버와 **독립적으로 동작**해.
    - **DaemonSet** : `Kubernetes API` 서버에 의해 관리되고, 모든 (또는 선택된) `Node`에서 `Pod`를 실행하며, **더 많은 기능과 유연성을 제공한다.**

- `Multiple Scheduler`
    - **설명**
    
    `Multiple Scheduler`는 `Kubernetes 클러스터`에서 기본 스케줄러 외에도 사용자 정의 스케줄러를 추가로 실행하는 방식이다.
    
    이를 통해 다양한 `스케줄링 요구 사항`을 처리할 수 있고, 여러 스케줄러를 사용하면 각 `스케줄러`가 특정 유형의 `Pod`에 대해 **다른 스케줄링 로직을 적용**할 수 있다.
    

- `Multiple Scheduler`의 주요 특징
    - **다양한 스케줄링 전략**: 기본 스케줄러 외에 `사용자 정의 스케줄러`를 추가하여 다양한 스케줄링 로직을 적용할 수 있다.
    - **Pod에 스케줄러 지정**: 각 `Pod`는 어떤 스케줄러를 사용할지 `명시`할 수 있으며, 이를 통해 `Pod`마다 **다른 스케줄링 전략을 적용**할 수 있다.
    - **유연한 리소스 관리**: 다양한 스케줄러를 통해 `클러스터 자원`을 더 `유연`하게 `관리`하고 `최적화`할 수 있다.

- `Multiple Scheduler` 설정 방법
    1. **사용자 정의 스케줄러 생성**: 사용자 정의 스케줄러를 작성하고, 이를 `컨테이너 이미지`로 빌드한다. 
        
        `사용자 정의 스케줄러`는 Kubernetes 기본 스케줄러와 동일한 `스케줄링 로직`을 따르지만, 사용자 정의 `로직`을 `추가`할 수 있다.
        
        `사용자 정의 스케줄러`를 직접 만들지 않고, `default-scheduler`가 사용하는 이미지를 써도 된다.
        
    2. **`사용자 정의 스케줄러` 배포**: 사용자 정의 스케줄러를 `Kubernetes` 클러스터에 배포한다.
        
        이를 위해 `Deployment`를 사용할 수 있다.
        

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      component: my-custom-scheduler
  template:
    metadata:
      labels:
        component: my-custom-scheduler
    spec:
      containers:
      - name: my-custom-scheduler
        image: my-custom-scheduler-image
        command:
        - my-custom-scheduler
        - --scheduler-name=my-custom-scheduler
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        volumeMounts:
        - name: kubeconfig
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: kubeconfig
        hostPath:
          path: /etc/kubernetes/scheduler.conf

```

1. **Pod에 스케줄러 지정**: 특정 `Pod`에 사용자 정의 스케줄러를 사용하도록 설정하려면,  `Pod 스펙`에서 `spec.schedulerName` 필드를 추가하면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: my-container
    image: my-image

```

- `Multiple Scheduler`의 사용 사례
    - **특정 워크로드 스케줄링**: 특정 유형의 `워크로드`에 대해 다른 스케줄링 로직을 적용하고 싶을 때 사용한다.예를 들어, `머신러닝 작업`은 `GPU 자원`이 있는 `Node`에만 스케줄링하도록 사용자 정의 스케줄러를 설정할 수 있다.
    - **특정 리소스 최적화**: `CPU 사용량`이 높은 작업과 `메모리 사용량`이 높은 작업을 각각 다른 스케줄러로 처리하여 리소스 사용을 최적화할 수 있다.
    - **특별한 스케줄링 요구 사항**: 특정 `비즈니스 로직`이나 정책에 따라 `Pod`를 `배치`해야 하는 경우 사용자 정의 스케줄러를 통해 이를 구현할 수 있다.

- `Scheduler Plugin`
    - **설명**
    
    `Kubernetes` 스케줄러 플러그인은 **기본 스케줄러의 기능을 확장**하거나 **변경**할 수 있는 메커니즘이다. 
    
    이를 통해 사용자는 `특정 요구 사항`에 맞는 **스케줄링 로직**을 구현할 수 있다. 
    
    플러그인은 `기본 스케줄러`의 다양한 단계에 `후크`를 추가하여 `커스터마이징`된 동작을 삽입할 수 있게 해준다.
    

- `스케줄러 플러그인`의 주요 기능
    - **확장성**: `기본 스케줄러`의 기능을 **플러그인을 통해 확장**할 수 있고, 이를 통해 더 **복잡한 스케줄링** 요구 사항을 처리할 수 있다.
    - **유연성**: `플러그인`은 다양한 `스케줄링 단계`에서 동작하도록 설정할 수 있으며, 이를 통해 스케줄링 과정의 특정 부분을 `커스터마이징`할 수 있다.
    - **커스터마이징**: 특정 `비즈니스 로직`이나 `정책`을 스케줄러에 적용할 수 있게 해준다.
        
        이를 통해 클러스터의 `리소스 사용`을 최적화하고, 특정 요구 사항을 `충족`시킬 수 있다.
        
    
- `스케줄러 플러그인`의 구조
    
    스케줄러 플러그인은 **여러 단계에서 동작**할 수 있다.
    
    - **Filter** : `Node` 후보군에서 `Pod`를 실행할 수 없는 Node를 필터링.
    - **Score** : 필터링된 `Node`에 점수를 매겨서 가장 적합한 `Node`를 **선택**.
    - **Reserve** : 선택된 `Node`에 리소스를 `예약`.
    - **Permit** : 선택된 `Node`에 `Pod`를 예약할 수 있는지 추가 `검사`를 `수행`.
    - **Pre-bind** : `Pod`가 `Node`에 바인딩되기 전에 추가 작업을 수행.
    - **Bind** : `Pod`를 `Node`에 바인딩.
    - **Post-bind** : `Pod`가 바인딩된 후 `추가 작업`을 수행.

- `Scheduler Profiles`
    - **설명**
    
    `스케줄러 프로파일`은 `단일 스케줄러 인스턴스` 내에서 여러 스케줄링 정책을 정의하고 **사용하는 기능**이다.
    
    이를 통해 동일한 `클러스터`에서 다양한 스케줄링 요구 사항을 처리할 수 있다.
    
    각 `프로파일`은 서로 **다른 스케줄링 플러그인 세트**를 사용하여 특정 유형의 `Pod`에 대해 다른 스케줄링 동작을 적용할 수 있다.
    

- 스케줄러 프로파일의 주요 특징
    - **다양한 스케줄링 정책** : 각 `프로파일`은 서로 다른 스케줄링 **플러그인과 설정**을 가질 수 있어, 다양한 스케줄링 `요구 사항`을 처리할 수 있다.
    - **Pod에 스케줄러 프로파일 지정** : 각 `Pod`는 특정 스케줄러 `프로파일`을 사용하도록 설정할 수 있어, 이를 통해 `Pod`마다 다른 스케줄링 전략을 적용할 수 있다.
    - **유연한 리소스 관리** : `스케줄러 프로파일`을 사용하여 클러스터 자원을 더 `유연`하게 관리하고 `최적화`할 수 있다.

- `스케줄러 프로파일` 설정 예시
    
    `스케줄러 프로파일`을 설정하려면, 스케줄러 설정 파일에서 여러 프로파일을 정의하고 각 `프로파일`에 사용할 플러그인을 `지정`하면 된다.
    
1. **스케줄러 설정 파일 작성**
    
    ```yaml
    apiVersion: kubescheduler.config.k8s.io/v1beta1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: default-scheduler
      plugins:
        filter:
          enabled:
          - name: "NodeResourcesFit"
          - name: "SimplePlugin"
        score:
          enabled:
          - name: "NodeResourcesFit"
          - name: "SimplePlugin"
    
    - schedulerName: high-priority-scheduler
      plugins:
        filter:
          enabled:
          - name: "NodeResourcesFit"
          - name: "PriorityPlugin"
        score:
          enabled:
          - name: "NodeResourcesFit"
          - name: "PriorityPlugin"
    
    ```
    
    위 예시에서 `default-scheduler`와 `high-priority-scheduler` 
    
    두 개의 프로파일이 정의되어 있어. 각 프로파일은 다른 플러그인 세트를 사용한다.
    

1. **Pod에 스케줄러 프로파일 지정**
    
    각 Pod는 `spec.schedulerName` 필드를 사용하여 특정 스케줄러 프로파일을 지정할 수 있다.
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-default-pod
    spec:
      schedulerName: default-scheduler
      containers:
      - name: my-container
        image: my-image
    
    ---
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-high-priority-pod
    spec:
      schedulerName: high-priority-scheduler
      containers:
      - name: my-container
        image: my-image
    
    ```
    
    위 예시에서 `my-default-pod`는 `default-scheduler` 프로파일을 사용하고, 
    
    `my-high-priority-pod`는 `high-priority-scheduler` 프로파일을 사용한다.
    

- `스케줄러 프로파일`의 사용 사례
    - **다양한 워크로드 처리**: `동일한 클러스터`에서 다양한 유형의 워크로드에 대해 **다른 스케줄링 정책을 적용**할 수 있다. 예를 들어, `배치 작업`과 `실시간 작업`을 다른 스케줄링 정책으로 처리할 수 있어.
    - **특정 정책 적용**: `특정 정책`이나 `요구 사항`에 맞게 **스케줄링 전략을 세밀하게 조정**할 수 있다. 예를 들어, `고우선순위` 작업에 대해 `특별한 스케줄링 전략`을 적용할 수 있다.
    - **리소스 최적화**: 다양한 `리소스 요구 사항`을 가진 `Pod`에 대해 최적화된 `스케줄링`을 제공할 수 있다.
