# 5️⃣ Week 5 (Section 7 - Item 160)

- `Authentication`

`Kubernetes Authentication`은 클러스터에 접근하는 사용자 및 시스템의 `신원`을 확인하는 

중요한 과정이다. `Kubernetes`는 아래와 같은 다양한 인증 방법을 제공한다.

**1. X.509 Client Certificates**

사용자가 `클라이언트 인증서`를 사용하여 클러스터에 인증하는 방식이다.

클라이언트 `인증서`와 `키`를 생성하고 `Kubernetes API` 서버에 신뢰할 수 있는 `CA 인증서`를 설정하는

방식으로 `설정`할 수 있다.

**2. Bearer Token**

정적 토큰 파일 또는 `OAuth2`와 같은 **외부 인증 제공자**를 통해 생성된 `토큰`을 `사용`하는 방식이다.

API 서버의 `—-token-auth-file` 플래그를 사용하여 `토큰 파일`을 지정해 사용할 수 있다.

**3. OpenID Connect Tokens**

`OIDC`를 사용하여 사용자를 인증하는 방식이다.

Google, Azure AD, `Keycloak` 등의 `제공자`와 통합이 가능하다.

API 서버의 여러 `OIDC` 관련 플래그를 설정하여 `OIDC` 인증을 구성해서 사용할 수 있다.

**4. Webhook Token Authentication**

Webhook을 사용하여 사용자 인증을 외부 시스템으로 위임하는 방식이다.

API 서버의 `—-authentication-token-webhook-config-file`  플래그를 사용하여  `Webhook` 

`구성 파일`을 지정하여 사용한다.

**5. Bootstrap Tokens**

`클러스터 부트스트랩 프로세스` 동안 사용되는 일회성 토큰이다.

`Kubeadm` 등을 사용하여 초기 클러스터 `부트스트랩` 시 설정한다.

**6. Static Password File**

간단한 `텍스트 파일`에 사용자 **이름**과 **비밀번호**를 저장하는 방식이다.

API 서버의 `—-basic-auth-file` 플래그를 사용하여 파일을 지정하여 사용할 수 있다.

**7. Static Token File**

간단한 `형태`의 토큰 기반 인증을 제공하는 방법 중 하나이다.

사용자와 토큰을 미리 정의된 파일에 저장하여 `Kubernetes API` 서버가 이 파일을 사용해 인증을 수행한다. 

보통 `CSV 파일`을 사용한다.

```bash
// 토큰 파일 예시
token,user,uid,"group1,group2..."

1a2b3c4d5e6f7g8h9i0j,johndoe,1001,"developers,admins"
5f4e3d2c1b0a9i8h7g6f,janedoe,1002,"developers"
```

`Kubernetes API` 서버에 `—-token-auth-file` 플래그를 추가하여 `CSV 파일`의 경로를 지정하면,

사용자가 보낸 `토큰`을 통해 `인증`을 `수행`하도록 설정된다.

- `Authorization`
    - **RBAC(Role-Based Access Control)**
        - `사용자`의 `역할`에 따라 접근 권한을 제어하는 방식을 말한다.
        - Role, ClusterRole, RoleBinding, ClusterRoleBinding으로 구성된다.
        
        ```yaml
        kind: Role
        apiVersion: rbac.authorization.k8s.io/v1
        metadata:
          namespace: default
          name: pod-reader
        rules:
        - apiGroups: [""]
          resources: ["pods"]
          verbs: ["get", "watch", "list"]
        ```
        
    
    - **ABAC(Attribute-Based Access Control)**
        - 사용자 속성을 기반으로 접근 권한을 제어하는 방식을 말한다.
        - JSON 파일로 정책을 정의한다.
        
        ```yaml
        [
          {
            "user": "johndoe",
            "namespace": "default",
            "resource": "pods",
            "readonly": true
          }
        ]
        ```
        
        - `—-authorization-policy-file` 플래그를 사용하여 정책 파일을 지정해 사용한다.
    
    - **Node**
        - `노드` 자체가 `클러스터`에서 수행할 수 있는 작업을 제어한다.
        - `—-authorization-mode=Node` 플래그를 사용하여 설정할 수 있다.
    
    - **Webhook Mode**
        - 외부 `Webhook`을 사용하여 `인가 요청`을 처리하는 방식이다.
        - `—-authorization-webhook-config-file` 플래그를 사용하여 `웹훅 구성 파일`을 지정하여 설정할 수 있다.
        
        ```yaml
        kind: Config
        apiVersion: v1
        clusters:
        - name: webhook
          cluster:
            server: https://example.com/authorize
        contexts:
        - context:
            cluster: webhook
          name: webhook
        current-context: webhook
        ```
        
    
    - **AlwaysAllow - Default**
        - `모든 요청`을 `허용`하는 설정이다.
        - 기본 값으로 사용되며, `—-authorization-mode=AlwaysAllow` 플래그를 사용하여 설정한다.
    
    - **AlwaysDeny**
        - 모든 요청을 거부하는 설정이다.
        - `—-authorization-mode=AlwaysDeny` 플래그를 사용하여 설정한다.
    
    - `Chaning`
        - `인가 모드`는 혼합하여 사용이 가능하며, 각 모드가 `체인 형태`로 순차적으로 적용된다.
        - 요청이 허용되거나 거부될 때까지 다음 `인가 모드`로 평가가 계속된다.
        - 첫 번째 `인가 모드`가 요청을 `허용`하면 나머지 인가 모드는 `평가`되지 않고, 마지막 `인가 모드`가 요청을 거부하면 `최종적`으로 요청이 거부된다.

- `Accounts`
    - **User Accounts**
        - 일반적인 `유저`의 계정을 뜻하며 `외부 시스템`을 통해 관리된다.
        - `Kubernetes`는 `User Accounts`를 자체적으로 관리하지 않으며, `인증`을 외부 시스템에 `위임`한다.
        - LDAP, OAuth, OpenID Connect 등 `외부 시스템`을 통해 관리된다.
    
    - **Service Accounts**
        - `클러스터` 내에서 실행되는 `애플리케이션`, `파드`, `컨트롤러` 등 시스템 구성 요소를 컨트롤 하기 위한 계정이다.
        - `Kubernetes`는 Service Account를 자체적으로 관리한다.
        - 각 네임스페이스에 기본 `Service Account`가 생성된다.
        - `RBAC`를 통해 `Service Account`의 권한을 관리한다.

- `TLS`
    - **Public Key**
        - 공개 키는 클라이언트와의 통신에서 사용되는 `공개 키 암호화 방식`의 일부이다.
        - 일반적으로 `*.crt`, `*.pem` 형식을 사용한다.
    - **Private Key**
        - `소유자`가 서명하거나 데이터를 `암호화`하는 데 사용된다.
        - `*.key`, `*-key.pem` 파일을 주로 사용한다.
        
- `Certificate Signing Request`
    - `CSR`은 인증서 서명 요청으로, 공개 키 인증서를 발급받기 위해 인증 기관에 제출하는 `데이터 패키지`다.
    - `CSR`은 인증서를 발급받고자 하는 주체의 정보를 포함하며, 이 정보는 주체가 제어하는 `비밀 키`와 연결된다.
    
    ```yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: my-user
    spec:
      request: $(cat my-csr.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - client auth
    ```
    
    - `CSR` 승인은 `kubectl certificate approve {csr-name}` 명령으로 할 수 있다.

- `Sign Certificates`
    - `인증서 서명`은 인증 기관이 다른 주체의 공개 키 인증서에 `디지털 서명`을 하여 그 `유효성`을 `보장`하는 것을 의미한다.
    - `CSR 제출 > CA 검증 > CSR 서명 > 인증서 발급 > 인증서 사용` 순으로 서명이 이루어진다.
    - `인증서 서명`은 공개 키 인프라에서 중요한 역할을 하며, `Kubernetes`에서도 CSR 리소스를 통해 인증서를 발급받아 사용할 수 있다.

- `CSR-APPROVING`
    - 제출된 `CSR(Certificate Signing Request)`를 **승인**하는 과정이다.
    - `Kubernetes` 클러스터 관리자가 수동으로 `검토`하고 `승인`하거나, `자동화`된 프로세스를 통해 `승인`할 수 있다.
    
- `CSR-SIGNING`
    - `CSR`을 인증기관(CA)이 서명하여 `최종 인증서`를 `발급`하는 과정이다.
    - `CSR Signing`은 주로 클러스터 내부의 `CA`에 의해 `자동`으로 `처리`된다.
    - 이를 통해 `안전`하고 `신뢰`할 수 있는 인증서를 발급받아 `클러스터` 내에서 사용할 수 있다.
    
- `KubeConfig`
    - `KubeConfig`는 Kubernetes 클러스터와 상호작용하기 위한 설정 정보를 포함하는 파일이다.
    - `kubectl`과 같은 Kubernetes 클라이언트 도구가 클러스터와 통신하는 데 필요한 정보를 제공한다.
    - `KubeConfig` 파일은 주로 `$HOME/.kube/config` 경로에 위치하며, 여러 클러스터와 사용자에 대한 정보를 관리할 수 있다.
    
    ### 주요 구성 요소
    
    1. **Clusters**
    2. **Contexts**
    3. **Users**
    
    ### 1. Clusters
    
    `Clusters` 섹션은 Kubernetes 클러스터에 대한 정보를 정의합니다. 여기에는 클러스터의 `API 서버 주소`와 `인증서 정보`가 포함된다.
    
    - **`name`** : 클러스터의 이름으로, `context`에서 참조할 때 사용
    - **`cluster`** : 클러스터의 상세 정보를 포함
        - **server**: Kubernetes API 서버의 주소(URL).
        - **certificate-authority**: 클러스터의 인증 기관(CA) 인증서 파일 경로.
            - 보통 `certificate-authority-data`를 사용하여 인증서를 직접 포함하기도 한다.
    
    ```yaml
    clusters:
    - name: my-cluster
      cluster:
        server: https://my-cluster.example.com
        certificate-authority: /path/to/ca.crt
    ```
    
    ### 2. Contexts
    
    `Contexts` 섹션은 클러스터, 사용자, 네임스페이스 정보를 결합하여 설정을 정의한다.
    
    이를 통해 사용자는 특정 `클러스터`에서 특정 사용자로 작업할 때 `사용`할 설정을 쉽게 `전환`할 수 있다.
    
    - **`name`** : context의 이름으로, `kubectl config use-context` 명령어로 전환할 때 사용
    - **`context`** : context의 상세 정보를 포함
        - **cluster** : 사용할 클러스터의 이름. `clusters` 섹션의 name과 일치해야 한다.
        - **user** : 사용할 사용자의 이름. `users` 섹션의 name과 일치해야 한다.
        - **namespace** : 기본 네임스페이스 (선택 사항).
    
    ```yaml
    contexts:
    - name: my-context
      context:
        cluster: my-cluster
        user: my-user
        namespace: default
    ```
    
    ### 3. Users
    
    `Users` 섹션은 `Kubernetes` 클러스터와 상호작용할 때 사용할 사용자 자격 증명을 정의한다.
    
    여기에는 인증서, 토큰, 사용자 이름 및 비밀번호 등이 `포함`될 수 있다.
    
    - **`name`** : 사용자의 이름으로, `context`에서 참조할 때 사용
    - **`user`** : 사용자의 `상세 정보` 를 포함합니다.
        - **client-certificate** : 클라이언트 인증서 파일 경로.
        - **client-key** : 클라이언트 비밀 키 파일 경로.
        - **token** : 인증 토큰.
        - **username** : 기본 인증에 사용할 사용자 이름.
        - **password** : 기본 인증에 사용할 비밀번호.
    
    ```yaml
    users:
    - name: my-user
      user:
        client-certificate: /path/to/client.crt
        client-key: /path/to/client.key
    
    ```
    
    ### KubeConfig 파일 예시
    
    ```yaml
    apiVersion: v1
    kind: Config
    clusters:
    - name: my-cluster
      cluster:
        server: https://my-cluster.example.com
        certificate-authority: /path/to/ca.crt
    contexts:
    - name: my-context
      context:
        cluster: my-cluster
        user: my-user
        namespace: default
    users:
    - name: my-user
      user:
        client-certificate: /path/to/client.crt
        client-key: /path/to/client.key
    current-context: my-context
    ```
    
- `API Groups`
    - /metrics
        - **설명** : 이 `엔드포인트`는 클러스터의 `메트릭 데이터`를 제공한다. 주로 `Prometheus`와 같은 모니터링 시스템에서 사용
        - **예시** : `https://<kube-apiserver>/metrics`
        - **사용** : 클러스터의 `성능` 및 `상태`를 `모니터링`하기 위해 사용
        
    - **/healthz**
        - **설명** : 이 `엔드포인트`는 API 서버의 상태를 확인하기 위해 사용된다. `건강 상태`를 체크하여 서비스 가용성을 보장한다.
        - **예시** : `https://<kube-apiserver>/healthz`
        - **사용** : API 서버가 정상적으로 `동작`하는지 확인하는 데 사용
        
    - **/version**
        - **설명** : 이 `엔드포인트`는 Kubernetes API 서버의 `버전 정보`를 제공
        - 예시 : `https://<kube-apiserver>/version`
        - **사용** : 클러스터의 `Kubernetes 버전`을 확인하는 데 사용
        
    - **/api**
        - **설명** : `Core API 그룹`의 엔드포인트이다. `기본 리소스`(예: Pods, Services, Namespaces)를 포함한다.
        - **예시** : `https://<kube-apiserver>/api/v1`
        - **사용** : 기본 `Kubernetes 리소스`와 상호작용하는 데 사용된다.
        
    - **/apis**
        - **설명** : `Named API` 그룹의 엔드포인트이다. 확장 및 `사용자 정의 리소스`를 포함한다.
        - **예시** : `https://<kube-apiserver>/apis/apps/v1`
        - **사용** : `확장된 Kubernetes 리소스`(예: Deployments, StatefulSets)와 상호작용하는 데 사용
        
    - **/logs**
        - **설명** : API 서버의 `로그 데이터`를 제공한다.
        - **예시** : `https://<kube-apiserver>/logs`
        - **사용** : API 서버의 로그를 확인하여 `문제`를 `진단`하고 `디버깅`하는 데 사용
    
    - `Core Group`
        - **설명** : `Core API` 그룹은 `Kubernetes`의 기본 API 그룹으로, 일반적으로 `/api` 엔드포인트를 통해 접근한다.
        - **포함 리소스** : Pods, Services, Namespaces, Nodes 등.
        - **예시** : `https://<kube-apiserver>/api/v1/pods`
        
    - `Named Group`
        - **설명** : `Named API` 그룹은 `확장 기능`과 `추가 기능`을 제공하는 API 그룹으로, `/apis` 엔드포인트를 통해 접근한다.
        - **포함 리소스** : Deployments, StatefulSets, DaemonSets, CustomResourceDefinitions 등.
        - **예제** : `https://<kube-apiserver>/apis/apps/v1/deployments`
    
    - `Kube Proxy ≠ Kubectl Proxy`
        - **Kube Proxy**:
            - Kubernetes 클러스터 내의 `네트워킹`을 관리하는 컴포넌트이다.
            - 클러스터 내부의 각 `노드`에서 실행되며, `네트워크 규칙`을 생성하여 서비스 간의 `통신`을 가능하게 한다.
            - **기능** : 네트워크 트래픽 라우팅, 서비스 로드 밸런싱, 네트워크 정책 적용.
            - **작동 방식** : `iptables`, `ipvs`, 또는 `유저 스페이스 모드`로 네트워크 트래픽을 처리한다.
            
        - **Kubectl Proxy**:
            - **설명** : 로컬 머신에서 실행되는 `프록시 서버`로, 로컬 클라이언트가 `Kubernetes API` 서버와 통신할 수 있도록 지원한다.
            - **기능** : `로컬 호스트`에서 API 서버로의 요청을 `프록시`하며, 인증 정보를 사용하여 안전하게 통신한다.
            - **사용 예시** : 개발자가 로컬 환경에서 `Kubernetes API`에 접근할 때 사용된다.
            - **명령어** : `kubectl proxy`
