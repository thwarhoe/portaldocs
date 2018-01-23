
<a name="checklist-for-onboarding-hosting-service"></a>
## Checklist for onboarding hosting service

The procedure for onboarding to the hosting service is as follows. Click on the arrow next to the checklist entry for an in-depth discussion of each step.

<details>

<summary>1. Update IsDevelopmentMode flag to false</summary>

The **Content Unbundler** tool requires setting the development mode to `false` to assign the correct build version to the zip file.

Update the **IsDevelopmentMode** flag in the `web.config` file to false, as in the following example.
```xml
    <add key="Microsoft.Portal.Extensions.<extensionName>.ApplicationConfiguration.IsDevelopmentMode" value="false"/>
```

The following example updates the **IsDevelopmentMode** flag for a monitoring extension.

```xml
    <add key="Microsoft.Portal.Extensions.MonitoringExtension.ApplicationConfiguration.IsDevelopmentMode" value="false"/>
```

If the **IsDevelopmentMode** flag setting should be reserved for release builds only, use a `web.Release.config` transform, as specified in [http://go.microsoft.com/fwlink/?LinkId=125889](http://go.microsoft.com/fwlink/?LinkId=125889).
</details>
<details>
<summary>2. Install ContentUnbundler and import targets</summary>

**Microsoft.Portal.Tools.ContentUnbundler** provides a **Content Unbundler** tool that can be run against the extension assemblies to extract static content and bundles. 

* If you installed the **Content Unbundler** tool by using **Visual Studio**, **NuGet Package Manager** or `NuGet.exe`, it will automatically add the following target.

  ```xml
  <Import Project="$(PkgMicrosoft_Portal_Tools_ContentUnbundler)\build\Microsoft.Portal.Tools.ContentUnbundler.targets" />
  ```

* If you are using **CoreXT** global `packages.config`, the previous target should be added to the `.csproj` file manually.

</details>

<details>
<summary>3. Verify the version number of the build</summary>

The zip file generated during the build should be named `<BUILD_VERSION>.zip`, where <BUILD_VERSION> is the current version number.

* CoreXT extensions

    Display the version number of the build.
    In this example, the computer displays the following build version.

    ```cs
    $>set CURRENT_BUILD_VERSION
    CURRENT_BUILD_VERSION=5.0.0.440
    ```

* Non-CoreXT extensions

    There are multiple build systems used by various teams. After you have determined which build version is used by your team, please send a pull request to help other extension developers. The pull request is located at [https://aka.ms/portalfx/pullrequest](https://aka.ms/portalfx/pullrequest).

    If the extension build does not have a version number, the `AssemblyInfo.cs` file in the **Visual Studio** project can be edited to set the build version to 1.0.0.0.  If the file does not exist, add it in the same folder as the one that contains the `<extensionName>.csproj` and `web.config` files. The process is as follows.

    1. Add new file **AssemblyInfo.cs**.

       ```xml <Compile Include="AssemblyInfo.cs" /> ```

    1. Update **AssemblyInfo.cs** content

     ```cs
    //-----------------------------------------------------------------------------
    // Copyright (c) Microsoft Corporation.  All rights reserved.
    //-----------------------------------------------------------------------------

       using Microsoft.Portal.Framework;

       [assembly: AllowEmbeddedContent("Microsoft.Portal.Extensions.<extensionName>")]
       [assembly: System.Reflection.AssemblyFileVersion("1.0.0.0")]
     ```

     **NOTE**: `Microsoft.Portal.Extensions.<extensionName>` specifies the fully qualified name of the extension. The build version is hard-coded to 1.0.0.0.
</details>
<details>
<summary>4. Provide environment-specific configuration files</summary>
  <!-- TODO:  If the production file can contain all 3 stamps, determine whether this example can  include all 3 names -->

Environment configuration files serve two purposes.

* They [override settings](#overriding-settings) for the target environment 
* They [load the extension](#loading-in-the-target-environment) in the target environment

<a name="checklist-for-onboarding-hosting-service-overriding-settings"></a>
### Overriding settings

The content of the configuration file is a json object with key/value pairs for settings to be overridden.  If there are no settings to override, the file should contain an empty json object. 

The settings for the portal framework are in the format of `Microsoft.Azure.<extensionName>.<settingName>`, where `settingName`, without the angle brackets, is the name of the setting. The framework will propagate the setting to the client in the format of `<settingName>`. For example, the `web.config` file that contains this setting would resemble the following.

```xml
<add key="Microsoft.Azure.<extensionName>.<settingName>" value="myValue" />
```

The equivalent configuration file would resemble the following.

```json
{
    "settingName": "myValue"      
}
```

<a name="checklist-for-onboarding-hosting-service-loading-in-the-target-environment"></a>
### Loading in the target environment

In order to load the extension in a specific environment, the  configuration file should be an embedded resource in the `Content\Config\*` directory of the **Visual Studio** project, as in the following example.

 ![alt-text](../media/portalfx-extensions-hosting-service/contentConfig.png  "Trace Event Parameters")

<!--TODO: Determine whether the phrase ' in the EmbeddedContentMetadata.txt file' needs to be includedn or whether it is too much detail. -->

If the file is not set as an `EmbeddedResource` in the EmbeddedContentMetadata.txt file, it will not be included in the output that gets generated by the **Content Unbundler** tool. 

 The files are named using the following convention:

`<host>.<domain>.json`

where

**host**: Optional. The environment that is associated with the domain. Values are  `ms` and `rc`. When this node is omitted, the dot that precedes the domain name is also omitted.

**domain**: Contains the value `portal.azure.com`.


The following are examples for each environment.

1. Dogfood 

   The configuration file is named  `df.onecloud.azure-test.net.json`, as in the following example.

    ```xml
    <EmbeddedResource Include="Content\Config\df.onecloud.azure-test.net.json" />
    ```

1. Production

    The production environment uses three stamps, as in the following table.

   | Environment | Stamp               |
   | ---         | ---                 |
   | RC          | rc.portal.azure.com |
   | MPAC        | ms.portal.azure.com |
   | PROD        | portal.azure.com    |

   One single configuration file contains all three stamps.  The configuration file is named  `*.portal.azure.com.json`, as in the following example.

    ```xml
    <EmbeddedResource Include="Content\Config\portal.azure.com.json" />
    <EmbeddedResource Include="Content\Config\rc.portal.azure.com.json" />
    <EmbeddedResource Include="Content\Config\ms.portal.azure.com.json" />
    ```

1. Mooncake 
    
    The configuration file is named  `portal.azure.cn.json`, as in the following example.

   ```xml
    <EmbeddedResource Include="Content\Config\portal.azure.cn.json" />
   ```

1. Blackforest

   The configuration file is named `portal.microsoftazure.de.json`, as in the following example.

    ```xml
    <EmbeddedResource Include="Content\Config\portal.microsoftazure.de.json" />
    ```


1. FairFax 

   The configuration file name is named `portal.azure.us.json`, as in the following example.

    ```xml
    <EmbeddedResource Include="Content\Config\portal.azure.us.json" />
    ```

</details>
<details>
<summary>5. Execute Content Unbundler to generate zip file</summary>

When the extension project is built, the **Content Unbundler** tool generates a folder and a zip file that are named the same as the extension version. The folder contains all content required to serve the extension.

You can override any of the default configuration parameters for the build environment.

* **ContentUnbundlerSourceDirectory**: This contains the name of the build output directory that contains the `web.config` file and the /bin directory. The default value is `$(OutputPath)`.

* **ContentUnbundlerOutputDirectory**: This contains the name of the output directory in which the **Content Unbundler** will place the unbundled content.  It will create a folder named `HostingSvc`. The default value is `$(OutputPath)`.

* **ContentUnbundlerRunAfterTargets**: This is used to sequence when the RunContentUnbundler target will run.  The value of this property is used to set the `AfterTargets` property of the RunContentUnbundler.  The default value is `AfterBuild`. 

* **ContentUnbundlerExtensionRoutePrefix**: This contains the prefix name of the extension that is supplied as part of onboarding to the extension host.

* **ContentUnbundlerZipOutput**: Zips the unbundled output that can be used for deployment. A value of `true` zips the output, whereas a value of `false`  does not create a .zip file. Defaults to `false`.   

The following examples are customized  configuration files.
* CoreXT extension configuration

    The following example is the customized configuration for a **CoreXT**  extension named "scheduler". 

```xml
  <PropertyGroup>
    <ContentUnbundlerSourceDirectory>$(WebProjectOutputDir.Trim('\'))</ContentUnbundlerSourceDirectory>
    <ContentUnbundlerOutputDirectory>$(BinariesBuildTypeArchDirectory)\HostingSvc</ContentUnbundlerOutputDirectory>
    <ContentUnbundlerExtensionRoutePrefix>scheduler</ContentUnbundlerExtensionRoutePrefix>
    <ContentUnbundlerZipOutput>true</ContentUnbundlerZipOutput>
  </PropertyGroup>
```

* Non-CoreXT extension configuration

    Outside of CoreXT, the default settings in the `targets` file should work for most cases. The only property that needs to be overridden is **ContentUnbundlerExtensionRoutePrefix**, as in the following example.

```xml
  <PropertyGroup>
    <ContentUnbundlerExtensionRoutePrefix>scheduler</ContentUnbundlerExtensionRoutePrefix>
  </PropertyGroup>
```

</details>
<details>
<summary>6. Upload safe deployment config </summary>

<!-- TODO:  Determine all of the contents of the storage account.  Also determine where else the zip file is discussed. -->
<!-- Then remove the following sentence: 
In addition to the zip files, the hosting service expects a config file to be located in the storage account. 
-->

Safe deployment practices require that extensions are rolled out to all data centers in a staged manner. The out-of-the-box hosting service provides the capability to deploy an extension in five stages, where each stage corresponds to one of five locations, or data centers.  The stages are as follows.

1. **stage1**: "centraluseuap"
1. **stage2**: "westcentralus"
1. **stage3**: "southcentralus"
1. **stage4**: "westus"
1. **stage5**: "*"

<!-- TODO:  Determine whether an extension can use the "stageDefinition" and "$sequence" parameters, or if they are reserved for hosting service use -->

When a user requests an extension in the Azure Portal, the portal will determine which version to load, based on the datacenter that is nearest to the user's geographical location.

 The config file specifies the versions of the extension that the hosting service will download, process and serve. It is authored by the developer as part of creating the extension.  The config file is located in the storage account, and its name is  `config.json`.  The file name is case sensitive, as are the names of the properties that it contains, as in the following example.

```json
{
    "$version": "3",
    "stage1": "1.0.0.5",
    "stage2": "1.0.0.4",
    "stage3": "1.0.0.3",
    "stage4": "1.0.0.2",
    "stage5": "1.0.0.1",
    "<friendlyName>": "2.0.0.0"
}
```

**$version**:  Required attribute. This is the version of the current `config.json` schema. The hosting service requires extension developers to use the latest version, which is 3.

**stage(1-5)**: Required attributes. A valid version number for the extension.  The version number is associated with the datacenter that has the same stage number.

* In this example, based on the preceding `config.json` file, if a user in the Central US region requests to load `Microsoft_Azure_<extensionName>`, then the hosting service will load the stage 1 version for the user, which is version 1.0.0.5. However, if a user in Singapore loads the extension, then the hosting service will load the stage 5 version, which is 1.0.0.1.

**friendlyName**: Optional. A unique name, without the angle brackets, that is assigned to a specific build version for sideloading. There is no limit to the number of friendly names can be provided by the developer for development and testing.

<!-- TODO: are friendly names separated by commas? -->
For more information about testing extensions with stages in configuration files, see [portalfx-extensions-hosting-service-scenarios.md#deploying-a-new-version-to-a-stage](portalfx-extensions-hosting-service-scenarios.md#deploying-a-new-version-to-a-stage).
</details>
<details>
<summary>7. Registering the extension with the hosting service</summary>

<!-- TODO:  determine how to make a public endpoint in order to copy the build files to it. -->

Extensions should publish the extracted deployment artifacts that are generated during the build to a public endpoint. This means copying the zip files for the build, along with the `config.json` file, to a directory that          .

<!-- TODO:  Verify whether this is the extension name or the extension directory.  -->
1. The developer should have access to two public endpoints, one for the Dogfood environment and one for the production environment.  An example of a Dogfood environment endpoint is `https://mybizaextensiondf.blob.core.windows.net/<extensionName>`.  An example of a PROD environment endpoint is `https://mybizaextensionprod.blob.core.windows.net/<extensionName>`.

    <!-- TODO:  determine what 'at the same level' means.-->

1. Make sure that all the zip files and the `config.json` file are at the same level previous to copying the extension to the endpoints.

1. After these files are available on a public endpoint, file a request to register the public endpoint.  The link is located at   [https://aka.ms/extension-hosting-service/onboarding](https://aka.ms/extension-hosting-service/onboarding). To onboard the extension, please provide following information in the request. 

    * **Extension Name**: The name of the extension, as specified in the  `extension.pdl` file.

    * **Dogfood storage account**:  The public read-only endpoint that serves zip files for the Dogfood environment.

    * **Prod storage account**: The public read-only endpoint that serves zip files for the production environment.

<!-- Determine whether this SLA should be the same as the table in portalfx-extensions-configuration-scenarios.md -->

The SLA for onboarding the extension is in the following table, expressed in business days.

| Environment | SLA     |
|-------------|---------|
| DOGFOOD     | 5 days  |
| MPAC        | 7 days  |
| PROD        | 12 days |
| BLACKFOREST | 15 days |
| FAIRFAX     | 15 days |
| MOONCAKE    | 15 days |

</details>
