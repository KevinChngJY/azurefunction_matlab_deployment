# Deploy MATLAB algorithm to Azure Function – MATLAB Compiler SDK (Use Window for development)
Use window as development environment and pusblish to Azure Function

---

## Introduction :

MATLAB is not supported natively with Azure Function – Serverless. Recently, Azure Function supports docker image, now we can create Azure Functions as a custom Docker container using a Linux base image. Docker containers open the door to hosting in a lot more environments than previously possible. This combo give us the flexibility to easily deploy and run our microservices either in the cloud or on-premises.

This customer image allows us to install additional dependency or configuration that isn't provided by the built in image. We insall MATLAB runtime (MCR) inside this custom docker container to provide the environment to run MATLAB code.

Although you are allowed to install additional dependency inside the custom docker, it is still restricted us to only use the following languages: C#,Java,Javascript,PowerShell,Python,TypeScript

For this solution, I use python as a bridge to linkup Azure Function and MCR. However, when you are selecting other languages, kindly make sure that your selected linux base image is 64bit which MCR requires 64bits environment.

Azure Function Image Overview:
https://hub.docker.com/_/microsoft-azure-functions-base

Azure Function image for python :
https://hub.docker.com/_/microsoft-azure-functions-python

Deploying function code in a custom Linux container requires Premium plan or a Dedicated (App Service) plan hosting. Therefore, completing this example might incurs costs of a few US dollars in your Azure account, which you can minimize by cleaning-up resources when you're done.


### Pre-requisites:
1) [Azure Function Tool](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2)
2) The Azure CLI version 2.4 or later.
3) Python 3.8 (64-bit), Python 3.7 (64-bit), Python 3.6 (64-bit), which are supported by Azure Functions.
4) Docker and Docker ID
5) MATLAB and MATLAB Compiler SDK (R2020a)

### Prerequisite check
In a terminal or command window, 

1) Run 

```
func --version 
```

to check that the Azure Functions Core Tools are version 2.7.1846 or later.

2) Run 

```
az --version 
```

to check that the Azure CLI version is 2.0.76 or later.

3) Run 

```
az login 
```

to sign in to Azure and verify an active subscription.

4) Run 

```
python --version (Linux/MacOS) or py --version (Windows) 
```

to check your Python version reports 3.8.x, 3.7.x or 3.6.x.

5) Run

```
docker login 
```

to sign in to Docker. This command fails if Docker isn't running, in which case start docker and retry the command.

### References:
1) [Generate Python Package and Build Python Application](https://www.mathworks.com/help/compiler_sdk/gs/create-a-python-application-with-matlab-code.htmll)
2) [Create a function on Linux using a custom container (Azure Function)]https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-linux-custom-image?tabs=bash%2Cportal&pivots=programming-language-python)

---

## Example :

### MARLAB

1. Build MATLAB function as follows:

![s1_01](img/s1_01.jpg)

2. Open libary compiler to generate python package:

![s2_02](img/s2_02.jpg)

![s2_03](img/s2_03.jpg)

3. Now you have the generated python package inside the "for_redistribution_files_only" folder :

![s2_04](img/s2_04.jpg)

Files inside the "for_redistribution_files_only" folder:

![s2_05](img/s2_05.jpg)

4. Open cmd, navigate to this foler to install this generated python package.

```
python setup.py install
```

You would get the outcome as follows:

![s2_06](img/s2_06.jpg)


### Create and test the local functions project
I use powershell for the following steps.

5. In a terminal or command prompt, run the following command for your chosen language to create a function app project in a folder named LocalFunctionsProject.

```
func init LocalFunctionsProject --worker-runtime python --docker
```

The --docker option generates a Dockerfile for the project, which defines a suitable custom container for use with Azure Functions and the selected runtime.

6. Navigate into the project folder:

```
cd LocalFunctionsProject
```

7. Add a function to your project by using the following command, where the --name argument is the unique name of your function and the --template argument specifies the function's trigger. 

```
func new --name HttpExample --template "HTTP trigger"
```

8. Edit HTTP triger to import our generated Python package :

Open HttpExample Folder, then open "init.py", edit it to as follows:

```
import logging
import azure.functions as func
import myadd

def main(req: func.HttpRequest) -> func.HttpResponse:
	logging.info('Python HTTP trigger function processed a request.')
	value1 = req.params.get('value1')
	value2 = req.params.get('value2')
	
	if not value1:
		try:
			req_body = req.get_json()
		except ValueError:
			pass
		else:
			value1= req_body.get('value1')
			value2= req_body.get('value2')
	
	if value1:
		myadd.initialize_runtime(['-nojvm', '-nodisplay'])
		m=myadd.initialize()
		final = m.myadd(float(value1),float(value2))
		return func.HttpResponse(f"Hello, value1+value2 is {final}")
	else:
		return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )
```

9. Edit HTTP triger to import our generated Python package :

Open HttpExample Folder, then open "function.json", edit it to as follows:

```
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Notes : Change authLevel from function to anonymous.

10. To test the function locally, start the local Azure Functions runtime host in the root of the project folder:

```
func start  
```

Once you see the HttpExample endpoint appear in the output, navigate to http://localhost:7071/api/HttpExample?value1=2%value2=3. The browser should display the summation of valiue1 and value2 that echoes back Functions.

### Build the container image and test locally

11. Copy MATLAB generated python package to the root of Azure Function projct folder.

Copy from :

![s1_13](img/s1_13.jpg)

![s1_14](img/s1_14.jpg)

Paste to :

![s1_15](img/s1_15.jpg)

12. Edit the docker files: (the dockerfile is in the root of project folder)


```
# To enable ssh & remote debugging on app service change the base image to the one below
# FROM mcr.microsoft.com/azure-functions/python:3.0-python3.7-appservice
FROM mcr.microsoft.com/azure-functions/python:3.0-python3.7

ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true

COPY requirements.txt /
RUN pip install -r /requirements.txt

COPY . /home/site/wwwroot

ENV DEBIAN_FRONTEND noninteractive
COPY setup.py /
ADD /myadd  /myadd
RUN python setup.py install

RUN apt-get -q update && \
    apt-get install -q -y --no-install-recommends \
      xorg \
      unzip \
      wget \
      curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

	
# Download the MCR from MathWorks site an install with -mode silent
RUN mkdir /mcr-install && \
    mkdir /opt/mcr && \
    cd /mcr-install && \
    wget --no-check-certificate -q https://ssd.mathworks.com/supportfiles/downloads/R2020a/Release/3/deployment_files/installer/complete/glnxa64/MATLAB_Runtime_R2020a_Update_3_glnxa64.zip && \
    unzip -q MATLAB_Runtime_R2020a_Update_3_glnxa64.zip && \
    rm -f MATLAB_Runtime_R2020a_Update_3_glnxa64.zip && \
    ./install -destinationFolder /opt/mcr -agreeToLicense yes -mode silent && \
    cd / && \
    rm -rf mcr-install

#Configure environment variables for MCR
ENV LD_LIBRARY_PATH /opt/mcr/v98/runtime/glnxa64:/opt/mcr/v98/bin/glnxa64:/opt/mcr/v98/sys/os/glnxa64:/opt/mcr/v98/extern/bin/glnxa64
ENV XAPPLRESDIR /etc/X11/app-defaults
```

If you are using different version of MATLAB,you might need to modify the docker file above so it install your specific version MATLAB runtime. You are welcome to approach me if you have question to modify the docker script, I try to help at my convenient.

13. In the root project folder, run the docker build command, and provide a name, azurefunctionsimage, and tag, v1.0.0. Replace <DOCKER_ID> with your Docker Hub account ID. This command builds the Docker image for the container.

```
docker build --tag <DOCKER_ID>/azurefunctionsimage:v1.0.0 .
```

Building this container for first time, it might take 1-2 hours (building subsequent changes is much faster) as we are downloding MCR and install it inside the docker container.

When the command completes, you can run the new container locally.

14. To test the build, run the image in a local container using the docker run command, replacing again <DOCKER_ID with your Docker ID and adding the ports argument, -p 8080:80:

```
docker run -p 8080:80 -it <docker_id>/azurefunctionsimage:v1.0.0
```

Once the image is running in a local container, open a browser to http://localhost:8080, which should display the placeholder image shown below. 

![s2_07](img/s2_07.jpg)

After you've verified the function app in the container, stop docker with Ctrl+C.

### Push the image to Docker Hub
Docker Hub is a container registry that hosts images and provides image and container services. To share your image, which includes deploying to Azure, you must push it to a registry.

15. If you haven't already signed in to Docker, do so with the docker login command, replacing <docker_id> with your Docker ID. This command prompts you for your username and password. A "Login Succeeded" message confirms that you're signed in.

```
docker login
```

16. After you've signed in, push the image to Docker Hub by using the docker push command, again replacing <docker_id> with your Docker ID.

```
docker push <docker_id>/azurefunctionsimage:v1.0.0
```

Depending on your network speed, pushing the image the first time might take a 2-4 hours (pushing subsequent changes is much faster). The image is around 7GB as it includes MCR. While you're waiting, you can proceed to the next section and create Azure resources in another terminal.

### Create supporting Azure resources for your function

17. Sign in to Azure with the az login command:

```
az login
```

18. reate a resource group with the az group create command. The following example creates a resource group named AzureFunctionsContainers-rg in the westeurope region. (You generally create your resource group and resources in a region near you, using an available region from the az account list-locations command.)

```
az group create --name AzureFunctionsContainers-rg --location westeurope
```

You can't host Linux and Windows apps in the same resource group. If you have an existing resource group named AzureFunctionsContainers-rg with a Windows function app or web app, you must use a different resource group.

19. Create a general-purpose storage account in your resource group and region by using the az storage account create command. In the following example, replace <storage_name> with a globally unique name appropriate to you. Names must contain three to 24 characters numbers and lowercase letters only. Standard_LRS specifies a typical general-purpose account.

```
az storage account create --name <storage_name> --location westeurope --resource-group AzureFunctionsContainers-rg --sku Standard_LRS
The storage account incurs only a few USD cents for this tutorial.
```

20. Use the command to create a Premium plan for Azure Functions named myPremiumPlan in the Elastic Premium 1 pricing tier (--sku EP1), in the West Europe region (-location westeurope, or use a suitable region near you), and in a Linux container (--is-linux).

```
az functionapp plan create --resource-group AzureFunctionsContainers-rg --name myPremiumPlan --location westeurope --number-of-workers 1 --sku EP1 --is-linux
```

We use the Premium plan here, which can scale as needed. To learn more about hosting, see Azure Functions hosting plans comparison. To calculate costs, see the Functions pricing page.

The command also provisions an associated Azure Application Insights instance in the same resource group, with which you can monitor your function app and view logs. For more information, see Monitor Azure Functions. The instance incurs no costs until you activate it.

### Create and configure a function app on Azure with the image
A function app on Azure manages the execution of your functions in your hosting plan. In this section, you use the Azure resources from the previous section to create a function app from an image on Docker Hub and configure it with a connection string to Azure Storage.

21. Create the Functions app using the az functionapp create command. In the following example, replace <storage_name> with the name you used in the previous section for the storage account. Also replace <app_name> with a globally unique name appropriate to you, and <docker_id> with your Docker ID.

```
az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime python --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0
```

The deployment-container-image-name parameter specifies the image to use for the function app. You can use the az functionapp config container show command to view information about the image used for deployment. You can also use the az functionapp config container set command to deploy from a different image.

22. Display the connection string for the storage account you created by using the az storage account show-connection-string command. Replace <storage-name> with the name of the storage account you created above:

```
az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv
```

23. Add this setting to the function app by using the az functionapp config appsettings set command. In the following command, replace <app_name> with the name of your function app, and replace <connection_string> with the connection string from the previous step (a long encoded string that begins with "DefaultEndpointProtocol="):

```
az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=<connection_string>
```

24. Get your function app url link and test it with your browser:

```
func azure functionapp list-functions <APP_NAME>
```

---
## Complete
---
