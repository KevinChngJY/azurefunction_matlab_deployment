Deploy MATLAB algorithm to Azure Function – MATLAB Compiler SDK (Use Linux for development)
Use window as development environment and pusblish to Azure Function

Introduction :
MATLAB is not supported natively with Azure Function – Serverless. Recently, Azure Function supports docker image, now we can create Azure Functions as a custom Docker container using a Linux base image. Docker containers open the door to hosting in a lot more environments than previously possible. This combo give us the flexibility to easily deploy and run our microservices either in the cloud or on-premises.

This customer image allows us to install additional dependency or configuration that isn't provided by the built in image. We insall MATLAB runtime (MCR) inside this custom docker container to provide the environment to run MATLAB code.

Although you are allowed to install additional dependency inside the custom docker, it is still restricted us to only use the following languages: C#,Java,Javascript,PowerShell,Python,TypeScript

For this solution, I use python as a bridge to linkup Azure Function and MCR. However, when you are selecting other languages, kindly make sure that your selected linux base image is 64bit which MCR requires 64bits environment.

Azure Function Image Overview: https://hub.docker.com/_/microsoft-azure-functions-base

Azure Function image for python : https://hub.docker.com/_/microsoft-azure-functions-python

Deploying function code in a custom Linux container requires Premium plan or a Dedicated (App Service) plan hosting. Therefore, completing this example might incurs costs of a few US dollars in your Azure account, which you can minimize by cleaning-up resources when you're done.

Pre-requisites:
Azure Function Tool
The Azure CLI version 2.4 or later.
Python 3.8 (64-bit), Python 3.7 (64-bit), Python 3.6 (64-bit), which are supported by Azure Functions.
Docker and Docker ID
MATLAB and MATLAB Compiler SDK
Prerequisite check
In a terminal or command window,

Run
func --version 
to check that the Azure Functions Core Tools are version 2.7.1846 or later.

Run
az --version 
to check that the Azure CLI version is 2.0.76 or later.

Run
az login 
to sign in to Azure and verify an active subscription.

Run
python --version (Linux/MacOS) or py --version (Windows) 
to check your Python version reports 3.8.x, 3.7.x or 3.6.x.

Run
docker login 
to sign in to Docker. This command fails if Docker isn't running, in which case start docker and retry the command.

References:
Generate Python Package and Build Python Application
[Create a function on Linux using a custom container (Azure Function)]https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-linux-custom-image?tabs=bash%2Cportal&pivots=programming-language-python)
Example :
