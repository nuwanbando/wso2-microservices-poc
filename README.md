# WSO2 Microservices POC

This repository contains a POC implemented with [WSO2 Microservices Framework for Java (MSF4J)](http://wso2.com/products/microservices-framework-for-java/), 
[Ballerinalang](https://ballerinalang.org) and [WSO2 API Manager](https://wso2.com/api-management/) on demonstrating 
how an enterprise solution can be developed according to [Microservices](https://martinfowler.com/articles/microservices.html) Architecture.

## Solution Architecture

The following diagram illustrates the solution architecture of this POC:
![](images/solution-architecture.png)

## Features

- Implementing microservices in WSO2 MSF4J microservices framework.
- The process of building Docker images for microservices.
- Securing microservices with JWT.
- Implementing integration services with [Ballerinalang](https://ballerinalang.org).
- The process of building Docker images for integration services.
- Wiring database connections, integrations between microservices and integration services.
- Exposing microservices and integration services via WSO2 API Manager.
- Deploying microservices, integration services and WSO2 API Manager on OpenShift.

## Getting Started

1. Clone this repository and switch to the latest tag:

   ````bash
   git clone https://github.com/imesh/wso2-microservices-poc.git
   ````

2. Download [Oracle JDK 1.8.144](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) Linux 64 bit distribution and copy it to the following folders:

   ````bash
   cp /path/to/jdk-8u144-linux-x64.tar.gz kubernetes-apim/base/apim/files/
   cp /path/to/jdk-8u144-linux-x64.tar.gz kubernetes-apim/base/analytics/files/
   ````

3. Download [MySQL Connector for Java 5.1.34](https://downloads.mysql.com/archives/c-j/) distribution and copy it's JAR file to the following folders:

   ````bash
   cp mysql-connector-java-5.1.34-bin.jar kubernetes-apim/base/apim/files/
   cp mysql-connector-java-5.1.34-bin.jar kubernetes-apim/base/analytics/files/
   ````

4. Download WSO2 API Manager 2.1.0 and WSO2 API Analytics 2.1.0 distributions via WUM and copy them to the following folders:

   ````bash
   cp wso2am-2.1.0.zip kubernetes-apim/base/apim/files/
   cp wso2am-analytics-2.1.0.zip kubernetes-apim/base/analytics/files/
   ````
5. Download [ballerinalang binary](https://ballerinalang.org/downloads/ballerina-runtime/ballerina-0.93.zip) and copy to the following folder in microservices directory

  ````bash
  cp ballerina-0.93.zip microservices/loan-applications-service/files/
  ````  
   
6. Install [Minishift](https://docs.openshift.org/latest/minishift/getting-started/index.html) or use an existing OpenShift cluster.

7. Login to OpenShift cluster using [OpenShift CLI](https://docs.openshift.org/latest/cli_reference/get_started_cli.html#installing-the-cli):

   ````bash
   oc login -u system:admin
   ````

8. If you are using Minishift execute the below command to configure your Docker CLI to point to the Minishift Docker daemon. This will allow the required Docker images to be built in the Minishift host itself:

   ````bash
   eval $(minishift docker-env)
   ````

9. Build MySQL Docker image:

   ````bash
   cd kubernetes-mysql/base/
   ./build.sh
   ````

10. Build microservices and their Docker images:

   ```bash
   cd microservices/
   ./build.sh
   ```
  
11. Build WSO2 API Manager and WSO2 API Analytics Docker images using the below command:

    ````bash
    cd kubernetes-apim/base/
    ./build.sh
    ````

12. If an existing OpenShift cluster is used copy the above Docker images into the OpenShift nodes or to a Docker registry:
   
    Copy the above Docker images over to the OpenShift Nodes. As an example use docker save command to create a tar file of the required image, scp the tar file to each node, and then use docker load command to load the image from the copied tar file on the nodes. Alternatively,if a private Docker registry is used, transfer the Docker images there and update the Docker image tags in the deployment files.

13. Create an user in OpenShift called admin and assign the cluster-admin role. This user will be used to deploy OpenShift resources:

    ````bash
    oc create user admin --full-name=admin
    oc adm policy add-cluster-role-to-user cluster-admin admin
    ````

14. Create a new project called wso2:

    ````bash
    oc new-project wso2 --description="WSO2" --display-name="wso2"
    ````
   
15. Create a service account called wso2svcacct in wso2 project and assign anyuid security context constraint:

    ````bash
    oc create serviceaccount wso2svcacct
    oc adm policy add-scc-to-user anyuid -z wso2svcacct -n wso2
    ````

16. Deploy the MySQL server, microservices, and WSO2 API Manager using the ```deploy.sh``` script found in the root folder:

    ````bash
    ./deploy.sh
    ````

17. Add /etc/hosts entries pointing to a OpenShift node IP address. For an example if OpenShift node IP is ```192.168.99.101```:

    ````bash
    192.168.99.101 wso2apim
    192.168.99.101 wso2apim-analytics
    192.168.99.101 wso2apim-gw
    ````

    The ```minishift ip``` command to can be used to find the IP address of Minishift VM.

18. Download and build [WSO2 API Manager CLI](https://github.com/imesh/wso2-apim-cli) using Golang:

    ````bash
    git clone https://github.com/imesh/wso2-apim-cli
    cd wso2-apim-cli
    go get github.com/urfave/cli
    go build .
    ````

19. Expose following environment variables:

    ````bash
    export DST_WSO2_APIM_ENDPOINT=https://wso2apim
    export DST_WSO2_APIM_GATEWAY_ENDPOINT=https://wso2apim-gw
    export DST_WSO2_APIM_USERNAME=admin
    export DST_WSO2_APIM_PASSWORD=admin
    ````

20. Copy ```Customers API``` and ```Loan Applications API``` zip files found in ```[wso2-microservices-poc]\apis\``` folder to the ```[wso2-apim-cli]\export\``` folder and execute the following command to import those to the WSO2 API Manager:

    ````bash
    cp [wso2-microservices-poc]/apis/*.zip [wso2-apim-cli]/export/
    cd [wso2-apim-cli]
    ./wso2-apim-cli import
    ````

    ````bash
    API CustomersAPI-v1.0.zip imported successfully
    API LoanApplicationsAPI-v1.0.zip imported successfully
    Client id and client secret obtained
    Access token generated: 3a5b71d6-98ea-3fef-82df-6ce5c6f20df9
    API CustomersAPI-v1.0 published successfully
    API LoanApplicationsAPI-v1.0 published successfully
    ````

21. Download and install [Postman](https://www.getpostman.com/) API client application.

22. Import Postman project found in ```[wso2-microservices-poc]/postman/``` folder into Postman.

23. Log into WSO2 API Manager using the URL [https://wso2apim/store](https://wso2apim/store) and credentials admin/admin.

24. Subscribe to both Customers API and Loan Applications API.

25. Navigate to "Applications" -> "Default Application" -> "Production Keys" and press the "Generate keys" button.

26. Copy the "Access Token" generated and update it in the Postman project under "Authorization" header in each request.

27. Invoke the "Create Customer" request via Postman.

28. Copy the "Customer ID" from the above response, add it to the body of the "Create Loan Application" request and invoke.

29. Now login to the OpenShift console and view the Loan Applications container log:

    ````bash
    HTTP GET /status/{referenceNumber} resource invoked: [referenceNumber] PERSONAL2017000002
    Invoking HTTP GET http://loans:8080/status/PERSONAL2017000002
    HTTP GET / resource invoked
    Invoking HTTP GET http://loans:8080/
    HTTP POST / resource invoked:
    {"type":"Personal","customerId":"1","amount":40000.0,"income":200000.0,"period":12}
    Find customer: HTTP GET http://customers:8080/1
    Customer response: {"id":1,"fname":"Shane","lname":"Smith","address":"First Street","state":"NY","postalCode":"12345","country":"United States"}
    Find credits of customer: HTTP GET http://credits:8080/1
    Customer credit response: {"totalCreditAmount":0.0}
    Total available credit amount: 80000.0
    Total credit amount: 0.0
    Available credit amount: 80000.0
    Create loan application: HTTP POST http://loans:8080/
    {"type":"Personal","customerId":"1","amount":40000.0,"income":200000.0,"period":12}
    Loan application created successfully: 
    {"referenceNumber":"PERSONAL2017000001"}
    Create customer credit: HTTP POST http://credits:8080/
    {"customerId":"1","referenceNumber":"PERSONAL2017000001","amount":40000.0}
    ````

## Removing Deployment

- Execute the ```undeploy.sh``` script found in the root folder for undeploying all OpenShift resources.

- If MiniShift is used, execute the below command to remove the temporary data folders used for persistence volumes:

  ````bash
  minishift ssh "sudo rm -rf /tmp/data/"
  ````