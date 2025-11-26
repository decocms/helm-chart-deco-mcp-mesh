# Helm Chart: chart-deco-mcp-mesh

Helm Chart para deploy da aplicação MCP Mesh (Deco CMS) no Kubernetes. Este chart fornece uma solução completa e parametrizável para deploy da aplicação com suporte a persistência, autenticação, autoscaling e muito mais.

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Estrutura do Chart](#estrutura-do-chart)
- [Templates e Funcionamento](#templates-e-funcionamento)
- [Valores Configuráveis](#valores-configuráveis)
- [Exemplos de Uso](#exemplos-de-uso)
- [Manutenção e Atualização](#manutenção-e-atualização)
- [Troubleshooting](#troubleshooting)

## 🎯 Visão Geral

Este Helm Chart encapsula todos os recursos Kubernetes necessários para executar a aplicação MCP Mesh:

- **Deployment**: Aplicação principal com configurações de segurança
- **Service**: Exposição interna da aplicação
- **ConfigMap**: Configurações não sensíveis
- **Secret**: Dados sensíveis (autenticação)
- **PersistentVolumeClaim**: Armazenamento persistente para banco de dados
- **ServiceAccount**: Conta de serviço para o pod
- **HorizontalPodAutoscaler**: Autoscaling automático (opcional)

### Características Principais

- ✅ **Parametrizável**: Todas as configurações via `values.yaml`
- ✅ **Reutilizável**: Deploy em múltiplos ambientes com diferentes valores
- ✅ **Seguro**: Security contexts, non-root, capabilities drop
- ✅ **Flexível**: Suporte a volumes adicionais, tolerations, affinity
- ✅ **Observável**: Health checks, labels padronizados
- ✅ **Escalável**: HPA opcional para autoscaling

## 📦 Pré-requisitos

- Kubernetes 1.32+
- Helm 3.0+
- `kubectl` configurado para acessar o cluster
- StorageClass configurada (para PVC)

## 🚀 Instalação

### Instalação Básica

```bash
# Instalar com valores padrão
helm install deco-mcp-mesh .

# Instalar em um namespace específico
helm install deco-mcp-mesh . --namespace deco-mcp-mesh --create-namespace

# Instalar com valores customizados
helm install deco-mcp-mesh . -f meu-values.yaml
```

### Verificar Instalação

```bash
# Ver status do release
helm status deco-mcp-mesh

# Ver recursos criados
kubectl get all -l app.kubernetes.io/instance=deco-mcp-mesh

# Ver logs
kubectl logs -l app.kubernetes.io/instance=deco-mcp-mesh
```

### Desinstalar

```bash
helm uninstall deco-mcp-mesh
```

## ⚙️ Configuração

### Valores Principais

Os principais valores configuráveis estão em `values.yaml`.

Principais seções:

| Parâmetro | Descrição | Padrão |
|-----------|-----------|--------|
| `replicaCount` | Número de réplicas | `3` |
| `image.repository` | Repositório da imagem | `ghcr.io/decocms/admin/mesh` |
| `image.tag` | Tag da imagem | `latest` |
| `service.type` | Tipo do Service | `ClusterIP` |
| `persistence.enabled` | Habilitar PVC | `true` |
| `persistence.distributed` | PVC suporta ReadWriteMany | `true` |
| `persistence.accessMode` | Modo de acesso do PVC | `ReadWriteMany` |
| `persistence.storageClass` | StorageClass do PVC | `efs` |
| `autoscaling.enabled` | Habilitar HPA | `false` |
| `database.engine` | Banco (`sqlite`/`postgresql`) | `sqlite` |
| `database.url` | URL do banco quando PostgreSQL | `""` |

### Personalização de Valores

Crie um arquivo `custom-values.yaml`:

```yaml
replicaCount: 2

image:
  tag: "v1.2.3" # Exemplo

service:
  type: LoadBalancer
  port: 80

resources:
  requests:
    memory: "300Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

persistence:
  size: 10Gi
  storageClass: "efs"
```

Instale com valores customizados:

```bash
helm install deco-mcp-mesh . -f custom-values.yaml
```

## 📁 Estrutura do Chart

```
chart-deco-mcp-mesh/
├── Chart.yaml              # Metadados do chart
├── values.yaml             # Valores padrão
├── templates/              # Templates Kubernetes
│   ├── _helpers.tpl        # Funções auxiliares
│   ├── deployment.yaml     # Deployment da aplicação
│   ├── service.yaml        # Service
│   ├── configmap.yaml      # ConfigMap principal
│   ├── configmap-auth.yaml # ConfigMap de autenticação
│   ├── secret.yaml         # Secret
│   ├── pvc.yaml            # PersistentVolumeClaim
│   ├── serviceaccount.yaml # ServiceAccount
│   ├── hpa.yaml            # HorizontalPodAutoscaler
│   └── NOTES.txt           # Mensagens pós-instalação
└── README.md               # Este arquivo
```

## 🔧 Templates e Funcionamento

### 1. `_helpers.tpl` - Funções Auxiliares

Este arquivo define funções reutilizáveis usadas em todos os templates:

#### `chart-deco-mcp-mesh.name`
Retorna o nome base do chart:
```yaml
{{- define "chart-deco-mcp-mesh.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}
```
- Usa `nameOverride` se definido, senão usa `Chart.Name`
- Trunca para 63 caracteres (limite do Kubernetes)

#### `chart-deco-mcp-mesh.fullname`
Retorna o nome completo do recurso:
```yaml
{{- define "chart-deco-mcp-mesh.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
```
- **Importante**: Usa apenas o `Release.Name`, ignorando o nome do chart
- Se você instalar com `helm install deco-mcp-mesh`, todos os recursos terão nome `deco-mcp-mesh`

**Exemplo**:
```bash
helm install deco-mcp-mesh .
```
- Deployment: `deco-mcp-mesh`
- Service: `deco-mcp-mesh`
- ConfigMap: `deco-mcp-mesh-config`
- Secret: `deco-mcp-mesh-secrets`
- PVC: `deco-mcp-mesh-data`

#### `chart-deco-mcp-mesh.labels`
Gera labels padronizados:
```yaml
helm.sh/chart: chart-deco-mcp-mesh-0.1.0
app.kubernetes.io/name: chart-deco-mcp-mesh
app.kubernetes.io/instance: deco-mcp-mesh
app.kubernetes.io/version: latest
app.kubernetes.io/managed-by: Helm
```

#### `chart-deco-mcp-mesh.selectorLabels`
Labels usados para seleção (selectors):
```yaml
app.kubernetes.io/name: chart-deco-mcp-mesh
app.kubernetes.io/instance: deco-mcp-mesh
```

### 2. `deployment.yaml` - Deployment da Aplicação

#### Estrutura Condicional

```yaml
{{- if not .Values.autoscaling.enabled }}
replicas: {{ .Values.replicaCount }}
{{- end }}
```
- Se HPA estiver habilitado, não define `replicas` (HPA controla)

#### Estratégia de Deployment (Auto-detecção)

```yaml
strategy:
  type: {{ include "chart-deco-mcp-mesh.deploymentStrategy" . }}
```

O chart detecta automaticamente a estratégia de deployment adequada:
- **RollingUpdate**: usado quando PostgreSQL OU storage distribuído (ReadWriteMany)
- **Recreate**: usado por padrão para SQLite com ReadWriteOnce

Você pode sobrescrever definindo explicitamente `strategy.type` em `values.yaml`.

#### Variáveis de Ambiente

```yaml
env:
  - name: NODE_ENV
    valueFrom:
      configMapKeyRef:
        name: {{ include "chart-deco-mcp-mesh.fullname" . }}-config
        key: NODE_ENV
  - name: DATABASE_URL
    {{- if eq (lower (default "sqlite" .Values.database.engine)) "postgresql" }}
    valueFrom:
      secretKeyRef:
        name: {{ include "chart-deco-mcp-mesh.fullname" . }}-secrets
        key: DATABASE_URL
    {{- else }}
    valueFrom:
      configMapKeyRef:
        name: {{ include "chart-deco-mcp-mesh.fullname" . }}-config
        key: DATABASE_URL
    {{- end }}
```
- Referencia ConfigMap dinamicamente usando `fullname`
- **Segurança**: `DATABASE_URL` usa `secretKeyRef` quando PostgreSQL (credenciais sensíveis) ou `configMapKeyRef` quando SQLite (apenas caminho de arquivo)

#### Volumes Condicionais

```yaml
{{- if .Values.persistence.enabled }}
- name: data
  persistentVolumeClaim:
    claimName: {{ include "chart-deco-mcp-mesh.fullname" . }}-data
{{- else }}
- name: data
  emptyDir: {}
{{- end }}
```
- Se `persistence.enabled: false`, usa `emptyDir` (dados temporários)

#### Volumes Adicionais

```yaml
{{- with .Values.volumes }}
  {{- toYaml . | nindent 8 }}
{{- end }}
```
- Permite adicionar volumes customizados via `values.yaml`

#### Topology Spread Constraints

```yaml
{{- if and .Values.topologySpreadConstraints (gt (len .Values.topologySpreadConstraints) 0) }}
topologySpreadConstraints:
  {{- range .Values.topologySpreadConstraints }}
  - maxSkew: {{ .maxSkew }}
    topologyKey: {{ .topologyKey }}
    whenUnsatisfiable: {{ .whenUnsatisfiable }}
    labelSelector:
      {{- toYaml .labelSelector | nindent 12 }}
  {{- end }}
{{- end }}
```
- Distribui pods uniformemente entre zonas/disponibilidade
- **Importante**: `labelSelector` é obrigatório quando configurado

### 3. `service.yaml` - Service

```yaml
selector:
  {{- include "chart-deco-mcp-mesh.selectorLabels" . | nindent 4 }}
```
- Usa `selectorLabels` para conectar ao Deployment

```yaml
{{- if .Values.service.sessionAffinity }}
sessionAffinity: {{ .Values.service.sessionAffinity }}
{{- end }}
```
- Renderiza apenas se `sessionAffinity` estiver definido
- **Por padrão**: não há afinidade de sessão (requisições distribuídas entre todos os pods)
- **Se configurado**: `sessionAffinity: ClientIP` garante que requisições do mesmo IP sejam direcionadas ao mesmo pod

### 4. `configmap.yaml` - ConfigMap Principal

```yaml
data:
  NODE_ENV: {{ .Values.configMap.meshConfig.NODE_ENV | quote }}
  PORT: {{ .Values.configMap.meshConfig.PORT | quote }}
  {{- if ne (lower (default "sqlite" .Values.database.engine)) "postgresql" }}
  DATABASE_URL: {{ include "chart-deco-mcp-mesh.databaseUrl" . | trim | quote }}
  {{- end }}
```
- `| quote` garante que valores sejam strings válidas no YAML
- **Segurança**: `DATABASE_URL` só vai no ConfigMap quando for SQLite (caminho de arquivo, não sensível)
- Quando for PostgreSQL, `DATABASE_URL` vai no Secret (contém credenciais)

### 5. `configmap-auth.yaml` - ConfigMap de Autenticação

```yaml
auth-config.json: |
  {
    "emailAndPassword": {
      "enabled": {{ .Values.configMap.authConfig.emailAndPassword.enabled }}
    }
  }
```
- Gera JSON a partir de valores YAML
- Montado como arquivo no pod

### 6. `secret.yaml` - Secret

```yaml
{{- if not .Values.secret.secretName }}
apiVersion: v1
kind: Secret
...
stringData:
  BETTER_AUTH_SECRET: {{ .Values.secret.BETTER_AUTH_SECRET | quote }}
{{- end }}
```

#### Lógica de Criação do Secret

O chart suporta dois cenários de gerenciamento de secrets:

1. **Criar novo Secret** (padrão):
   - Se `secret.secretName` estiver vazio ou não definido, cria um novo Secret
   - Usa `stringData` (Helm codifica automaticamente para base64)
   - Nome do Secret: `{{ release-name }}-secrets`

2. **Usar Secret existente**:
   - Se `secret.secretName` estiver definido, **não cria** um novo Secret
   - O Deployment referencia o Secret existente especificado em `secretName`
   - Útil para usar secrets gerenciados por External Secrets Operator, Sealed Secrets, etc.

**Resumo da lógica**:
- Se `secret.secretName` vazio/indefinido → **cria** novo Secret
- Se `secret.secretName` definido → **não cria** Secret, apenas referencia o existente

### 7. `pvc.yaml` - PersistentVolumeClaim

```yaml
{{- if .Values.persistence.enabled -}}
{{- if not .Values.persistence.claimName }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "chart-deco-mcp-mesh.fullname" . }}-data
spec:
  accessModes:
    - {{ .Values.persistence.accessMode }}
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
{{- end }}
{{- end }}
```

#### Lógica de Criação do PVC

O chart suporta três cenários de persistência:

1. **Criar novo PVC** (padrão):
   ```yaml
   persistence:
     enabled: true
     claimName: ""  # ou omitir
   ```
   - Cria um novo PVC com o nome `{{ release-name }}-data`
   - Usa os parâmetros definidos em `persistence` (size, storageClass, accessMode)

2. **Usar PVC existente**:
   ```yaml
   persistence:
     enabled: true
     claimName: "meu-pvc-existente"
   ```
   - **Não cria** um novo PVC
   - Referencia o PVC existente especificado em `claimName`
   - Útil para reutilizar dados de instalações anteriores ou PVCs criados manualmente

3. **Sem persistência**:
   ```yaml
   persistence:
     enabled: false
   ```
   - **Não cria** PVC
   - O Deployment usa `emptyDir` (dados temporários, perdidos ao reiniciar o pod)

**Resumo da lógica**:
- Se `persistence.enabled: false` → sem PVC (usa `emptyDir`)
- Se `persistence.enabled: true` E `persistence.claimName` vazio/indefinido → **cria** novo PVC
- Se `persistence.enabled: true` E `persistence.claimName` definido → **não cria** PVC, apenas referencia o existente

### 8. `serviceaccount.yaml` - ServiceAccount

```yaml
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
...
{{- end }}
```
- Cria ServiceAccount apenas se `serviceAccount.create: true`

### 9. `hpa.yaml` - HorizontalPodAutoscaler

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
...
{{- end }}
```
- Cria HPA apenas se `autoscaling.enabled: true`
- Quando habilitado, remove `replicas` do Deployment

### 10. `NOTES.txt` - Mensagens Pós-Instalação

Exibe instruções após o install/upgrade:

```yaml
{{- if contains "ClusterIP" .Values.service.type }}
  echo "To access the application, run:"
  echo "  kubectl port-forward svc/$SERVICE_NAME 8080:80"
{{- end }}
```
- Mensagens diferentes baseadas no tipo de Service

## 📝 Valores Configuráveis

### Imagem

```yaml
image:
  repository: ghcr.io/decocms/admin/mesh
  pullPolicy: Always  # Always, IfNotPresent, Never
  tag: "latest"       # Sobrescreve Chart.AppVersion se definido
```

### Replicas e Estratégia

```yaml
replicaCount: 3  # Ignorado se autoscaling.enabled: true

strategy:
  # type: ""  # Deixe vazio para auto-detecção:
  #   - RollingUpdate: se database.engine=postgresql OU persistence.distributed=true OU accessMode=ReadWriteMany
  #   - Recreate: se SQLite com ReadWriteOnce (padrão)
  # rollingUpdate:
  #   maxSurge: 1
  #   maxUnavailable: 0
```

**Auto-detecção de Estratégia**: Se `strategy.type` estiver vazio ou não definido, o chart detecta automaticamente a estratégia adequada:
- **RollingUpdate**: usado quando `database.engine=postgresql` OU `persistence.distributed=true` OU `accessMode=ReadWriteMany`
- **Recreate**: usado por padrão para SQLite com ReadWriteOnce (quando apenas um pod pode montar o volume)

Você também pode definir explicitamente `strategy.type: "RollingUpdate"` ou `"Recreate"` se desejar sobrescrever a auto-detecção.

### Restrições de Escalabilidade

- `replicaCount > 1` **só é permitido** quando você possui storage distribuído (`persistence.distributed: true` ou `accessMode: ReadWriteMany`) **ou** está usando PostgreSQL (`database.engine: postgresql`).
- `autoscaling.enabled: true` exige a mesma condição acima (storage distribuído ou PostgreSQL).
- Caso não atenda a esses requisitos, mantenha `replicaCount: 1` e faça ajustes de capacidade via escalabilidade vertical (CPU/RAM).

### Service

```yaml
service:
  type: ClusterIP  # ClusterIP, NodePort, LoadBalancer
  port: 80
  targetPort: 3000
  # sessionAffinity: ""  # Opcional: "ClientIP" para afinidade de IP, ou omitir (padrão: nenhuma afinidade)
  # sessionAffinityConfig:
  #   clientIP:
  #     timeoutSeconds: 10800  # 3 horas
```

**Nota**: Por padrão, o service não possui afinidade de sessão (`sessionAffinity` não está definido), o que significa que as requisições serão distribuídas entre todos os pods disponíveis de forma round-robin. Se precisar de afinidade de IP (sticky sessions), descomente e configure `sessionAffinity: ClientIP`.

### Recursos

```yaml
resources:
  requests:
    memory: "300Mi"
    cpu: "250m"
  limits:
    memory: "600Mi"
    cpu: "500m"
```

### Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 4
```

### Persistência

```yaml
persistence:
  enabled: true
  storageClass: "efs"      # "" usa a padrão
  accessMode: ReadWriteMany
  size: 10Gi
  claimName: ""            # Se definido, usa PVC existente
  distributed: true        # Marque true se o PVC oferecer ReadWriteMany

**Importante**: marque `distributed: true` ou altere `accessMode` para `ReadWriteMany` quando utilizar storage distribuído (EFS, NFS, CephFS etc.). Sem isso, o chart bloqueará múltiplas réplicas e o uso do autoscaling.
```

### Banco de Dados

```yaml
database:
  engine: sqlite        # sqlite | postgresql
  url: ""               # Obrigatório quando engine=postgresql
```

- `sqlite`: usa o arquivo local `/app/data/mesh.db` (próprio para uma réplica).
- `postgresql`: exige `database.url` (ex.: `postgresql://user:pass@host:5432/db`) e dispensa storage compartilhado para escalar horizontalmente.

**Segurança**: O `DATABASE_URL` é armazenado de forma segura:
- **SQLite**: vai no ConfigMap (é apenas um caminho de arquivo, não sensível)
- **PostgreSQL**: vai no Secret (contém credenciais sensíveis como usuário e senha)

O Deployment referencia automaticamente o local correto (`configMapKeyRef` para SQLite ou `secretKeyRef` para PostgreSQL) baseado no `database.engine`.

### Autoscaling

```yaml
autoscaling:
  enabled: false
  minReplicas: 3
  maxReplicas: 6
  # targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

**Importante**: habilite o autoscaling apenas se `persistence.distributed: true` (ou `accessMode: ReadWriteMany`) ou se estiver usando PostgreSQL (`database.engine: postgresql`). Caso contrário, o chart falhará durante o render.

### ConfigMap

```yaml
configMap:
  meshConfig:
    NODE_ENV: "production"
    PORT: "3000"
    HOST: "0.0.0.0"
    BETTER_AUTH_URL: "http://localhost:8080"
    BASE_URL: "http://localhost:8080"
    # DATABASE_URL é preenchido automaticamente a partir de database.engine/url
  
  authConfig:
    emailAndPassword:
      enabled: true
    socialProviders:
      google:
        clientId: "your-google-client-id.apps.googleusercontent.com"
        clientSecret: "your-google-client-secret"
      github:
        clientId: "your-github-client-id"
        clientSecret: "your-github-client-secret"
    saml:
      enabled: false
      providers: []
    emailProviders:
      - id: "resend-primary"
        provider: "resend"
        config:
          apiKey: "your-resend-api-key"
          fromEmail: "noreply@example.com"
    inviteEmailProviderId: "resend-primary"
    magicLinkConfig:
      enabled: true
      emailProviderId: "resend-primary"
```

### Secret

O chart suporta três cenários de gerenciamento de secrets:

1. **Criar novo Secret** (padrão):
   ```yaml
   secret:
     secretName: ""  # ou omitir
     BETTER_AUTH_SECRET: "change-this-to-a-secure-random-string-at-least-32-chars"
   ```
   - Cria um novo Secret com o nome `{{ release-name }}-secrets`
   - Usa os valores definidos em `secret` (BETTER_AUTH_SECRET e opcionalmente DATABASE_URL para PostgreSQL)

2. **Usar Secret existente**:
   ```yaml
   secret:
     secretName: "meu-secret-existente"  # Nome do secret que já existe no cluster
     # BETTER_AUTH_SECRET não é necessário quando usando secret existente
   ```
   - **Não cria** um novo Secret
   - Referencia o Secret existente especificado em `secretName`
   - O Secret existente deve conter as chaves necessárias:
     - `BETTER_AUTH_SECRET` (obrigatório)
     - `DATABASE_URL` (obrigatório apenas se `database.engine=postgresql`)
   - Útil para usar secrets gerenciados por External Secrets Operator, Sealed Secrets, ou outros sistemas

3. **Sem Secret** (não suportado):
   - O Secret é sempre necessário para `BETTER_AUTH_SECRET`

**⚠️ Importante**: Gere um secret seguro:
```bash
openssl rand -base64 32
```

**Resumo da lógica**:
- Se `secret.secretName` vazio/indefinido → **cria** novo Secret
- Se `secret.secretName` definido → **não cria** Secret, apenas referencia o existente

### Security Context

```yaml
podSecurityContext:
  fsGroup: 1001
  fsGroupChangePolicy: "OnRootMismatch"

securityContext:
  runAsNonRoot: true
  runAsUser: 1001
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: false
```

### Node Selection

```yaml
nodeSelector:
  kubernetes.io/arch: amd64

tolerations: []
# - key: "env"
#   operator: "Equal"
#   value: "dev"
#   effect: "NoSchedule"

affinity: {}
# - podAntiAffinity:
#     preferredDuringSchedulingIgnoredDuringExecution:
#       - weight: 100
#         podAffinityTerm:
#           labelSelector:
#             matchLabels:
#               app.kubernetes.io/name: chart-deco-mcp-mesh
#           topologyKey: kubernetes.io/hostname
```

### Topology Spread Constraints

```yaml
# Topology Spread Constraints (opcional - deixe vazio [] para desabilitar)
# IMPORTANTE: labelSelector é obrigatório quando topologySpreadConstraints está configurado
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: chart-deco-mcp-mesh
        app.kubernetes.io/instance: deco-mcp-mesh
```

**Importante**: O `labelSelector` é obrigatório quando `topologySpreadConstraints` está configurado. Isso garante que os pods sejam distribuídos uniformemente entre zonas/disponibilidade, melhorando a alta disponibilidade da aplicação.

### Volumes Adicionais

```yaml
volumes: []
# - name: extra-config
#   configMap:
#     name: my-config

volumeMounts: []
# - name: extra-config
#   mountPath: "/etc/config"
#   readOnly: true
```

### Containers Extras no Pod

Você pode adicionar containers extras ao Pod (como sidecars, proxies, etc.) sem remover o container padrão da aplicação.  
O chart sempre mantém o container principal e **concatena** o que for definido em `extraContainers`:

```yaml
extraContainers: []
# - name: cloudsql-proxy
#   image: gcr.io/cloudsql-docker/gce-proxy:1.33.1
#   args:
#     - "/cloud_sql_proxy"
#     - "-instances=PROJECT:REGION:INSTANCE=tcp:5432"
```

- Se `extraContainers` não for definido ou estiver vazio, o Pod terá apenas o container padrão (comportamento atual).
- Se você definir `extraContainers`, todos esses containers serão adicionados ao mesmo Pod junto com o container principal.

### ServiceAccount

```yaml
serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""  # Se definido, usa este nome (não cria)
```

### Naming

```yaml
nameOverride: ""        # Substitui Chart.Name
fullnameOverride: ""    # Substitui Release.Name (tem prioridade)
```

## 💡 Exemplos de Uso

### Exemplo 1: Deploy Básico

```bash
helm install mesh .
```

### Exemplo 2: Deploy com Valores Customizados

```yaml
# production-values.yaml
replicaCount: 3

image:
  tag: "v1.0.0"

service:
  type: LoadBalancer

resources:
  requests:
    memory: "300Mi"
    cpu: "250m"
  limits:
    memory: "600Mi"
    cpu: "500m"

persistence:
  size: 10Gi
  storageClass: "efs"

configMap:
  meshConfig:
    NODE_ENV: "production"
    BASE_URL: "https://mesh.example.com"
```

```bash
helm install mesh-prod . -f production-values.yaml
```

### Exemplo 3: Deploy com Autoscaling

```yaml
# autoscaling-values.yaml
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 6
  targetMemoryUtilizationPercentage: 80

resources:
  requests:
    memory: "300Mi"
    cpu: "250m"
  limits:
    memory: "600Mi"
    cpu: "500m"
```

```bash
helm install mesh . -f autoscaling-values.yaml
```

### Exemplo 4: Deploy com PVC Existente

```yaml
# existing-pvc-values.yaml
persistence:
  enabled: true
  claimName: "existing-mesh-data"  # Nome do PVC que já existe no cluster
  # Quando claimName está definido, o chart NÃO cria um novo PVC
  # Apenas referencia o PVC existente especificado
```

```bash
# O PVC deve existir antes de instalar o chart
kubectl get pvc existing-mesh-data

# Instalar usando o PVC existente
helm install mesh . -f existing-pvc-values.yaml

# O Deployment será criado referenciando o PVC existente
# Nenhum novo PVC será criado por este chart
```

**Quando usar**:
- Migrar dados de uma instalação anterior
- Reutilizar dados entre diferentes releases do Helm
- Usar PVCs criados manualmente ou por outros processos

### Exemplo 5: Deploy sem Persistência (Desenvolvimento)

```yaml
# dev-values.yaml
persistence:
  enabled: false  # Usa emptyDir (dados temporários)

replicaCount: 1

resources:
  requests:
    memory: "300Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

```bash
helm install mesh-dev . -f dev-values.yaml
```

### Exemplo 6: Deploy com Nome Customizado

```bash
# Usa apenas o release name
helm install meu-mesh .

# Ou sobrescreve completamente
helm install meu-mesh . \
  --set fullnameOverride=mesh-customizado
```

### Exemplo 7: Deploy com Secret Existente

```yaml
# existing-secret-values.yaml
secret:
  secretName: "external-secrets-operator-secret"  # Nome do secret que já existe no cluster
  # BETTER_AUTH_SECRET não é necessário quando usando secret existente
  # O secret existente deve conter as chaves:
  #   - BETTER_AUTH_SECRET (obrigatório)
  #   - DATABASE_URL (obrigatório apenas se database.engine=postgresql)
```

```bash
# O Secret deve existir antes de instalar o chart
kubectl get secret external-secrets-operator-secret

# Verificar se contém as chaves necessárias
kubectl get secret external-secrets-operator-secret -o jsonpath='{.data}' | jq 'keys'

# Instalar usando o Secret existente
helm install mesh . -f existing-secret-values.yaml

# O Deployment será criado referenciando o Secret existente
# Nenhum novo Secret será criado por este chart
```

**Quando usar**:
- Usar secrets gerenciados por External Secrets Operator
- Usar Sealed Secrets ou outros sistemas de gerenciamento de secrets
- Compartilhar secrets entre diferentes releases do Helm
- Usar secrets criados manualmente ou por outros processos

## 🔄 Manutenção e Atualização

### Atualizar Valores

```bash
# Editar values.yaml ou criar novo arquivo
vim custom-values.yaml

# Atualizar release
helm upgrade deco-mcp-mesh . -f custom-values.yaml

# Ver histórico
helm history deco-mcp-mesh

# Rollback
helm rollback deco-mcp-mesh
```

### Atualizar Imagem

```bash
# Opção 1: Atualizar values.yaml e fazer upgrade
helm upgrade deco-mcp-mesh . \
  --set image.tag=v1.2.3

# Opção 2: Se pullPolicy: Always, apenas reiniciar
kubectl rollout restart deployment/deco-mcp-mesh
```

### Atualizar ConfigMap/Secret

```bash
# Editar values.yaml
vim values.yaml

# Atualizar
helm upgrade deco-mcp-mesh .

# Reiniciar pods para pegar mudanças
kubectl rollout restart deployment/deco-mcp-mesh
```

### Verificar Mudanças Antes de Aplicar

```bash
# Ver o que será gerado
helm template deco-mcp-mesh .

# Ver diff entre versões
helm diff upgrade deco-mcp-mesh .
```

### Backup do Banco de Dados

```bash
# Se usando PVC
POD=$(kubectl get pod -l app.kubernetes.io/instance=deco-mcp-mesh -o jsonpath='{.items[0].metadata.name}')
kubectl cp seu-namespace/$POD:/app/data/mesh.db ./backup-$(date +%Y%m%d).db
```

## 🐛 Troubleshooting

### Pod Não Inicia

```bash
# Ver eventos
kubectl describe pod -l app.kubernetes.io/instance=deco-mcp-mesh

# Ver logs
kubectl logs -l app.kubernetes.io/instance=deco-mcp-mesh

# Verificar PVC
kubectl get pvc -l app.kubernetes.io/instance=deco-mcp-mesh
```

### PVC Não Monta

```bash
# Verificar StorageClass
kubectl get storageclass

# Ver detalhes do PVC
kubectl describe pvc deco-mcp-mesh-data

# Verificar se pod pode montar (ReadWriteOnce permite apenas 1 pod)
kubectl get pods -l app.kubernetes.io/instance=deco-mcp-mesh
```

### Health Checks Falhando

```bash
# Verificar se endpoint /health existe
kubectl exec -it deployment/deco-mcp-mesh -- wget -O- http://localhost:3000/health

# Ajustar valores em values.yaml
livenessProbe:
  initialDelaySeconds: 60  # Aumentar se app demora para iniciar
```

### Service Não Conecta

```bash
# Verificar labels do Deployment
kubectl get deployment deco-mcp-mesh -o yaml | grep -A 5 labels

# Verificar selector do Service
kubectl get service deco-mcp-mesh -o yaml | grep -A 5 selector

# Verificar endpoints
kubectl get endpoints deco-mcp-mesh
```

### Imagem Não Puxa

```bash
# Verificar imagePullSecrets
kubectl get pod -l app.kubernetes.io/instance=deco-mcp-mesh -o yaml | grep imagePullSecrets

# Criar secret se necessário
kubectl create secret docker-registry regcred \
  --docker-server=ghcr.io \
  --docker-username=USERNAME \
  --docker-password=TOKEN

# Adicionar ao values.yaml
imagePullSecrets:
  - name: regcred
```

### HPA Não Funciona

```bash
# Verificar HPA
kubectl get hpa deco-mcp-mesh

# Ver métricas
kubectl top pods -l app.kubernetes.io/instance=deco-mcp-mesh

# Verificar se metrics-server está instalado
kubectl get deployment metrics-server -n kube-system
```

## 🔐 Segurança

### Secrets Management

**⚠️ Não commite secrets no Git!**

Opções recomendadas:


1. **External Secrets Operator**:
```yaml
secret:
  BETTER_AUTH_SECRET: ""  # Preenchido via ExternalSecret
```

2. **Valores via linha de comando**:
```bash
helm install deco-mcp-mesh . \
  --set secret.BETTER_AUTH_SECRET=$(cat secret.txt)
```

### Security Context

O chart já inclui:
- ✅ `runAsNonRoot: true`
- ✅ `allowPrivilegeEscalation: false`
- ✅ `capabilities.drop: ALL`
- ⚠️ `readOnlyRootFilesystem: false` (pode ser habilitado com volumes tmpfs)

## 📊 Monitoramento

### Labels para Seleção

Todos os recursos têm labels padronizados:

```bash
# Ver todos os recursos do release
kubectl get all -l app.kubernetes.io/instance=deco-mcp-mesh

# Ver logs
kubectl logs -l app.kubernetes.io/instance=deco-mcp-mesh

# Ver métricas
kubectl top pods -l app.kubernetes.io/instance=deco-mcp-mesh
```

### Health Checks

- **Liveness**: Mata e recria pods com problemas
- **Readiness**: Remove pods do Service quando não estão prontos

## 📚 Referências

- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Go Template Documentation](https://pkg.go.dev/text/template)

## 📄 Licença

Este chart é parte do projeto deco-mcp-mesh.
