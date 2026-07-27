---
trigger: always_on
description: <!-- markdownlint-disable MD022 MD032 -->
---


<!-- markdownlint-disable MD022 MD032 -->
# Deploying to remote servers with AppVeyor Deployment Agent
{:.no_toc}

AppVeyor Deployment Agent (Deployment Agent) is a service running on remote server and helping to deploy select artifact as IIS website or Windows application/service.

* Comment to trigger ToC generation
{:toc}
<!-- markdownlint-enable MD022 MD032 -->


## Software requirements

The following is required on the server to run Deployment Agent:

* Windows Server 2012 (Windows 8) or newer
* .NET Framework 4.5.2 or newer
* Web Role (IIS) is installed if you are deploying web site


## Installing AppVeyor Deployment Agent

1. Add new environment with **Agent** provider selected. Open environment settings and copy **Environment access key**.
2. Download AppVeyor Deployment Agent (.msi)
    * [**AppVeyor Deployment Agent - latest** (v6.3.3)](/downloads/deployment-agent/latest/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v6.3.3]({{ site.downloads_url }}/6.3.3/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v6.3.2]({{ site.downloads_url }}/6.3.2/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v6.1.0]({{ site.downloads_url }}/6.1.0/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v5.5.0]({{ site.downloads_url }}/5.5.0/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v5.3.0]({{ site.downloads_url }}/5.3.0/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v5.0.8]({{ site.downloads_url }}/5.0.8/AppveyorDeploymentAgent.msi)
    * [AppVeyor Deployment Agent v3.52.0]({{ site.downloads_url }}/3.52.0/AppveyorDeploymentAgent.msi)
3. Specify **Environment access key** during Deployment Agent installation.
4. Server is ready for deployment.


## Unattended Deployment Agent installation

Run the following in PowerShell console:

```powershell
(new-object net.webclient).DownloadFile('{{ site.url }}/downloads/deployment-agent/latest/AppveyorDeploymentAgent.msi', 'AppveyorDeploymentAgent.msi')
msiexec /i AppveyorDeploymentAgent.msi /quiet /qn /norestart /log install.log ENVIRONMENT_ACCESS_KEY=<your_access_key> DEPLOYMENT_GROUP=<your_deployment_group>
```

Replace `<your_access_key>` and `<your_deployment_group>` with your environment access key and your deployment group respectively.


## What artifacts can be deployed

Deployment Agent recognizes artifacts of two types which may contain either web application or Windows application/service:

* Zip archive
* Web Deploy package

To be deployable with Deployment Agent *artifact must have a name*.
Name should not have any spaces. All unnamed artifacts are skipped by Deployment Agent provider.


## How to get named artifacts

There are few possible ways of packaging artifact deployable by Agent:

1. When **Package Web Application projects** option is enabled on **Build** tab of project settings AppVeyor automatically publishes (applies web config transforms)
   and uploads VS.NET Web Application projects as artifacts named after the name of VS.NET project.
2. Specify **Deployment name** while adding artifact entry on **Artifacts** tab of project settings.
3. From script, for example **After build script**:

    ```text
    appveyor PushArtifact <zip_path> -DeploymentName MyApp
    ```

## Configuring deployment settings

Use **Provider settings** of Agent environment to configure *which* artifacts should be deployed by Agent and *how*. By default, nothing configured - nothing deployed.

Settings have format `<artifact_name>.<setting_name>` where `<artifact_name>` is artifact's **Deployment name**.

For example, let the build has the following artifacts:

![Artifacts](/assets/img/docs/agent-deploy-artifacts.png)

In order for Deployment Agent to deploy that artifact as IIS web site **Provider settings** will be:

![Artifacts](/assets/img/docs/agent-provider-settings.png)

## Global settings

There are few settings which have format `<setting_name>` instead of `<artifact_name>.<setting_name>`. Those settings are global and apply to each artifact deployment.

`agents_expected` - Number of remote agents expected to start deployment. Deployment marked as failed if number of agents started deployment is less than expected. Default is 1. Set to 0 to disable this check.

`agents_timeout` - Time in seconds to wait for remote agents to start deployment. Default is 10 seconds. Minimum is 5 seconds. Maximum is 60 seconds. Increase in case default value of 10 seconds appears to be not enough. Decrease if observe agents always pick up job much faster.

## Overriding settings while deploying from build

You can use environment variables for setting values on environment configuration, for example:

![agent-settings-with-variables](/assets/img/docs/deployment/agent/agent-settings-with-variables.png)

where `site_name` is environment variable. At the bottom of that screen we are defining its "default" value,
i.e. the value used when you deploy from Environments and build environment variables are not present.

However, when you deploy from a build you can override those environment variables like:

```yaml
deploy:
  - provider: Environment
    name: test-pc
    site_name: www.site-to-deploy.com
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appveyor/website](https://github.com/appveyor/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
