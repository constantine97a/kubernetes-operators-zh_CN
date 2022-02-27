# Running an Operator as a Deployment Inside a Cluster
在集群外运行 Operator，便于测试和调试，但生产 Operator 运行为 Kubernetes 部署。这种部署方式涉及一些额外的步骤：

1. 构建图像。 Operator SDK 的构建命令链到底层 Docker 守护进程以构建 Operator 镜像，并在运行时获取完整的镜像名称和版本：
   ```shell
    $ operator-sdk build jdob/visitors-operator:0.1
   ```
2. 配置部署。 使用镜像名称更新 SDK 生成的 deploy/operator.yaml 文件。 要更新的字段名为 image ，可以在以下位置找到：
   ```shell
   spec -> template -> spec -> containers
   ```
   生成的文件默认值为 REPLACE_IMAGE，您应该更新它以反映在上一个命令中构建的图像的名称。 构建后，将映像推送到外部可访问的存储库，例如 Quay.io (https://quay.io) 或 Docker Hub (https://hub.docker.com)。

3. 部署 CRD。 SDK 生成一个可以正常工作的骨架 CRD， 但有关充实此文件的更多信息，请参见附录 B：
    ```shell
   $ kubectl apply -f deploy/crds/*_crd.yaml
   ```
4. 部署服务帐户和角色。 SDK 生成 Operator 所需的服务帐号和角色。 更新这些以将角色的权限限制为操作员运行所需的最低权限。 适当地确定角色权限范围后，将资源部署到集群中：
   ```shell 
   $ kubectl apply -f deploy/service_account.yaml 
   $ kubectl apply -f deploy/role.yaml
   $ kubectl apply -f deploy/role_binding.yaml
   ```
   您必须按列出的顺序部署这些文件，因为角色绑定需要角色和服务帐户都存在。
5. 创建 Operator 部署。 最后一步是部署 Operator 本身。 您可以使用之前编辑的 operator.yaml 文件将 Operator 镜像部署到集群中：
    ```shell 
   $ kubectl apply -f deploy/operator.yaml 
   ```