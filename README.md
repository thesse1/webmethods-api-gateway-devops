# About API Gateway DevOps

Please note: This fork implements the original webmethods-api-gateway-devops template for a specific demo scenario with specific requirements. Some of the files in this repository have been changed/extended/added to cover the requirements of the demo scenario. This readme has been extended to cover the specific demo scenario.

As each organization builds APIs using API Gateway for easy consumption and monetization, the continuous integration and delivery are integral part of the API Gateway solutions to meet the consumer demands. We need to automate the management of APIs and policies to speed up the deployment, introduce continuous integration concepts and place API artifacts under source code management. As new apps are deployed, the API definitions can change and those changes have to be propagated to other external products like API Portal. This requires the API owner to update the associated documentation and in most cases this process is a tedious manual exercise. In order to address this issue, it is a key to bring in DevOps style automation to the API life cycle management process in API Gateway. With this, enterprises can deliver continuous innovation with speed and agility, ensuring that new updates and capabilities are automatically, efficiently and securely delivered to their developers and partners in a timely fashion and without manual intervention. This enables a team of API Gateway policy developers to work in parallel developing APIs and policies to be deployed as a single API Gateway configuration.

![GitHub Logo](/images/api.png)

In addition to the functionality of the original webmethods-api-gateway-devops template, this specific scenario demonstrates how to automatically adjust the API Gateway assets for the deployment on different stages. It implements the following requirements: APIs should have separate sets of applications (with different identifiers) on different stages. The correct deployment of these applications should be enforced automatically. All applications are created on the Development environment with names ending with "_DEV", "_STAGE" or "_PROD" indicating their intended usage. All applications should be exported and managed in VCS, but only the _STAGE and _PROD applications should be imported on the respective STAGE and PROD environments. This is implemented by manipulating the assets on a dedicated build environment: Initially, all assets (including all applications) are imported on the build environment. Then all applications except for _STAGE or _PROD, respectively, are automatically deleted from the build environment using the API Gateteway Appication Management API. Finally, the API project is exported again from the build environment (now only including the right applications for the target environment) and imported on the target environment.

## webMethods API Gateway assets and configurations
The following API Gateway assets and configurations can be moved across API Gateway stages:
 - Gateway APIs 
 - Policy Definitions/Policy Templates/Global Policies
 - Applications
 - Aliases
 - Plans
 - Packages
 - Subscriptions
 - Users/Groups/ACLs/Teams
 - General Configurations like Load balancer, Extended settings
 - Security configurations
 - Destination configurations
 - External accounts configurations

## webMethods API Gateway deployment
webMethods API Gateway can be deployed with many flavors.
 - Standlone API Gateway with embedded elastic search.
 - Clustered API Gateway with embedded elastic search.
 - Standalone API Gateway with external elastic search and Kibana.
 - Clustered API Gateway with external elastic search and Kibana.
 
The docker compose files for these different deployment styles can be found at https://github.com/SoftwareAG/webmethods-api-gateway/tree/master/samples/docker/deploymentscripts
 
 ## Devops and CI/CD in webMethods API Gateway
 The CI/CD and devops flow can be achieved in multiple ways. 
 ### Using webmethods Deployer and Asset Build environment
  API Gateway asset binaries can be build using Asset Build environment and promoted across stages using WmDeployer. More information on this way of CI/CD and Devops automation can be found at http://techcommunity.softwareag.com/pwiki/-/wiki/Main/Staging%2C+Promotion+and+DevOps+of+API+Gateway+assets 
 ### Using Promotion management APIs
  The promotion APIs that are exposed by API Gateway can be used for the devops automation. More information on these APIs can be found at https://github.com/SoftwareAG/webmethods-api-gateway/blob/master/apigatewayservices/APIGatewayPromotionManagement.json
 ### Directly using the API Gateway Archive API for exporting and importing asset definitions
  This approach is followed in this scenario.

# About this repository

This repository provides assets/scripts for users to setup a CI/CD for API Gateway assets. Please note, the assets/scripts/methods provided in this repository explain one of the ways of setting up the CI/CD and not the only way. The samples provided are mainly to explain the capabilities and not be used as-is. One can clone this repository, then modify it to suite their organizational needs - for example, have different number of environments, different type of deployment (Kubernetes based) etc.

The samples in this repository use the API Gateway Archive API for automation of the Devops flow.

The repository has the following folders

  - apis : Contains the API Gateway assets of the organization.
  - apis_STAGE : Contains the API Gateway assets for deployment on the STAGE environment.
  - configuration : Contains staged specfic API Gateway Administration configurations.
  - deployment-descriptor : Contains staged specific docker compose files to create API Gateway environments.
  - tests : Contains the Postman Test suites and Environmental variables required for the test suites and the Postman utility collections.
  - utilities : Contains the Postman collections for import, promotion management (not used in the demo scenario) and preparation for STAGE environment.
  - bin : Library scripts that are used to create API Gateway environments, export, import and test assets and prepare them for the STAGE environment.
  - pipelines : Contains some sample Jenkins (not used in the demo scenario) and the Azure pipelines as an insight for organizations. 
  
# Devops CI/CD usecases

This github project contains the scripts that can help setup simple CI/CD flow for APIs using API Gateway.

## API Gateway environments
 
This sample project contains deployment descriptors under /deployment-descriptors (docker-compose files) to create different API Gateway environments (Dev, QA and Prod).

The deployment-descriptors are available to create following environments,
 - build enviroment, a single-node API Gateway. This is mostly for building APIs in a developer machine and assert them with the tests.
 - dev enviroment, a single-node API Gateway for development
 - qa and prod environements which are three node clustered API Gateways with a nginx load balancer.

More examples with different flavors of deployment can be found at  
https://github.com/SoftwareAG/webmethods-api-gateway/tree/master/samples/docker/deploymentscripts. 

## gateway-setup.sh
The gateway-setup.sh under /bin uses these deployment descriptors and create the environments.

| Parameter | Description |
| ------ | ------ |
| stage |  Possible value are build,dev,qa,prod. |
| apigateway_image | The DTR for API Gateway image. |
| terracotta_image | The DTR for Terracotta image in case of cluster environments |
| apigateway_server_port | API Gateway server port.Default is 5555 |
| apigateway_ui_port | API Gateway UI port.Default is 9072 |
| apigateway_es_port |  API Gateway Elastic search port.Default is 9240 |
| create_new | Create new API Gateway container even if an  existing container is running by killing it.Default is false | 
| import_configurations | Import configurations into the created container|
| cleanup | Cleanup the created images |

Sample usage to create a prod environment
```sh
 $ gateway-setup.sh --stage prod  --apigateway_image mycompany_apigateway_image:latest --terracotta_image mycompany_terracotta_image:latest --create_new true 
```
To clean up the created environment
```sh
 $ gateway_setup.sh --stage dev --cleanup
```

More examples of deployment-descriptors with different flavors of API Gateway deployment can be found at  https://github.com/SoftwareAG/webmethods-api-gateway/tree/master/samples/docker/deploymentscripts.
One can clone this repository and add their staging environments under /deployment-descriptors  and use them in the scripts.

# Develop APIs and test using API Gateway

The most common usecase for an API Developer is to develop APIs in their local dev environments and then export them to flat file representation such that they can be integrated to any VCS. Also Developers need to import their APIs from an VCS i.e flat file representation to their local dev environments for further updates.

The gateway_import_export_utils.sh under /bin can be used for this. Using this script the developers can import/export APIs from their local Dev API Gateway to their VCS local repo and vice versa.

## gateway_import_export_utils.sh

The gateway_import_export_utils.sh can be used for developers import and export APIs (projects) as a flat file representation of the VCS. The export_payload.json file in each project folder under /apis defines which API Gateway assets belong to this project. The assets will be imported/exported into/from their respective project folders under /apis.

| Parameter | README |
| ------ | ------ |
| import(or)export |  To import or export from the flat file. |
| api_name | The API project to import.|
| apigateway_url |  APIGateway url to import or export from.Default is http://localhost:5555.|
| apigateway_es_port | API Gateway Elastic search port.Default is 9240|
| username |  The APIGateway username.Default is Administrator.|
| password | The APIGateway password.Default is manage.|

Sample Usage for importing the API petstore that is present as flat file under apis/petstore into API Gateway server at http://localhost:5555 

```sh
 $ gateway_import_export_utils.sh  --import --api_name petstore --apigateway_url http://localhost:5555
```
Sample Usage for exporting the API petstore that is present in the  API Gateway server at http://localhost:5555  as flat file under apis/petstore

```sh
 $ gateway_import_export_utils.sh  --export --api_name petstore --apigateway_url http://localhost:5555
```

## gateway_build.sh 
The next common scenario for an API developer is to assert the changes made to the APIs do not break their customer scenarios. To enable this, a build script gateway_build.sh under /bin can be used.

This gateway_build.sh creates an single node Docker-based API Gateway environment using the deployment descriptor under /deployment-descriptors/build/ and will import all the APIs under the folder /apis and then run all the  tests suites under /tests against the build environment created.

| Parameter | Description |
| ------ | ------ |
| apigateway_image | The DTR for API Gateway image. |
| apigateway_server_port | API Gateway server port.Default is 5555 |
| apigateway_ui_port | API Gateway UI port.Default is 9072 |
| apigateway_es_port |  API Gateway Elastic search port.Default is 9240 |
| create_new | Create new API Gateway container even if an  existing container is running by killing it.Default is false | 
| test_suite |  The postman collection test_suite to run. Default will not run any test.To run all tests pass *|
| skip_import | To skip the import of APIs |

Sample usage to run all tests by creating a build environment
``` sh 
gateway_build.sh --apigateway_image mycompany_apigateway_image:latest --apigateway_server_port 5558 --apigateway_ui_port 9075  --apigateway_es_port 9243 --test_suite \*
```

Sample usage to run only a subset of tests
``` sh 
gateway_build.sh --apigateway_server_port 5558 --test_suite ../tests/test-suites/APITest.json
```

## gateway_prepare_STAGE.sh

This script also creates a Docker-based API Gateway build environment, imports all APIs and runs the configured tests just like gateway_build.sh. In addition to that, it manipulates the assets on the build environment (after running the tests) as defined in the Prepare_STAGE.json Postman collection in /utilities/prepare. In our demo scenario, the Postman collection will remove all applications with names not ending with "_STAGE". Then the script exports all APIs into the /apis_STAGE folder for all API projects configured there. From there, the assets can be imported into the STAGE environment using the gateway_import_export_utils_STAGE.sh.

| Parameter | Description |
| ------ | ------ |
| apigateway_image | The DTR for API Gateway image. |
| apigateway_server_port | API Gateway server port.Default is 5555 |
| apigateway_ui_port | API Gateway UI port.Default is 9072 |
| apigateway_es_port |  API Gateway Elastic search port.Default is 9240 |
| create_new | Create new API Gateway container even if an  existing container is running by killing it.Default is false | 
| test_suite |  The postman collection test_suite to run. Default will not run any test.To run all tests pass *|
| skip_import | To skip the import of APIs |

Sample usage to run all tests by creating a build environment
``` sh 
gateway_prepare_STAGE.sh --apigateway_image mycompany_apigateway_image:latest --apigateway_server_port 5558 --apigateway_ui_port 9075  --apigateway_es_port 9243 --test_suite \*
```

Sample usage to run only a subset of tests
``` sh 
gateway_prepare_STAGE.sh --apigateway_server_port 5558 --test_suite ../tests/test-suites/APITest.json
```

## gateway_import_export_utils_STAGE.sh

The gateway_import_export_utils_STAGE.sh can be used for developers import and export APIs (projects) as a flat file representation of the VCS. The export_payload.json file in each project folder under /apis_STAGE defines which API Gateway assets belong to this project. The assets will be imported/exported into/from their respective project folders under /apis_STAGE.

| Parameter | README |
| ------ | ------ |
| import(or)export |  To import or export from the flat file. |
| api_name | The API project to import.|
| apigateway_url |  APIGateway url to import or export from.Default is http://localhost:5555.|
| apigateway_es_port | API Gateway Elastic search port.Default is 9240|
| username |  The APIGateway username.Default is Administrator.|
| password | The APIGateway password.Default is manage.|

Sample Usage for importing the API petstore that is present as flat file under apis_STAGE/petstore into API Gateway server at http://localhost:5555 

```sh
 $ gateway_import_export_utils_STAGE.sh  --import --api_name petstore --apigateway_url http://localhost:5555
```
Sample Usage for exporting the API petstore that is present in the API Gateway server at http://localhost:5555 as flat file under apis_STAGE/petstore

```sh
 $ gateway_import_export_utils_STAGE.sh  --export --api_name petstore --apigateway_url http://localhost:5555
```

# Pipelines
The key to proper devops is continuous integration and continuous deployment. Organizations use standard tools such as Jenkins and Azure to design their integration and assuring continous delivery.

This repository contains a sample Jenkins and Azure pipline that can be used by an organization for continuous integration & deployment of their APIs from developing them to delivering them to their consumers. These pipelines depict how an API (project) that is present in VCS can be promoted to across different API Gateway environments. The Azure pipeline has adjusted in this repository for the specific demo scenario.

References
-  Jenkins Pipelines https://www.jenkins.io/doc/book/pipeline/
-  Azure pipelines https://azure.microsoft.com/en-in/services/devops/pipelines/

# Example
A sample CI/CD flow starting from a API Developer to propage the change to Prod depicted in the below image. This flow is implemented by the Jenkins pipeline and the original Azure pipeline in the original Software AG webmethods-api-gateway-devops template.

![GitHub Logo](/images/devopsFlow.png)

For the demo scenario, the Azure pipeline has been adjusted to cover the automatic removal of non-STAGE applications for the deployment on the STAGE environment. The promotion to PROD environment has been disabled in the demo scenario.

Let's consider this example
 - An API developer  wants to make a change to the petstore API. All of the apis of the organization are available in VCS whose local repo is present under the /apis folder. This flat file representation of the API should be converted and imported into the developer's local development API Gateway enviroment for changes to be made.
  For this the user uses the /bin/gateway_import_export_utils.sh to do this and import this API to the mydev.apigateway:5556.
  ```sh 
   /bin/gateway_import_export_utils.sh  --import --api_name petstore --apigateway_url http://mydev.apigateway:5556
  ```
  - The API Developer makes the necessary changes to the petstore API. 
  - The API developer needs to ensure that the change that was made does not cause regressions. For this, the user needs to run the set of function/regression tests over his change before the change gets propagated to the next stage. 
  To run the set of tests in the developer instance he can use the /bin/gateway_build.sh
  ```sh 
   /bin/gateway_build.sh --apigateway_image mycompany_apigateway_image:latest --apigateway_server_port 5558 --apigateway_ui_port 9075  --apigateway_es_port 9243 --test_suite \*
  ```
  This would create a docker instance of API Gateway and run all the tests.This sample repository contains a simple set of Regression tests for the petstore API.These are located under /tests folder.
  
  - Now this change made by the API developer has to be pushed back to the VCS system such that this propogates to the next stage.i.e Convert(export) the API in the development environment to the local repository /apis. This can be done by executing the following command
  ```sh 
   /bin/gateway_import_export_utils.sh  --export --api_name petstore --apigateway_url http://localhost:5556
  ```
> Note : In case the API developer needs to create an API from scratch that is not availabe already , then he skips the first step.
  
  - After this is done the changes from the Developers local repo are commited to the VCS. 
  
  -  Continuous integration and automation is usually achieved with the help of CI/CD tools like Jenkins/Azure pipelines.
One can configure webhooks over their VCS systems that can get triggered when an change is commited to the repository.
Please refer to https://plugins.jenkins.io/generic-webhook-trigger/ for configuring webhooks over Jenkins and 
https://docs.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops for Azure pipelines.

 - The sample piplines given in this repository are present under /pipelines that do the following.
   - Checkout from the VCS system all the apis.
   - Build and test them on QA environment.
   - Rollout to the higher stage that is done by executing the Promotion mangement APIs of API Gateway.
   
 - For the demo scenario, the Azure pipeline has been adjusted to perform the following tasks.
   - Checkout from the VCS system all the apis.
   - Build and test them on Build environment.
   - Prepare assets for deployment on STAGE environment (using the Prepare_STAGE.json Postman collection), i.e., remove all non-STAGE applications.
   - Export assets from Build environment and import on STAGE environment.
   - The Rollout step is deactivated in the demo scenario.
   
   These pipelines get executed as a result of the webhook that takes care of validating the APIs and on succesful test results doing a promotion of the APIs to higher stages.
   
   > Note : The jenkins pipline uses the jenkins.properties file that contains an variable 'api_project' to promote specific API alone to the next stage. In Azure this is achieved by an inline parameter 'apiProject'
   
   ## Environment based configurations
   One other most common scenario for a staged API Gateway environment is different configuration values for different stages. This repository contains this staged configurations under the folder '/configurations'. The samples configurations are
   - dev stage containing a logConfig that would set the logging level to debug with no load balancer.
   - qa stage containing a different set of Load balancing configurations.
   - prod stage containing a different set of Load balancing configurations.
   
   Importing this configuration to the staged API Gateway can be managed using the gateway-setup.sh and the parameter import_configurations=true. 
   
   Below section explains how the aliases can be used to dynamically inject configurations across stages. Besides Aliases, admin configurations can be maintained independently of the API projects under the /configuration folder. One can export the admin configuration and maintain them under the /configuration folder specific to environments and import them back when a new instance of the environment is created, as shown in this project. 
   
   One can also use externalized configurations (http://techcommunity.softwareag.com/pwiki/-/wiki/Main/Starting%20API%20Gateway%20using%20externalized%20configurations) to inject different configurations for different environments. Currently, not all the admin configurations are available through externalized configurations though.
   
   
   ## Variable substitutions
   With CI/CD, one of the common usecases is to make use of different values for configurations at different stages.  In this example, we have used stage specific aliases to demonstrate the use of different configurations for different environments. Aliases can be configured for different stages with different values and during promotion of the APIs, the respective stage specific alias values are used and promoted. In our petstore sample, we have use an routing Alias that can have different backend API endpoint values for different stages.  
