# Deploy MATLAB Algorithm to Azure Function

---

## Introduction :

Azure Function is a serverless compute service that enables user to run event-triggered code without having to provision or manage infrastructure. Being as a trigger-based service, it runs a script or piece of code in response to a variety of events.

Azure Functions can be used to achieve decoupling, high throughput, reusability and shared. Being more reliable, it can also be used for the production environments.

However, as of March 2021, Azure Function is only supporting C#, Javascript, F#, Java, Powershell, Python, and TypeScript.They are mainly opensource language.

In this documentation, it will walk you through two ways to embed MATLAB Algorithm into Azure Function.
  1) through MATLAB Coder > C# (.dll)
  2) through MATLAB Compiler SDK > Python Package > Docker Image

Generally, solution(1) would allow your Azure function response faster than solution(2), as the C# is directly supported by Azure function natively. For solution (2), it requires more storage space for the docker image. Intutively, solution (2) might be preferable due to MATLAB Runtime supports almost all MATLAB Function but MATLAB Coder is only supporting limited function for code generation. 

Function supported for code generation (MATLAB Coder):
[MathWorks Documentation](https://www.mathworks.com/help/coder/ug/functions-and-objects-supported-for-cc-code-generation.html)

## Configure your local environment :

