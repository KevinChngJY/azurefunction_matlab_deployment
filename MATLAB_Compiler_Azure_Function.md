# Deploy MATLAB algorithm to Azure Function – MATLAB Compiler SDK (Use Window for development)
Use window as development environment and pusblish to Azure Function

---

## Introduction :

MATLAB is not supported natively with Azure Function – Serverless. For workaround, we have to generate standalone c/c++ source code from MATLAB algorithm and wrap it as dll. 
By default, Azure function supports 32bits Platform, it might require more troubleshooting if we try to switch it to 64bits. Therefore, the generated MATLAB .dll has to be 32bits. This process could be done using MATLAB Coder and Visual Studio Compiler. Subsequently, we embed this .dll into our Azure Function dotnet framework and later deploy to Azure Cloud. However, MATLAB Coder does not support code generation for all MATLAB function, you may refer to the list below for supported function :
https://www.mathworks.com/help/coder/ug/functions-and-objects-supported-for-cc-code-generation.html

### Pre-requisites:
1) [Install Azure Function Tool](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2)
2)	Microsoft Visual Studio 9.0/10.0/11.0/12.0/14.0/15.0
3)	MATLAB & MATLAB Coder

### References:
1)	[Generate 32bits dll using MATLAB Coder](https://www.mathworks.com/help/coder/ug/build-32-bit-dll-on-64-bit-windows-platform-using-msvc-toolchain.html)
2)	[Create C# function in Azure for Command Line (Azure Function)](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-csharp?tabs=azure-cli%2Cbrowser)

---

## Example :
