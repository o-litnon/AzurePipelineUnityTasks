# Unity Tasks for Azure DevOps Pipelines

This collection of unity tasks adds CI/CD tooling for use in Azure Pipelines on Azure DevOps when working with [Unity](https://www.unity3d.com) projects.

# Get Started
Example  pipeline:
- ./azure-pipeline.yml 

Step 1: [Clone](https://learn.microsoft.com/en-us/azure/devops/repos/git/clone) this repo into your Azure project so you can reference it easily in your pipeline. You could optionally copy the files directly into your repository and skip Step 4.

Step 2: Install [UnityHub](https://docs.unity3d.com/2020.1/Documentation/Manual/GettingStartedInstallingHub.html) on your build agents

Step 3: Install [powershell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell) on your build agents

Step 4: Import this repository as a [resource](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources)

```
resources:
  repositories:
    - repository: templates
      type: git
      name: AzurePipelineUnityTasks
      ref: refs/heads/main
```

Step 5: Reference the remplates by using the name of the resource repository 
- template: Git/Task_Clean.yaml@templates

(or without the @templates postfix if the files exist locally)

```
variables:
  shouldDeploy: ${{eq(variables['Build.SourceBranch'], 'refs/heads/master')}}
  projectPath: path/to/project
  buildName: NameOfMyGame

stages:
- stage: "WindowsOS_Win"
  dependsOn: []
  pool:
    name: Default
    demands:
    - Agent.OS -equals Windows_NT
  jobs:
  - job: Job_Unity
    displayName: "Test, Build, Deploy"
    timeoutInMinutes: 180
    steps:
      - template: Git/Task_Clean.yaml@templates
      - template: Git/Task_Reset.yaml@templates
      - checkout: self
        clean: false
        lfs: true
        fetchDepth: 0
        persistCredentials: true
      - template: Unity/Tasks_SetupTestBuild.yaml@templates
        parameters:
          os: win
          projectPath: ${{variables.projectPath}}
          shouldTest: true
          shouldBuild: true
          buildTarget: Win64
          buildOutputPath: $(Build.ArtifactStagingDirectory)
          buildName: ${{variables.buildName}}
          modules:
            - windows-il2cpp
```