README.md

under the adminierTemplate.yaml we got the line 16 with this value "@@SuperSecretToKube@@", thats means this is a Variable and is going to be replaced with Replace Tokens extension

https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens

The Azure Pipeline consists on a really simple yaml 
```YAML
trigger: none

pool:
  vmImage: 'ubuntu-latest'

steps:
#Download the Secrets and the Secret name is going to be available as var name that means if you have an Azure Key Vault Secret called myPhoneNumber, the secret under the pipeline is going to be $(myPhoneNumber)
- task: AzureKeyVault@1
  inputs:
    azureSubscription: 'HERE YOU NEED TO GET AN AZURE RM SERVICE CONNECTION'
    KeyVaultName: 'NAME OF YOUR KEYVAULT'
    SecretsFilter: '*'

# This task matches the @@VARNAME@@ and replaces the value, this means your file need to have @@ and the valueyou are trying to replace
- task: replacetokens@3
  inputs:
    rootDirectory: '.'
    targetFiles: '**/adminierTemplate.yaml'
    encoding: 'auto'
    writeBOM: true
    actionOnMissing: 'warn'
    keepToken: false
    tokenPrefix: '@@'
    tokenSuffix: '@@'
#The tokenPrefix and tokenSuffix are suggested you can edit your file as you want, you an change that to any valeu as you desire 

# this task is to maintain your secret and is not shown under the Pipeline Logs
- script: |
    echo The Value $(SuperSecretToKube) is not going to be leaked 
  displayName: 'Value from the keyvault '

# this task is to maintain your secret and is not shown under the Pipeline Logs
- bash: |
   cd k8
   cat vipinExample.yaml

#This task is only to copy the file and publis as result to shown the value is changed
- task: CopyFiles@2
  inputs:
    Contents: '**/adminierTemplate.yaml'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

#This task is only to copy the file and publis as result to shown the value is changed
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```