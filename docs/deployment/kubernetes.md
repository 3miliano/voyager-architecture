# Kubernetes Deployment - Namespace Organization and Deployment Patterns

**Platform**: Kubernetes  
**Orchestration**: Kubernetes-native patterns  
**Purpose**: Container orchestration and resource management

## Overview

The Voyager platform is deployed entirely on Kubernetes, utilizing cloud-native patterns for scalability, reliability, and operational efficiency. Each service runs in its own namespace with dedicated resources and proper isolation.

## Namespace Organization

### Core Service Namespaces

#### Execution Layer
- **`jarvis`** - SPy program execution engine with KEDA autoscaling
- **`jeeves`** - Run orchestration and execution metadata management

#### Data Layer
- **`journal`** - Dynamic fact querying and time-series data management

#### Process Management
- **`grimoire`** - Process document storage and management

#### Book Services
- **`books`** - Shared book implementations and document processing

#### Chat & Agent Services
- **`threads`** - AI-powered chat orchestration and agent management

#### Integration Services
- **`herald`** - External event ingestion and trigger routing
- **`triage`** - Automated exception handling and retry logic

#### Platform Services
- **`envoy-gateway-system`** - Public edge with authentication and routing
- **`user`** - Identity and authorization management
- **`bdk`** - Book Development Kit
- **`llm`** - Model and prompt management with LLM services
- **`bifrost`** - Network management with Tailscale integration
- **`vault`** - Secret management and synchronization

#### Customer Deployments
- **`customer-a`**, **`customer-b`**, **`customer-*`** - Customer-specific book deployments

### Namespace Isolation Strategy

#### Resource Boundaries
- **CPU/Memory Quotas**: Each namespace has defined resource quotas
- **Storage Limits**: Persistent volume claim limits per namespace
- **Object Limits**: Limits on number of pods, services, and other objects
- **Network Policies**: NetworkPolicies for network isolation between namespaces

#### Security Isolation
- **RBAC**: Role-based access control per namespace
- **Service Accounts**: Dedicated service accounts for each service
- **Pod Security**: Pod security policies and security contexts
- **Secret Isolation**: Secrets scoped to their respective namespaces

## Helm-Based Deployment

### Helm Chart Organization

All Voyager services are deployed using Helm charts for consistent, versioned deployments. Each functional group has dedicated Helm charts:

#### Core Service Helm Charts
- **`helm-jarvis`** - SPy program execution engine deployment
- **`helm-jeeves`** - Run orchestration and metadata management
- **`helm-grimoire`** - Process management and AI integration
- **`helm-bdk`** - Book Development Kit and lifecycle management
- **`helm-llm`** - LLM services, model management, and AI proxy
- **`helm-user-and-organization-manager`** - Identity and authorization
- **`helm-authentication-and-authorization`** - Gateway auth components

#### AI Agent Helm Charts
- **`helm-process-designer`** - Process Designer agent and tools
- **`helm-spy-mapper`** - SPy Mapper agent and mapping tools
- **`helm-spy-writer`** - SPy Writer agent and code generation tools

#### Specialized Helm Charts
- **`helm-idp-book`** - Intelligent Document Processing (IdP) book implementations

### Helm Deployment Benefits
- **Version Management**: Consistent versioning across all service deployments
- **Configuration Management**: Centralized configuration management via values files
- **Rollback Capability**: Easy rollback to previous versions
- **Environment Consistency**: Consistent deployments across development, staging, and production
- **Dependency Management**: Proper dependency management between services
- **GitOps Integration**: Integration with GitOps workflows for automated deployments

### Helm Chart Structure
```yaml
# Example: helm-jeeves chart structure
charts/
  helm-jeeves/
    Chart.yaml
    values.yaml
    values-staging.yaml
    values-production.yaml
    templates/
      deployment.yaml
      service.yaml
      configmap.yaml
      secret.yaml
      hpa.yaml
      networkpolicy.yaml
      servicemonitor.yaml
    charts/
      postgresql/  # Sub-chart for database
```

### Deployment Commands
```bash
# Deploy Jeeves service
helm upgrade --install jeeves ./charts/helm-jeeves \
  --namespace jeeves \
  --create-namespace \
  --values ./charts/helm-jeeves/values-production.yaml

# Deploy AI services
helm upgrade --install llm-services ./charts/helm-llm \
  --namespace llm \
  --create-namespace \
  --values ./charts/helm-llm/values-production.yaml

# Deploy AI agents
helm upgrade --install process-designer ./charts/helm-process-designer \
  --namespace threads \
  --values ./charts/helm-process-designer/values-production.yaml
```

## Deployment Patterns

### Standard Service Deployment Pattern

#### Typical Service Architecture
```yaml
# Example: Jeeves deployment pattern
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jeeves-run-manager
  namespace: jeeves
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jeeves
      component: run-manager
  template:
    metadata:
      labels:
        app: jeeves
        component: run-manager
    spec:
      serviceAccountName: jeeves-service-account
      containers:
      - name: run-manager
        image: voyager/jeeves:latest
        ports:
        - containerPort: 8080
          name: grpc
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: jeeves-db-secret
              key: database_url
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 2Gi
        livenessProbe:
          grpc:
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          grpc:
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      - name: grpc-transcoder
        image: voyager/envoy-transcoder:latest
        ports:
        - containerPort: 8081
          name: http
        env:
        - name: UPSTREAM_GRPC_SERVICE
          value: "localhost:8080"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

### Database Deployment Pattern

#### PostgreSQL Pattern (Used by Jeeves, Grimoire, Threads, UOM)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jeeves-postgresql
  namespace: jeeves
spec:
  serviceName: jeeves-postgresql
  replicas: 1
  selector:
    matchLabels:
      app: postgresql
      service: jeeves
  template:
    metadata:
      labels:
        app: postgresql
        service: jeeves
    spec:
      containers:
      - name: postgresql
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: jeeves
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: jeeves-db-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: jeeves-db-secret
              key: password
        ports:
        - containerPort: 5432
          name: postgresql
        volumeMounts:
        - name: postgresql-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 2Gi
  volumeClaimTemplates:
  - metadata:
      name: postgresql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
      storageClassName: gp3
```

### Event-Driven Scaling Pattern (Jarvis)

#### KEDA Autoscaling Configuration
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: jarvis-spy-interpreter-scaler
  namespace: jarvis
spec:
  scaleTargetRef:
    name: jarvis-spy-interpreter
  minReplicaCount: 1
  maxReplicaCount: 20
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092
      consumerGroup: jarvis-spy-interpreter
      topic: spy_control
      lagThreshold: '5'
      offsetResetPolicy: latest
    authenticationRef:
      name: kafka-auth-trigger
```

### Controller Pattern (BDK, LLM Services)

#### Custom Controller Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: book-controller
  namespace: bdk
spec:
  replicas: 1
  selector:
    matchLabels:
      app: book-controller
  template:
    metadata:
      labels:
        app: book-controller
    spec:
      serviceAccountName: book-controller
      containers:
      - name: controller
        image: voyager/book-controller:latest
        env:
        - name: WATCH_NAMESPACE
          value: ""
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OPERATOR_NAME
          value: "book-controller"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        command:
        - /manager
        args:
        - --leader-elect
```

## Resource Management

### Resource Quotas per Namespace

#### Standard Service Namespace Quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: jeeves
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
    requests.storage: 100Gi
    pods: "20"
    services: "10"
    secrets: "20"
    configmaps: "20"
```

#### Customer Namespace Quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: customer-quota
  namespace: customer-acme
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    persistentvolumeclaims: "5"
    requests.storage: 50Gi
    pods: "10"
    services: "5"
```

### Horizontal Pod Autoscaling

#### HPA for Stateless Services
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: jeeves-hpa
  namespace: jeeves
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: jeeves-run-manager
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

## Storage Strategy

### Persistent Volume Classes

#### High-Performance Storage (Databases)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-high-iops
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

#### Standard Storage (General Purpose)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-standard
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### Backup Strategy

#### Velero Backup Configuration
```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *"
  template:
    includedNamespaces:
    - jeeves
    - grimoire
    - threads
    - user
    - journal
    excludedResources:
    - pods
    - replicasets
    storageLocation: default
    volumeSnapshotLocations:
    - default
    ttl: 720h0m0s
```

## Networking

### Service Mesh Architecture

#### Istio Service Mesh (Optional)
```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: voyager-control-plane
spec:
  values:
    global:
      meshID: voyager
      network: voyager-network
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
    ingressGateways:
    - name: istio-ingressgateway
      enabled: false # Using Envoy Gateway instead
```

### Network Policies

#### Namespace Isolation Policy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: jeeves
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-envoy-gateway
  namespace: jeeves
spec:
  podSelector:
    matchLabels:
      app: jeeves
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: envoy-gateway-system
    ports:
    - protocol: TCP
      port: 8081
```

## Security Configuration

### Pod Security Standards

#### Restricted Pod Security
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jeeves
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Service Account and RBAC

#### Service-Specific RBAC
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jeeves-service-account
  namespace: jeeves
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: jeeves
  name: jeeves-role
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jeeves-rolebinding
  namespace: jeeves
subjects:
- kind: ServiceAccount
  name: jeeves-service-account
  namespace: jeeves
roleRef:
  kind: Role
  name: jeeves-role
  apiGroup: rbac.authorization.k8s.io
```

## Monitoring and Observability

### Prometheus Monitoring

#### Service Monitor for Metrics Collection
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jeeves-monitor
  namespace: jeeves
spec:
  selector:
    matchLabels:
      app: jeeves
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### Logging Configuration

#### Fluent Bit DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: fluent-bit-config
          mountPath: /fluent-bit/etc/
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: fluent-bit-config
        configMap:
          name: fluent-bit-config
```

## Deployment Lifecycle

### GitOps with ArgoCD

#### ArgoCD Application
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: voyager-platform
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/voyager/platform-config
    targetRevision: main
    path: kubernetes
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Rolling Updates Strategy

#### Deployment Update Strategy
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jeeves-run-manager
  namespace: jeeves
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: run-manager
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
```

## Disaster Recovery

### Multi-Region Strategy
- **Primary Region**: Main deployment region
- **Secondary Region**: Disaster recovery region with data replication
- **Cross-Region Backup**: Regular cross-region backups
- **DNS Failover**: Automated DNS failover for disaster scenarios

### Recovery Procedures
1. **Database Recovery**: Restore from point-in-time backups
2. **Application Recovery**: Redeploy applications from GitOps repository
3. **Data Validation**: Validate data integrity after recovery
4. **Service Verification**: Comprehensive service verification after recovery

## Performance Optimization

### Node Affinity and Taints
- **Database Nodes**: Dedicated nodes for database workloads
- **Compute Nodes**: General-purpose nodes for application workloads
- **GPU Nodes**: Specialized nodes for AI/ML workloads (if needed)

### Resource Optimization
- **Vertical Pod Autoscaling**: VPA for right-sizing containers
- **Cluster Autoscaling**: Automatic node scaling based on demand
- **Resource Limits**: Proper resource limits to prevent resource contention
- **Quality of Service**: Appropriate QoS classes for different workloads
