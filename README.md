## Deploying Azure Function Apps using Azure DevOps by Henry Been

- DEPLOYING AZURE FUNCTION APPS USING AZURE DEVOPS:
    - Start using CI/CD pipelines to automatically deploy your Azure Function apps. CI/CD stands for continuous integration and continuous deployment.
    - GitHub: Provides you with Git source control and GitHub Actions as the CI/CD pipeline.
    - Azure DevOps: Provides you with Git source control and uses Azure Pipelines as the CI/CD pipeline.
    - Within Pipelines, we normally differentiate between two parts:
        - The build stage, and this is where the CI part comes in, continues integration.
            - Here, we transform your sources into a deployable package.
        - The other stages are deployment stages. This is where you do continuous deployment.
            - This part of the pipeline is about deploying the package you built before into one or more environments.
        - And these two things combined often together with source control we call the deployment pipeline.
    - Demo: Building the Function App:
        - The build stage. A typical build stage will contain at least three parts:
            - The first one is about building the application. We take the source code and transform it into a deployable package.
            - Next, we will run all the tests that can be run without deploying the application. Often, these are the unit tests and sometimes some type of integration tests.
            - And finally, we will publish the package we created by building the application and store it in a centralized registry or along with the pipeline.
            - Demo.FunctionApp. In this FunctionApp, I have created two functions:
                - HelloWorld and HelloYou.
                ```javascript
                    {
                        "IsEncrypted":false,
                        "Values": {
                            "AzureWebJobsStorage":"UseDevelopmentStorage=true",
                            "FUNCTIONS_WORKER_RUNTIME":"dontnet-isolated",
                            "Server":"SuperFunctionsApp"
                        }
                    }
                ```
            - Demo.FunctionApp.Test project.
            - main.yaml. Demo.Deployment in a subfolder called pipeline.
                - YAML file that step by step describes what the building stage for my application should look like. This syntax is specific to Azure DevOps.
            - Stages for our pipeline:
                - Build. And in that stage, we're going to have a job, and in that job, we're going to have a number of tasks that together make up the job of building our application.
                    - The first task is about selecting the proper .NET SDK.
                    - Build the function app. I do that by using a task called .NET Core and then invoking the build command and referencing my FunctionApp project.
                    - Once my application is compiled, I can execute the tests that come along with it.
                    - I then create an archive, a ZIP file, out of all the files that together make up the FunctionApp.
                    - And at the bottom, I store the outcome of the build operation into what is called a pipeline artifact.
                        - What's important to understand is that we can take this file and create a pipeline out of it. 
                    - The final thing that we can now do is go back to the overview of our pipeline run and go to artifacts and see that one artifact was created while running this pipeline.
                        - The artifact is called function‑package and it contains a zip file. This is the outcome of the build pipeline. This is the compiled version of our function app.
            - Demo: Deploying the Function App:
                - Let's take the pipeline we have created in the previous clip and extend it to also deploy your code.
                    - In Azure DevOps we're going to review what is called a service connection.
                    - Next, we'll add a stage to our deployment pipeline.
                    - Then we'll add a stage to deploy into the production environment.
                        - And while doing so, we're going to extract a shared template.
                - The first thing I want to show you here is that I've created a new service connection. You can find this under your project settings and then go to Service connections. 
                    - The service connection exists and that it is the connection from your Azure DevOps environment to your target Azure environment.
                    - And we're going to use that service connection from the pipeline to deploy our resources.
                - Add a new stage to the deployment pipeline called test.
                    - The steps in this stage first download the function package.
                    - The next task you see is the deployment of the Azure function into the precreated infrastructure.
                - The pipeline FunctionsCICD successfully completes two stages.
                    - And the second one includes the deployment of the Function App.
                - A third run has now been added, and this run also deploys our application into production.

 - CREATING THE AZURE FUNCTION APP INFRASTRUCTURE:
    - Infrastructure vs. Application Deployment:
        - In the case of an Azure Function app, the application is the actual functions code. This runs on the infrastructure in Azure. And there can be multiple components, for example, a function app, a Service Bus, or Application Insights.
        - Now the application layer is often written in a programming language. That can be Python, TypeScript, but in our case, it is C#. 
            - Now the infrastructure you can create by going into the Azure portal and defining the infrastructure you need.
                - We call this ClickOps.
            - Another approach to creating the infrastructure is by describing that infrastructure in code‑like files and in deploying those code‑like files from your pipeline.
                - Those code‑like files we call Infrastructure as Code, IaC.
        -Both your Infrastructure as Code and your code can be deployed using a CI/CD pipeline. Now when you're going to deploy infrastructure using a CI/CD pipeline, you can do that in one of two ways.
            - You can reuse the pipeline you are using to deploy your code to also deploy your infrastructure.
            - But you can also choose to have a different pipeline that deploys the infrastructure, which is used from a second pipeline to deploy your code.
                - If one team owns both infrastructure and code, it is likely that they're going to end up with a pipeline which deploys both together.
                -   If different teams own the infrastructure and the code that deploys to that infrastructure, it is likely they're going to end up with two pipelines.
    - Demo: Coding an Azure Function App Plan as Code Using Bicep:
        - To deploy the infrastructure for a function app from a pipeline, we first have to define it using an Infrastructure as Code language like Bicep.
            - I'm going to declare a number of Azure resources. I do this using the resource keyword.
                - Storage account. This is the resource of type Microsoft.Storage/storageAccounts.
                - functionAppPlan. Under the hood is called a Microsoft.Web/serverfarms.
                - Final resource that I'm adding is of type Microsoft.Web/sites/config. And if you look closely, you can see that this resource is nested in the functionApp resource. It is a so‑called subresource or child resource.
                    - This resource specifies the key value properties that should be passed as the application configuration when the functionApp starts running.
            - The final construct that I'm going to use is what is called an output. I'm outputting the AppServiceName, which is going to be of type string as the name of the functionApp that I declared above. This means that I can pick up this value and use it later on in another task in my pipeline.
    - Demo: Deploying Bicep Files from Azure DevOps:
        - Create the pipeline infrastructure that our application runs on.
            - I am now also publishing the bicep file to an artifact, and I do this so that we can use it in the deployment stage later on. 
            - I have removed the parameter for our app service name. This name will no longer need to be provided as we will get it after creating the infrastructure. We can see this when we open the file deployment.yaml.
            - I now download two artifacts. 
                - The first one, the function‑package.
                - The second one, the bicep‑package. 
                - After downloading the packages, the first task I now execute is the deployment of the Bicep template.
                    - This task takes the bicep file and puts it into Azure.
                        - It is basically a wrapper around the Azure CLI task.
            - We have a function app written in C# with two functions.
            - We have a Bicep file that defines the infrastructure and the configuration of our application.
            - And we have a pipeline that brings all of this together, packages it, and deploys it into Azure.