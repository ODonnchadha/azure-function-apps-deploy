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