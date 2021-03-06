# App Modernization and microservices using Service Fabric

## Overview 
---------------------------------------

This demo is focused on showcasing lift and shift scenarios with Full Framework apps and hosted them to Azure. Thanks to Service Fabric apps can be easily containerized and deployed to Azure. These apps can coexist with newer technologies such as Service Fabric .Net Core apps utilizing other paradigms such as Stateful Services and Reliable Collections. 
In the following demo we will examine how to lift and shift Full Framework apps to Azure and how to run / debug them locally using Visual Studio with the Service Fabric SDK and tools. 
The second part of the demo is intended to show how to deploy a Service Fabric stateful service using reliable collections which in turn, it is integrated with the rest of apps in Service Fabric.

## Application Overview

The application we are using in this sample is a hotel front-desk registration application. It's basic functionality is to check guest in and out.

<p align="center">
        <img src="images/image8.png"/>
    </p>

To modernize the application, it is followed a lift and shift approach to move it to Azure. To do so, we first need to containerize the application and later host it in Azure Service Fabric.

The application is the existing WebForms, WCF and Azure SQL Database pieces, as depicted below. This is a very traditional three-tire application, using Entity Framework to integrate with the data in the Azure SQL database, exposing it through a WCF service, which the WebForms application then interacts with.

   <p align="center">
        <img src="images/arch-overview-1.PNG"/>
    </p>

The final step of the modernization is to add a Service Fabric Stateful service to store registration information that can later be retrieved by the WebForms app to show different registration KPIs. Data is stored using SF Reliable Collections and distributed in different service partitions.

  <p align="center">
        <img src="images/arch-overview-2.png"/>
    </p>

## Key Takeaways
-------------------------------
The key takeaways of this demo are:
* Lift and shift Full Framework applications to Azure.
* Create Service Fabric Stateful Reliable Services that interact with Guest Full Framework apps.
* Debug apps and services locally with Service Fabric SDK.

## Demo Scenario
------------------------------
Deploy lift and shift full framework apps locally
* Illustrates how easy it is to deploy and debug apps in Service Fabric locally

Deploy lift and shift full framework apps to Azure
* Instructs the steps to deploy apps to Azure Service Fabric

Deploy stateful Reliable Service locally
* Presents an scenario where a stateful service is deployed along with the lift and shift full framework apps and how easy is to debug these services.

Deploy stateful Reliable Service to Azure
* Shows the steps to deploy the stateful Reliable Service scenario to Azure Service Fabric.

## Setup
-------------------------------------
### Local Setup
For the demo, a sandboxed Windows 10 virtual machine has been created in Azure with all the tools and SDKs preinstalled. Find RDP connection file at: **.\deploy\vm\SFDemo.rdp**
Optionally, in case of setting up another environment these are the requirements: 
*	Windows 10
*	Visual Studio 2017 Community Edition - Version 15.5 - https://www.visualstudio.com/ 
*	You need to have the Azure and .NET workload enabled
*	Docker CE - https://store.docker.com/editions/community/docker-ce-desktop-windows
*	Service Fabric SDK - 3.0 or never - https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started
*	Service Fabric Preview Tooling for 2017 - https://blogs.msdn.microsoft.com/azureservicefabric/2018/02/06/new-preview-tooling-for-visual-studio-2017/
Additionally, due to a Windows 10 limitation with dns and containers in VMs, UDP offload checksum in the VM needs to be disabled. Find Instructions at: **.\docs\DNS.in.VMs.pdf**

### Azure Setup
Execute the powershell script to create all the infrastructure necessary to deploy the applications to Service Fabric in Azure. The script is located at: **.\deploy\gen-sf-resources.ps1**
Replace the values and execute the following command in a Powershell console:
```
.\gen-sf-resources.ps1 -subscriptionId <subscriptionId> -resourceGroupName <resource group name> -vaultName <keyvault name> -vaultPwd <keyvault password> -clustername <cluster name> -clusterAdminUser <cluster admin name> -clusterAdminPwd <cluster admin password> -dbAdminUser <database username> -dbAdminPwd <database password> -location <resource location> -certPwd <certificate password>
```
Once the deployment is finished, the resources shown below should appear under your resource group in Azure Portal:

<p align="center">
    <img src="images/image1.png"/>
</p>

## Exercise 1: Deploy lift and shift full framework apps locally
-------------------------------------
1.	Open visual studio 2017 as Administrator
2.	Open the SmartHotel.Registration solution.

    <p align="center">
        <img src="images/image3.png"/>
    </p>
    > In the solution explorer 4 projects are shown. 
    The ApplicationModern  which contains all SF manifests.
    The Registration.Wcf contains a containerized WCF service.
    The Registration.Web contains a containerized Web Forms app.
    The Registration.StoreKPIs contains a .Net Core stateful Reliable Service for storing registration KPIs in Reliable Collections. 

3.	Open the Local.1Node and Local.5Node xml file and set the DefaultConnection parameter with the database UserId and Password  previously used in the setup chapter

    <p align="center">
        <img src="images/image4.png"/>
    </p>
    > These xml files configure the Service fabric settings when deploying to SF locally.

4.	Click Start button

    <p align="center">
        <img src="images/image5.png"/>
    </p>
    > With F5 experience, Visual Studio will generate the docker images for each service and deploy the app to the local SF cluster.

5.	Connect to Service Fabric Dashboard: http://localhost:19080/Explorer

    <p align="center">
        <img src="images/image6.png"/>
    </p>
    > Verify that the application and 3 services have been deployed. 
    Verify that the nodes, application and services are in healthy state in the Service Fabric dashboard (This might take a while).

6.	Open the Default.aspx.cs file in the Web.Registration project and set a breakpoint.

7.	Go to the web browser and enter the url: http://localhost:5000

    <p align="center">
        <img src="images/image7.png"/>
    </p>

    <p align="center">
        <img src="images/image8.png"/>
    </p>
    > Check that the Registration web app is shown in the browser and the breakpoint is hit in Visual Studio ready to debug.
    The Web app shows a list of customer registrations. If so, it means that all services are up and running and the Web App is able to communicate with WCF service in SF cluster.


## Exercise 2: Deploy lift and shift full framework apps to Azure

1.	Open the Cloud xml file and set the DefaultConnection parameter with the database UserId and Password  previously used in the setup chapter.

2.	Set the Registration_InstanceCount parameter to 3.

    <p align="center">
        <img src="images/image9.png"/>
    </p>
    > This xml file configure the Service fabric settings when deploying to SF in Azure.
    By setting up the InstanceCount parameters we are telling SF to scale the app to multiple instances during deployment.

3.	Right click on the ApplicationModern SF project and select Publish option.

    <p align="center">
        <img src="images/image10.png"/>
    </p>

4.	Select the connection endpoint pointing to the Service Fabric you previously created in the setup chapter.

5.	Select the Azure Container Registry you previously created in the setup chapter. 

    <p align="center">
        <img src="images/image11.png"/>
    </p>
    > When publishing, Visual Studio builds de docker images for each service and push them to the Container Registry. After that, it deploys the SF app to the cluster in Azure.

6.	Go to Azure Portal https://portal.azure.com

7.	Login and go to the resource group you used in the setup chapter.

8.	Click the Service Fabric Resource and select the Explorer link.

    <p align="center">
        <img src="images/image12.png"/>
    </p>

9.	When the browser warns about it is an insecure site. Continue and select the certificate when the browser asks to.

    <p align="center">
        <img src="images/image13.png"/>
    </p>

10.	Go to the dashboard, select the SmartHotel.Registration Service under Applications.

    <p align="center">
        <img src="images/image14.png"/>
    </p>
    > Verify that the application and 3 services have been deployed. Check that the nodes, application and services are in healthy state in the Service Fabric dashboard (This might take a while) and SmartHotel.Registration Service has 3 instances created


11.	Go to the browser and enter your cluster dnsname

12.	Refresh the browser multiple times

    <p align="center">
        <img src="images/image15.png"/>
    </p>
    > Check that the Service Instance Id is changing depending on the instance that is processing the request.

13.	Go to the dashboard, select the SmartHotel.Registration Service under Applications and click the right button.

14.	Select Scale Service option and set a -1 value

     <p align="center">
        <img src="images/image16.png"/>
    </p>
     > Change the number of instances from the dashboard. After a while, the service instances should be rescaled.


## Exercise 3: Deploy stateful Reliable Service locally

1.	Open visual studio 2017 as Administrator
2.	Open the SmartHotel.Registration solution.

    <p align="center">
        <img src="images/image17.png"/>
    </p>

    > In the solution explorer 4 projects are shown. 
    The ApplicationModern  which contains all SF manifests.
    The Registration.Wcf contains a containerized WCF service.
    The Registration.Web contains a containerized Web Forms app.
    The Registration.StoreKPIs contains a .Net Core stateful Reliable Service for storing registration KPIs in Reliable Collections. 

3.	Open the Local.1Node and Local.5Node xml file 
    > These xml files configure the Service fabric settings when deploying to SF locally.

4.	Set the **DefaultConnection** parameter with the database **UserId** and **Password**  previously used in the setup chapter.

5.	Set the **UseStoreKPIsStatefulService** parameter to **True**

6.	Click Start button
     <p align="center">
        <img src="images/image5.png"/>
    </p>
     > With F5 experience, Visual Studio will generate the docker images for each service and deploy the app to the local SF cluster.

10.	Connect to Service Fabric Dashboard: http://localhost:19080/Explorer
    <p align="center">
        <img src="images/image6.png"/>
    </p>
    > Verify that the application and 3 services have been deployed. Verify that the nodes, application and services are in healthy state in the Service Fabric dashboard (This might take a while).

11.	Go to the dashboard, select the SmartHotel.Registration.StoreKPIs Service under.

    <p align="center">
        <img src="images/image18.png"/>
    </p>
    > Verify that the stateful service and the 10 partitions are in Healthy state.

12.	Open the ValuesController.cs file in the Registration.StoreKPIs project and set a breakpoint to the Get/Id endpoint.

    <p align="center">
        <img src="images/image19.png"/>
    </p>

13.	Go to the web browser and enter the url: http://localhost:5000

14.	Click Display KPI button

    <p align="center">
        <img src="images/image20.png"/>
    </p>
    > The Display KPI button should appear. When clicking the button, it is redirected to the KPIs page where it is shown a chart with registration KPIs. 

    <p align="center">
        <img src="images/image21.png"/>
    </p>

15.	Select a user from the Dropdownlist

    <p align="center">
        <img src="images/image22.png"/>
    </p>
    > The breakpoint in the Registration.StoreKPIs service should be hit since the PKIs page retrieves the data from that stateful service. You can debug the service and check that the Reliable Collections where data is stored contains the registration KPIs for that specific user.

    <p align="center">
        <img src="images/image23.png"/>
    </p>


## Exercise 4: Deploy stateful Reliable Service to Azure

1.	Open the Cloud xml file and set the DefaultConnection parameter with the database UserId and Password  previously used in the setup chapter.

2.	Set the UseStoreKPIsStatefulService parameter to True

    <p align="center">
        <img src="images/image24.png"/>
    </p>
    > This xml file configure the Service fabric settings when deploying to SF in Azure.

3.	Right click on the ApplicationModern SF project and select Publish option.

    <p align="center">
        <img src="images/image10.png"/>
    </p>

4.	Select the connection endpoint pointing to the Service Fabric you previously created in the setup chapter.

5.	Select the Azure Container Registry you previously created in the setup chapter. 

    <p align="center">
        <img src="images/image11.png"/>
    </p>
    > When publishing, Visual Studio builds de docker images for each service and push them to the Container Registry. After that, it deploys the SF app to the cluster in Azure.

6.	Go to Azure Portal https://portal.azure.com

7.	Login and go to the resource group you used in the setup chapter.

8.	Click the Service Fabric Resource and select the Explorer link.

9.	When the browser warns about it is an insecure site. Continue and select the certificate when the browser asks to.

10.	Go to the dashboard, select the SmartHotel.Registration.StoreKPIs Service under Applications.

    <p align="center">
        <img src="images/image12.png"/>
    </p>
    > Verify that the application and 3 services have been deployed. Check that the nodes, application and services are in healthy state in the Service Fabric dashboard and SmartHotel.Registration Service has 3 instances created (This might take a while).

    <p align="center">
        <img src="images/image13.png"/>
    </p>

    <p align="center">
        <img src="images/image18.png"/>
    </p>
    > Verify that the Registration.StoreKPIs  stateful service and the 10 partitions are in Healthy state.

11.	Go to the browser and enter your cluster dnsname

12.	Click Display KPI button

    <p align="center">
        <img src="images/image25.png"/>
    </p>
    > The Display KPI button should appear. When clicking the button, it is redirected to the KPIs page where it is shown a chart with registration KPIs. 

    <p align="center">
        <img src="images/image21.png"/>
    </p>

13.	Click Back button to return to the main page.

14.	Click Register button

15.	Register a new reservation

    <p align="center">
        <img src="images/image26.png"/>
    </p>
    > Registers a new reservation and it is redirected to the main page.

16.	Click Display KPI button

    > The page shows the KPIs of all the customer registrations. This data is recollected from the stateful Register.StoreKPI service.

17.	Select the new created customer in the dropdownlist

    <p align="center">
        <img src="images/image27.png"/>
    </p>
    > The new customer should appear in the list and show its KPIs

## Summary
--------------------
Service Fabric allows us to easily lift and shift Full Framework applications and bring them to Azure providing all the benefits of the cloud such as reliability and scalability. Furthermore, Service Fabric tools helps us to deploy and debug the apps and services locally. 

## Feedback
-------------------------
Did you like the demo? Did you face any issues? Do you have ideas to include in the demo? Please share your inputs via email on [Azure App Dev Demos](mailto:azureappdevpstdemos@microsoft.com). We appreciate your feedback.
