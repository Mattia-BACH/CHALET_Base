## Configuration Management for MS Windows Targets

We describe here a possible approach that we consider interesting. It assumes the use of the TwinCAT Package Manager available from the TwinCAT version 4026.


### Elements to be managed

In this approach, we recommend to validate and freeze the "output" of our developments, e.g. the compiled PLC code (not the corresponding sources, which sometimes have complex dependencies).

An overview of the several elements to be managed is given in the table below.

| Project Element | Depends on | Results in | Location (**) |
|---|---|---|---|
| PLC Module | PLC Sources | **PLC binary (*)** | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\Port_851.app` |
| | Beckhoff Libraries | | |
| | Customer Libraries | | |
| | Global type system | | |
| | PLC compiler version | | |
| Simulink target (C++) module | Matlab/Simulink sources incl. settings | **TcCOM binory (*)** | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Repository\TE140x Module Vendor\MySimulinkModuleName\MySimulinkModuleVersion\MySimulinkModuleName.tmx` |
| (as example) | Matlab version | | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Repository\TE140x Module Vendor\MySimulinkModuleName\MySimulinkModuleVersion\MySimulinkModuleName_ModuleInfo.xml` |
| | Toolboxes version | | |
| | Simulink version | | |
| | Matlab compiler version | | |
| | C++ build tools | | |
| Additional modules | Several dependencies | **TcCOM binory (*)** | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\...` |
| (general) | Dependency 1 | | |
| | Dependency 2 | | |
| | ... | | |
| System parameters | Persistent data file | | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\Port_851.bootdata` |
| | TwinCAT configuration files (JSON, XML, ...) | | `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\CurrentConfig.xml` |
| | Additional custom configuration files | | |
| Runtime configuration | **Current configuration** | **Target package registry + packages** | TODO |

(*) Per target plattform, (**) TwinCAT version 4026


### Management of runtime configuration with tcpkg

For this part, we recommend to validate and freeze also the "input" part, i.e. the list of required packages, versions and dependencies. Here is a quick guide to do that with the TwinCAT Package Manager `tcpkg`.

First, retrieve the list of required packages from your validated target:

```shell
tcpkg list --installed
```

For this demonstration, please consider the following (maybe not very meaningful) list of packages:

```output
TwinCAT.XAR.Base 3.4.8
TwinCAT.XAR.CxDevice 1.1.6
TwinCAT.XAR.DriversBase 3.21.3
TwinCAT.XAR.ESB 1.4.3
TwinCAT.XAR.EtherCATSlave 2.3.3
TwinCAT.XAR.EtherNetIP 3.2.2
TwinCAT.XAR.NCPTP 3.1.5033
TwinCAT.XAR.NdisDriver 2.1.8
TwinCAT.XAR.OpcUaClient 2.2.35
TwinCAT.XAR.OpcUaServer 5.2.123
TwinCAT.XAR.OpcUaSysTray 1.3.28
TwinCAT.XAR.PLC 2.1.12
TwinCAT.XAR.Realtime 2.2.2
```

Use this list to fill the template [`MyRuntimeConfig.nuspec`](MyRuntimeConfig.nuspec). In the section `<dependencies>` enter the required workloads for your use case:

```editor
<dependency id="TwinCAT.XAR.Base 3.4.8" />
<dependency id="TwinCAT.XAR.CxDevice 1.1.6" />
...
```

The rest of the template, e.g. tags `<id>`, `<version>`, `<title>`, etc., should also be adapted for your use case.

You can now use the prepared `.nuspec` definition to create your custom workload:

```shell
tcpkg pack .\MyRuntimeConfig.nuspec
```

This will create the custom NuGet package `Beckhoff-Switzerland.CHALET.RuntimeMgmtMSWin.1.0.0.nupkg`, meaning that the name of the workoad is `Beckhoff-Switzerland.CHALET.RuntimeMgmtMSWin` (important for later). Of course, the name will be different if you have adapted the template file for your use case.

The generated NuGet package now needs to be part of a feed. Define therefore your local feed, e.g. in the folder `C:\Users\Administrator\Desktop\RuntimeLocalFeed`. In the third command below, it is important to use the full path of the folder (do not enter a relative path):

```shell
mkdir C:\Users\Administrator\Desktop\RuntimeLocalFeed
mv Beckhoff-Switzerland.CHALET.RuntimeMgmtMSWin.1.0.0.nupkg C:\Users\Administrator\Desktop\RuntimeLocalFeed
tcpkg source add -n RuntimeLocalFeed -s C:\Users\Administrator\Desktop\RuntimeLocalFeed --enabled
```

To use your new local feed without a signature, temporarily disable signature verification (with administrator privileges):

```shell
tcpkg config unset -n VerifySignatures
```

You will now be able to download all packages (including dependencies!) required by your custom workload:

```shell
mkdir LocalPackages
tcpkg download Beckhoff-Switzerland.CHALET.RuntimeMgmtMSWin --zip --output ./LocalPackages
```

You finally have the zipped file `LocalPackages/download.tcpkgzip` containing all NuGet packages for your runtime environment. Safely store this file in a known and safe location and use it for the setup of other targets requiring the same runtime environment!


### Towards CI/CD with tcpkg

Configuration management is often related to CI/CD topics. The TwinCAT package manager `tcpkg` offers the possibility to execute power shell scripts. These can be included in the `MyRuntimeConfig.nuspec` file with the following syntax:

```editor
<files>
    <file src="MyDeployScripts/MyDeployScript_1.ps" target="MyTargetFolder" />
    <file src="MyDeployScripts/MyDeployScript2.ps" target="MyTargetFolder" />
    <file src="MyMiscScripts/**" target="MyTargetFolder" />
</files>
```

Possible applications:

* Setup an automatic copy and activation of your PLC project into the `C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\` folder
* Automatically activate/deactivate the MS Windows Unified Write Filter to "freeze" the part of the operating system not expecting changes during machine operation
* Basically anything you can think of...