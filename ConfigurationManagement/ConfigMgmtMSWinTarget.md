## Configuration Management for Windows Targets

We describe here a possible approach.


### Elements to be managed

In the proposed approach, we recommend to validate and freeze the "output" of our developments, e.g. the compiled PLC code and not the corresponding sources, which sometimes have complex dependencies.

An overview of the several elements to be managed is given in the table below.

| Project Element | Depends on | Results in | Location (**) |
|---|---|---|---|
| PLC Project | PLC Sources | **PLC binary (*)** | C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\Port_851.app |
| | Beckhoff Libraries | | |
| | Customer Libraries | | |
| | Global type system | | |
| | PLC compiler version | | |
| Simulink target (C++) | Matlab/Simulink sources incl. settings | **TcCOM binory (*)** | TODO |
| (as example) | Matlab version | | |
| | Toolboxes version | | |
| | Simulink version | | |
| | Matlab compiler version | | |
| | C++ build tools | | |
| Additional projects | Several dependencies | **TcCOM binory (*)** | TODO |
| (general) | Dependency 1 | | |
| | Dependency 2 | | |
| | ... | | |
| System parameters | Persistent data file | | C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\Port_851.bootdata |
| | TwinCAT configuration files (JSON, XML, ...) | | C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\CurrentConfig.xml |
| | Additional custom configuration files | | |
| Runtime configuration | **Current configuration** | **Target package registry + packages** | TODO |

(*) Per target plattform, (**) TwinCAT version 4026


### Management of runtime configuration with tcpkg

For this part, we recommend to validate and freeze also the "input" part, i.e. the list of required packages, versions and dependencies.

Here is a quick guide to do that with the Beckhoff Package Manager `tcpkg`.

Retrieve the list of required packages from your validated target:

```shell
tcpkg list --installed
```

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

```shell
tcpkg pack .\MyConfig.nuspec
```

As administrator:

```shell
tcpkg config unset -n VerifySignatures
```

```shell
tcpkg download MyCustomWorkload --zip --output ./
```


### Towards CI/CD with tcpkg

Configuration management is often related to CI/CD topics. With `tcpkg` offers the possibility to execute power shell scripts. This can be defined in .nuspec -> include -> install script. Possible applications:

* Setup an automatic build and activation of the PLC project
* Automatically activate/deactivate the MS Windows Unified Write Filter to "freeze" the part of the operating system not expecting changes induced by the operation of the machine