# Cheat Sheet for Web app for Containers

Follow this cheat sheet for Azure CLI and Docker CLI to build , deploy , customize your app on Web app for Containers . To run Azure CLI and Docker CLI , use administrator mode . 

### BUILD APP 
Build your app locally before dockerizing the app. 

#### Dockerize the app

Build an image from the Dockerfile in the current directory and tag the image
	
	  docker build -t myapp:1.0 . 
List all images that are locally stored with the Docker engine

  	docker images

Delete an image from the local image store

	  docker rmi myapp:1.0


### DEPLOY IMAGE

#### Docker Hub 
Pull an image from a registry
	
     docker pull myapp:1.0
	
Retag a local image with a new image name and tag
	
     docker tag myapp:1.0 myrepo/myapp:1.0

Log in to a registry (Docker hub public repository)
    
    docker login 

OR Login to Azure Container registry (ACR)
	
	docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
	
Push an image to a registry
        
    docker push myrepo/myapp:1.0  

### CREATE WEB APP

Create an app service plan with the right Sku for your web app. For details , see [here](https://azure.microsoft.com/en-us/pricing/details/app-service/linux/)
  
         az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux

Create a web app. Give it a unique name

     az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan --deployment-container-image-name <your-docker-user-name>/myapp:0.1

Configure web app to use ACR image if using ACR 

    ```az webapp config container set``` command to assign the custom Docker image to the web app. Replace <app_name>, <docker-registry-server-url>, <registry-username>, and 	<password>. For Azure Container Registry, <docker-registry-server-url> is in the format https://<azure-container-registry-name>.azurecr.io.
 
       az webapp config container set --name <app_name> --resource-group myResourceGroup --docker-custom-image-name  myContainerRegistry.azurecr.io/starterapp --docker-registry-	server-url https://myContainerRegistry.azurecr.io --docker-registry-server-user <registry-username> --docker-registry-server-password <password>
 
Restart your app 
     
     	az webapp restart --resource-group myResourceGroup --name <your_app_name>

Browse your app

    https://<your_app_name>.azurewebsites.net 


#### Deploy Code with continous deployment 

##### Obtain a webhook

You can obtain the Webhook URL 
     
    az webapp deployment container show-cd-url -n <your_app_name> -g myResourceGroup

For the Webhook URL, you need to have the following endpoint: 
    
    https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook

You can obtain your publishingusername and publishingpwd by downloading the web app publish profile using the Azure portal.

##### Add a webhook 
**Docker Hub**

	Follow guidance here to add a webhook to Docker hub repo https://docs.docker.com/docker-hub/webhooks/
    
**ACR **
	
    Replace ```<webhook-url-web app>``` with web hook URL endpoint ```https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook```
 
 	   az acr webhook create --registry myContainerRegistry --name myacrwebhook01 --actions push --uri <webhook-url-web app>
 
When the image gets updated, the web app get updated automatically with the new image. Now push new changes to your docker image . 
  
### CUSTOM DOMAIN

Buy Domain from [Azure portal](https://docs.microsoft.com/en-us/azure/app-service/custom-dns-web-site-buydomains-web-app) or from any other provider. 

Map your domain to web app 

    az webapp config hostname add --webapp-name $webappname --resource-group myResourceGroup --hostname "www.mydomain.com"


### ADD SSL CERTIFICATE 

Get the Thumprint from your certificate 
	
	az webapp config ssl upload --certificate-file "<pfxPath-onlocal-machine>" --certificate-password "<pfxPassword>" --name $webappname --resource-group $resourceGroup --query 	thumbprint --output tsv

Bind the uploaded SSL certificate to the web app.
	
	az webapp config ssl bind --certificate-thumbprint "<thumbprint>" --ssl-type SNI --name $webappname --resource-group $resourceGroup

### RUN
Browse your app using ```http://www.mydomain.com```


