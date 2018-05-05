# Try ASP.NET Core with Linux Container on your Windows machine

## Requirement

- Windows OS (Desktop or Server) with Hyper-V or Virtual Box
- Visual Studio or other tools for developing ASP.NET Core

This demo uses Windows 10 on Azure with Nested Hyper-V and Visual Studio. However, you can use your own tool.

## Set up

Follow the [document](https://developers.redhat.com/products/cdk/hello-world/) to set up Red Hat Container Development Kit.

If you haven't enabled Hyper-V on your Windows machine, enable Hyper-V at first. 

- Windows Server

```ps1
Install-WindowsFeature -Name Hyper-V -ComputerName <computer_name> -IncludeManagementTools -Restart  
```

- Windows 10 Professional or Enterprise

```ps1
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### Nested Hyper-V consideration

If you use Windpws with Nested Hyper-V, you should create an internal Network Switch instead of external Switch. You can follow this [Microsoft cocument](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization),

```ps1
New-VMSwitch -Name "InternalNATSwitch" -SwitchType Internal
```

In this case, HYPERV_VIRTUAL_SWITCH could be as follows.

```ps1
$env:HYPERV_VIRTUAL_SWITCH=“InternalNATSwitch”
```

After completed configuring Internal NAT, you can create a VM with the created Internal Network Switch. If you use Windows 10 with Nested Hyper-V, you can't enable DNS feature on Windows 10 as a Windows feature. In this case, you can [assign static private IP with an experimental feature](https://docs.openshift.org/latest/minishift/using/experimental-features.html#hyperv-static-ip). You can start minishift with the following command.

```ps1
$env:HYPERV_VIRTUAL_SWITCH=“InternalNATSwitch”
$env:MINISHIFT_USERNAME='<RED_HAT_USERNAME>'
$env:MINISHIFT_PASSWORD='<RED_HAT_PASSWORD>'
$env:MINISHIFT_ENABLE_EXPERIMENTAL="y"
cd C:\path\to\minishift
.\minisift.exe start `
  --network-ipaddress 192.168.0.10
  --network-gateway 192.168.0.1
  --network-nameserver 8.8.8.8
  --iso-url centos
```

## Create your app and deploy to OpenShift

OpenShift doesn't require any configuration and code for running on OpenShift. You can deploy a simple ASP.NET Core web project to OpenShift. This demo uses a project just created from ASP.NET Core MVC project in Visual Studio.
To deploy your ASP.NET Core app to OpenShift, please push your project to Git repository. This demo uses a public Git repository. A private repo is also available as far as it can reach from OpenShift and setting a password.

If your Red Hat CDK is 3.7 or lower, you have to create .NET Core templates in your CDK.

```ps1
oc login -u system:admin
oc create --namespace openshift -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/dotnet_imagestreams.json
oc create --namespace openshift -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/dotnet-example.json
oc create --namespace openshift -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/dotnet-runtime-example.json
oc create --namespace openshift -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/dotnet-pgsql-persistent.json
```

Then deploy to OpenShift with `.NET Core` template with the following parameters.

- Version: 2.0
- Name: your favorite name
- Git repository URL: Git URL for your application
- Git Reference: Git branch to deploy, master by default. You can specify the branch name, commit id.
- Build Environment Variable: Key=DOTNET_STARTUP_PROJCET, Value=<your ASP.NET Core project name>


## Push your code change

If you want to push your code change to your app on OpenShift, push the change to git and start build in OpenShift Web Console.
If your OpenShift can receive the push trigger from Git server, you can configure to start a new build automatically when a new change is pushed. Please see [this document](https://docs.openshift.com/container-platform/3.9/dev_guide/builds/triggering_builds.html#webhook-triggers).
