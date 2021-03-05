# Deploy MATLAB Algorithm to Azure Function

---

## Introduction :

Azure Function is a serverless compute service that enables user to run event-triggered code without having to provision or manage infrastructure. Being as a trigger-based service, it runs a script or piece of code in response to a variety of events.

Azure Functions can be used to achieve decoupling, high throughput, reusability and shared. Being more reliable, it can also be used for the production environments.

However, as of March 2021, Azure Function is only supporting C#, Javascript, F#, Java, Powershell, Python, and TypeScript.They are mainly opensource language.

In this documentation, it will walk you through two ways to embed MATLAB Algorithm into Azure Function.
  1) through MATLAB Coder > C# (.dll)
  2) through MATLAB Compiler SDK > Python Package > Docker Image

Generally, solution(1) would allow your Azure function response faster than solution(2), as the C# is directly supported by Azure function natively. For solution (2), it requires more storage space for the docker image. Intuitively, solution (2) might be preferable due to MATLAB Runtime supports almost all MATLAB Function but MATLAB Coder is only supporting limited function for code generation. 

Function supported for code generation (MATLAB Coder):
[MathWorks Documentation](https://www.mathworks.com/help/coder/ug/functions-and-objects-supported-for-cc-code-generation.html)

Any inquiries, you may email me : kevin.chng@techsource-asia.com, I will try to help if my time is allowed.

---

## Solution 1 : MATLAB Coder

In this solution, we will generate standalone c/c++ source code from MATLAB algorithm and wrap it as dll.
By default, Azure function supports 32bits Platform, it might require more troubleshooting if we try to switch it to 64bits. Therefore, the generated MATLAB .dll has to be 32bits. This process could be done using MATLAB Coder and Visual Studio Compiler. Subsequently, we embed this .dll into our Azure Function dotnet framework and later deploy to Azure Cloud.

---

## Other possibilities
### Configure your local environment :
Before you begin, you must have the following:

1) [Azure Function Tool](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2)
2) Microsoft Visual Studio 9.0/10.0/11.0/12.0/14.0/15.0
3) MATLAB & MATLAB Coder

### References:
The links below are my references for this solution:

1) [Generate 32bits dll using MATLAB Coder](https://www.mathworks.com/help/coder/ug/build-32-bit-dll-on-64-bit-windows-platform-using-msvc-toolchain.html)
2) [Create C# function in Azure for Command Line (Azure Function)](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-csharp?tabs=azure-cli%2Cbrowser)

### Example :
Below is the detail steps to deploy Azure Function through MATLAB Coder:

[Example](https://github.com/KevinChngJY/azurefunction_matlab_deployment/blob/main/MATLAB_Coder_Azure_Function.md)

---

## Solution 2 : MATLAB Compiler SDK


