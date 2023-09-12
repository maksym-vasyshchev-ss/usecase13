# Solution Architecture

## System decomposition

High level architecture is depicted on "components" page of `components.drawio` diagram.

Here, 
Web App - SPA, user interface for all system actors
API Gateway - single point of entry for the backend, component responsible for authorization, routing and security.
Identity Provider - component responsible for authentication, supplies users with JWT tokens, using OAuth2 authentication protocol.
Tax Calculation Service - responsible for calculating taxes. As input, takes filled in form and configured tax rules from Configuration Service.
Configuration Service - responsible for configuring additional questions and calculation rules. Uses database to store information.
User Management Service - responsible for configuring users and roles. Uses database to store information.
Tax Forms Service - responsible for forms creation, listing, editing and deletion. Uses database to store information.
Regulator Integration Service - reponsible for sending filled form to the regulator for review.
Database - relational database used by many services. 
Regulator API - external API to send filled forms to.

### Rationale
The decision to use SPA is dictated by necessity to provide good user experience along with responsive UI that is capable to work on mobile devices. This makes possible in future to create another dedicated application for mobile, should such need arise. `CON-1` `CON-2`

The decision to use API Gateway is made to satisfy security requirements and provide a point at which traffic could be controlled and managed easily. It decouples backend implementation, is responsible for the authentication validation and routing to the respective services. Potential improvements include using caching directly from gateway to improve performance and cut load on compute resources. `CON-4` `QA-1`

Identity provider is required to manage logins and registration securily. This satisfies requirement to protect information at all steps. `CON-4` `QA-1`

Tax Calculation Service satisfies the requirement to have one and can be scaled separately. `CON-5` `CON-6`

Configuration, User Management, Tax Forms and Regulator Integration services are separated along bounded contexts boundaries, which mean they are responsible for part of functionality and are being changed only in case this functionality changes, which isolates amount of change required in a system, improves maintainability and allows their simultaneous development and evolution.

Database is recommended to be relational, because this gives a good balance between: 
- maturity (everyone know who to work with SQL),
- simplicity (SDKs available for all languages), - functionality (atomicity, reliability, replication and automatic backups)
- cost (cheaper than most managed noSQL).
As an alternative, noSQL database could be considered.
Each service should be responsible for its own part of database (either own tables or schemas).
`QA-3` `QA-4` `CON-3`

## System infrastructure

Infrastructure is depicted on "infrastructure" page of `components.drawio` diagram.

This particular recommendation is build on AWS cloud platform, which provides very good functionality and is a leader among cloud providers.

### Rationale
Most of the services are managed, to cut costs on maintenance and operation.


Amazon Cognito is a managed identity provider, supporting OAuth2. It can be used to build login and sign up functionality.

Amazon Amplify is a framework for building web and mobile apps, that provides reach functionality out of the box. It's hosting capabilities allow to serve SPA resources from edge locations, using content delivery network - this makes application highly avialable.

Amazon API Gateway is a managed API Gateway.

Amazon EKS is recommendedd for compute payload (hosting services). It allows engineers to develop within familiar containerized paradigm, providing good separation between microservices. At the same time, it is a managed service, which cuts on operation and maintenance costs. EKS is capable of autoscaling which is important, since the exact load is not known in advance.
It is recommended to run 2 clusters in 2 availability zones to satisfy availability requirements. 
`CON-3` `QA-2` `QA-4`

NLB - Network Load Balancer - will be needed to balance load between availability zones.


For database, the recommendation is to use RDS - managed reliational database. PostgreSQL, MySQL or Aurora engines can be used, depending on implementation team preferences.
It is recommended to keep 2 instances, primary and secondary, in 2 availability zones to satisfy data reliability requirements, plus automatic backups.
`QA-3` `QA-4`

## Technology choices

For the SPA, any of the leading frontend frameworks can be used - React, Angular, Vue.

For the backend, it can be Go, Python, Node.js, .NET, Java - the choice is up to development team. However, it is not recommended to use different frameworks in different services, since the solution is not big enough to have multiple teams working on it and bringind multiple backend technologies will put a strain on engineering team.

## Deployment

It is recommended to establish CI/CD process for the deployment.

Most of managed components and infrastructure configuration (Cognito, API Gateway, EKS, RDS, networks) could be deployed using declarative infrastructure descriptions, such as Terraform.

Since AWS cloud has been chosen and no preferences for the CI/CD platform are identified, AWD CodePipeline, CodeCommit and CodeBuild could be used.

The process description to deploy infrastructure:
1. The dev team commits code to an AWS CodeCommit repository, which initiates AWS CodePipeline to start processing the code changes through the pipeline.
2. AWS CodeBuild installs and executes Terraform according to build specification.
3. Terraform stores the state files in S3 and a record of the deployment in DynamoDB.

The process description to deploy backend services:
1. The dev team commits code to an AWS CodeCommit repository, which initiates AWS CodePipeline to start processing the code changes through the pipeline.
2. AWS CodeBuild packages the code changes and dependencies and builds a Docker image.
3. The new Docker image is pushed to Amazon Elastic Container Registry (Amazon ECR).
4. CodeBuild uses a Kubectl command line tool to invoke Kubernetes API and update the image tag for the microservice deployment.
Kubernetes performs a rolling update of the pods in the application deployment according to the new docker image specified in Amazon ECR. `CON-3`

Since AWS Amplify is recommended to be used to host SPA, it is also recommended to be used as CI/CD for it:
1. The dev team commits code to an AWS CodeCommit repository.
2. Amplify Hosting detects changes, and  initiates Amplify pipeline.
2. Amplify pipeline builds and deploys code changes according to configuration.

## Behavior of the system for the main use cases

1. User accesses application domain. Domain is managed by AWS Amplify. SPA is returned to user's browser.
2. SPA loads and requires user to login (redirects to Cognito)
3. User logs in, get's back to SPA
4. SPA uses Tax Forms Service to load list of previously filled forms
5. User starts new tax form
6. SPA loads additional questions configuration from Configuration Service
6. After user enters data, SPA saves the data using Tax Forms Service
7. Once user reached point when tax calculation is possible, SPA invokes Tax Calculation Service to calculate tax.
8. After reaching the end of tax entry form, user reviews the tax and initiates it's sending to regulator. SPA uses Regulator Integration Service to send tax form.