# ARM Templates

ARM templates are pretty complicated, here are just some thing I learned while studying.

## Child resources

Child resources are resources that only exist in the context of another resource. E.g, a virtualmachine extension is a child of a vm, a sql database is a child of a sql server. However, this are not 'dependent'. That is, if you need a child to be deployed after a parent, you need to add 'dependsOn' to the resource.

Example:

```json
"resources": [
  {
    "type": "Microsoft.Sql/servers",
    "apiVersion": "2022-05-01-preview",
    "name": "[parameters('serverName')]",
    "location": "[parameters('location')]",
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
    },
    "resources": [
      {
        "type": "databases",
        "apiVersion": "2022-05-01-preview",
        "name": "[parameters('sqlDBName')]",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
          },
        "dependsOn": [
          "[resourceId('Microsoft.Sql/servers', parameters('serverName'))]"
        ]
      }
    ]
  }
]
```

## Running ARM in Devops Pipeline

<https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/add-template-to-azure-pipelines>

There are three methods:

- Using the ARM template deployment task
  - easiest option, used when you have a template in your repo
- Using a powershell task
  - used when you want to use the same script that you are testing locally (you cannot locally test the ARM deployment task)
- copy/deploy task
  - this means its split into two tasks: copyfiles and deploy
  - The second 'deploy' task is very similar to the 'ARM deployment task' higher on this list. However, the difference is, the template is read from an accessible location (e.g. blob storage), rather than from git directly. The first 'copy' task simply copies the template to blob

## Deployment modes

ARM has two deployment modes. Both function similar to Terraform: create all resources, but if they already exist, update them where possible.

- Complete mode: Delete all resources in the resource group not specified in the template.
- Leave unchanged resources that are not specified in the template.

Note that for existing resources, properties are overwritten, even if they aren't specified, like they would be in terraform. The 'leave unchanged' only applies to *resources* not specified, not to properties.

