# Role-Based Access Control (RBAC)

当 Operator SDK 生成 Operator 项目时（无论是 Helm、Ansible 还是基于 Go 的 Operator），它都会创建许多用于部署 Operator 的清单文件。
其中许多文件授予已部署 Operator 执行其在其整个生命周期中执行的各种任务的权限。

Operator SDK 生成三个与 Operator 权限相关的文件：

**deploy/service_account.yaml**

Kubernetes 没有以用户身份进行身份验证，而是以服务帐户的形式提供了一种程序化的身份验证方法。在向 Kubernetes API 发出请求时，服务帐户充当 Operator pod 的身份。
此文件仅定义服务帐户本身，您无需手动编辑它。有关服务帐户的更多信息，请参阅 Kubernetes 文档 (https://oreil.ly/8oXS-)。

**deploy/role.yaml**

此文件为服务帐户创建和配置角色。 角色规定了服务帐户在与集群 API 交互时拥有的权限。Operator SDK 生成此文件的权限非常广泛，出于安全原因，您需要在将 Operator 部署到生产环境之前对其进行编辑。
在下一节中，我们将详细解释如何优化此文件中的默认权限。

**deploy/role_binding.yaml**

此文件创建一个角色绑定，它将服务帐户映射到角色。 您无需对生成的文件进行任何更改。


## Fine-Tuning the Role
在最基本的层面上，角色将资源类型映射到用户或服务帐户可能对这些类型的资源采取的操作（在角色资源术语中称为“动词”）。例如，以下角色授予部署的查看（但不是创建或删除）权限：
```yaml
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
```
由于 Operator SDK 不知道您的 Operator 需要与集群交互的程度，因此默认角色允许对各种 Kubernetes 资源类型执行所有操作。以下片段取自 SDK 生成的 Operator 项目，说明了这一点。 * 通配符允许对给定资源的所有操作：

```yaml
  ...
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  - configmaps
  - secrets
  verbs:
  - '*'
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  verbs:
  - '*'
  ...
```

毫不奇怪，将这种开放和广泛的权限授予服务帐户被认为是一种不好的做法。您应该进行的具体更改取决于您的 Operator 的范围和行为。一般来说，您应该尽可能限制访问，同时仍然允许您的 Operator 正常工作。

例如，以下角色片段提供了访客站点Operator所需的最小功能：
```yaml
...
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - secrets
  verbs:
  - create
  - list
  - get
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
  - get
  - update
  ...
```

配置 Kubernetes 角色的完整细节超出了本书的范围。 你可以在 Kubernetes RBAC 文档中找到更多信息 (https://oreil.ly/osBC3)。