# 自定义资源验证
添加新 API 时，Operator SDK 会生成骨架自定义资源定义。 该骨架可按原样使用； 无需进一步更改或添加即可创建自定义资源。

骨架 CRD 通过简单地将规范和状态部分定义为开放式对象来实现这种灵活性，分别表示用户输入和自定义资源状态：
```yaml 
spec:
    type: object
status:
    type: object
```
这种方法的缺点是 Kubernetes 无法验证这两个字段中的任何数据。由于 Kubernetes 不知道应该或不应该允许哪些值，只要清单解析，这些值是允许的。

为了解决这个问题，CRD 包括对 OpenAPI 规范 (https://oreil.ly/bzRIu) 的支持，以描述其每个字段的验证约束。您需要手动将此验证添加到 CRD 以描述规范和状态部分的允许值。

您将对 CRD 的规范部分进行两项主要更改：
• 添加属性映射。对于可能为此类型的自定义资源指定的每个属性，向此映射添加一个条目以及有关参数类型和允许值的信息。
• 或者，您可以添加一个必填字段，列出Kubernetes 应强制其存在的属性。在此列表中添加每个必需属性的名称作为条目。如果您在创建资源时省略了这些属性中的任何一个，Kubernetes 将拒绝该资源。

您还可以按照与规范相同的约定使用属性信息充实状态部分；但是，无需添加必填字段。

警告：在这两种情况下，现有的线类型：对象仍然存在；您将新添加的内容插入与此“类型”声明相同的级别。

您可以在 CRD 的以下部分中找到规范和状态字段：

```yaml
spec -> validation -> openAPIV3Schema -> properties
```

例如，visitorsApp CRD 的新增内容如下：
```yaml 
spec:
    type: object
    properties:
        size:
            type: integer
        title:
            type: string
    required:
        - size
status:
  type: object
  properties:
    backendImage:
        type: string
    frontendImage:
        type: string
```

此代码段只是您可以使用 OpenAPI 验证完成的一个示例。 您可以在 Kubernetes 文档 (https://oreil.ly/FfkJe) 中找到有关创建自定义资源定义的详细信息。