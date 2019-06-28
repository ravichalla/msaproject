## Prepare Environment

If you don't have an OpenShift instance running, we strongly suggest that you use minishift

### (Option 1) Use minishift (OpenShift 3.11)

Download Minishift from link:https://github.com/minishift/minishift/releases[minishift].
Execute:

---

$ minishift profile set msa-tutorial
$ minishift config set memory 8GB
$ minishift config set cpus 3
$ minishift config set image-caching true
$ minishift config set openshift-version v3.11.0
$ minishift addons enable anyuid
$ minishift addons enable admin-user
$ minishift start
$ eval $(minishift oc-env)

---

#### Access Openshift console

Open Openshift console: https://<openshift_ip>:8443/console/ +
(Accept the certificate and proceed)

Log in Openshift +
Use **developer/developer** as your credentials

## Install Microservices

### Option 1- Install all microservices using Ansible

Make sure to have Git, Ansible 2.4, Maven 3.1, JDK 1.8, NPM

Run commands:
git clone https://github.com/ravichalla/msaproject
cd helloworld-msa/ansible

First, edit the vars.yaml file to define the 'workdir' variable. And set the absolute path where you want to "git checkout" the source code of the microservices.
Then execute:

ansible-playbook helloworld-msa.yml

It might take 20min based on the local machine . You can access the urls , if successful ansible installation of components happened.

### option-2 Install each microservice individually

Create a project
In Openshift, click in the "New Project" button. Give it the name "helloworld-msa"

Important
You must use the exact project name since it will affect the generate endpoint’s URLs
Or run:

$ oc login <Openshift IP>:8443 -u developer -p developer
$ oc new-project helloworld-msa

###Deploy aloha (Vert.x) microservice
(Option 1) Deploy project via oc CLI
Basic project creation
$ git clone https://github.com/ravichalla/aloha
$ cd aloha/
$ oc new-build --binary --name=aloha -l app=aloha
$ mvn package; oc start-build aloha --from-dir=. --follow
$ oc new-app aloha -l app=aloha,hystrix.enabled=true
$ oc expose service aloha
Enable Jolokia and Readiness probe
$ oc set env dc/aloha AB_ENABLED=jolokia; oc patch dc/aloha -p '{"spec":{"template":{"spec":{"containers":[{"name":"aloha","ports":[{"containerPort": 8778,"name":"jolokia"}]}]}}}}'
$ oc set probe dc/aloha --readiness --get-url=http://:8080/api/health
(Option 2) Deploy project via Fabric8 Maven Plugin
\$ mvn package fabric8:deploy
Test the service endpoint
curl http://aloha-helloworld-msa.`minishift ip`.nip.io/api/aloha

###Deploy ola (Spring Boot) microservice
(Option 1) Deploy project via oc CLI
Basic project creation
$ git clone https://github.com/ravichalla/ola
$ cd ola/
$ oc new-build --binary --name=ola -l app=ola
$ mvn package; oc start-build ola --from-dir=. --follow
$ oc new-app ola -l app=ola,hystrix.enabled=true
$ oc expose service ola
Enable Jolokia and Readiness probe
$ oc set env dc/ola AB_ENABLED=jolokia; oc patch dc/ola -p '{"spec":{"template":{"spec":{"containers":[{"name":"ola","ports":[{"containerPort": 8778,"name":"jolokia"}]}]}}}}'
$ oc set probe dc/ola --readiness --get-url=http://:8080/api/health
(Option 2) Deploy project via Fabric8 Maven Plugin
\$ mvn package fabric8:deploy
Test the service endpoint
curl http://ola-helloworld-msa.`minishift ip`.nip.io/api/ola

###Deploy bonjour (NodeJS) microservice
Choose one of the following options/approaches to deploy this microservice.

Deploy project via oc CLI
Basic project creation
$ git clone https://github.com/ravichalla/bonjour
$ cd bonjour/
$ oc new-build --binary --name=bonjour -l app=bonjour
$ npm install; oc start-build bonjour --from-dir=. --follow
$ oc new-app bonjour -l app=bonjour
$ oc expose service bonjour
Enable Readiness probe
\$ oc set probe dc/bonjour --readiness --get-url=http://:8080/api/health
Test the service endpoint
curl http://bonjour-helloworld-msa.`minishift ip`.nip.io/api/bonjour

###Deploy api-gateway (Spring Boot)
The API-Gateway is a microservices architectural pattern. For more information about this pattern, visit: http://microservices.io/patterns/apigateway.html

(Option 1) Deploy project via oc CLI
Basic project creation
$ git clone https://github.com/ravichalla/api-gateway
$ cd api-gateway/
$ oc new-build --binary --name=api-gateway -l app=api-gateway
$ mvn package; oc start-build api-gateway --from-dir=. --follow
$ oc new-app api-gateway -l app=api-gateway,hystrix.enabled=true
$ oc expose service api-gateway
Enable Jolokia and Readiness probe
$ oc set env dc/api-gateway AB_ENABLED=jolokia; oc patch dc/api-gateway -p '{"spec":{"template":{"spec":{"containers":[{"name":"api-gateway","ports":[{"containerPort": 8778,"name":"jolokia"}]}]}}}}'
$ oc set probe dc/api-gateway --readiness --get-url=http://:8080/health
(Option 2) Deploy project via Fabric8 Maven Plugin
\$ mvn package fabric8:deploy
Test the endpoint
curl http://api-gateway-helloworld-msa.`minishift ip`.nip.io/api/gateway

###Deploy frontend (NodeJS/HTML5/JS)
frontend
Choose one of the following options/approaches to deploy the UI.

Deploy project via oc CLI
Basic project creation
$ git clone https://github.com/ravichalla/frontend
$ cd frontend/
$ oc new-build --binary --name=frontend -l app=frontend
$ npm install; oc start-build frontend --from-dir=. --follow
$ oc new-app frontend -l app=frontend
$ oc expose service frontend
Specify the OpenShift domain
\$ oc set env dc/frontend OS_SUBDOMAIN=<OPENSHIFT-DOMAIN>

# Using CDK

\$ oc set env dc/frontend OS_SUBDOMAIN=`minishift ip`.nip.io

# Example: OS_SUBDOMAIN=192.168.64.11.nip.io

$ oc set env dc/frontend OS_SUBDOMAIN=192.168.64.11.nip.io
(Optional) Enable Readiness probe
$ oc set probe dc/frontend --readiness --get-url=http://:8080/
Test the service endpoint
Access: http://frontend-helloworld-msa.<openshift-domain>/

###Deploy Kubeflix
Kubeflix provides Kubernetes integration with Netflix open source components such as Hystrix, Turbine and Ribbon.

Specifically it provides:

Kubernetes Instance Discovery for Turbine and Ribbon

Kubernetes configuration and images for Turbine Server and Hystrix Dashboard

kubeflix
Deploy using oc CLI
Execute:

$ oc process -f http://central.maven.org/maven2/io/fabric8/kubeflix/packages/kubeflix/1.0.17/kubeflix-1.0.17-kubernetes.yml | oc create -f -
$ oc expose service hystrix-dashboard --port=8080
\$ oc policy add-role-to-user admin system:serviceaccount:helloworld-msa:turbine
Enable the Hystrix Dashboard in the Frontend
Execute:

\$ oc set env dc/frontend ENABLE_HYSTRIX=true
Test the hystrix-dashboard
Access: http://hystrix-dashboard-helloworld-msa.<openshift-domain>/

###Deploy Jaeger
Jaeger is a fully compatible OpenTracing distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in microservice architectures.

The jaeger project provides all the required resources to start the Jaeger components to trace your microservices.

jaeger
The figure shows a trace detail view for one invocation of the service chaining. We can see the invocation start time, total duration, number of services being invoked and also a total number of spans (an operation in the system).

The showed timeline view allows us to understand relationships between invocations (serial vs parallel) and examine durations of these operations. A span also carries contextual logs and tags which can be queried. Each line represents one span so we can drill down to and see what happened at an operation level in the monitored system.

Deploy using oc CLI
Execute:

$ oc process -f https://raw.githubusercontent.com/jaegertracing/jaeger-openshift/0.1.2/all-in-one/jaeger-all-in-one-template.yml | oc create -f -
$ oc set env dc -l app JAEGER_SERVER_HOSTNAME=jaeger-all-in-one # redeploy all services with tracing
Enable the Jaeger Dashboard in the Frontend
Execute:

\$ oc set env dc/frontend ENABLE_JAEGER=true
Test the Jaeger console
Access: http://jaeger-helloworld-msa.<openshift-domain>/

###Use a SSO server to secure microservices
Keycloak is an open source Identity and Access Management solution aimed at modern applications and services. It makes it easy to secure applications and services with little to no code.

This also applied to logout. Keycloak provides single-sign out, which means users only have to logout once to be logged-out of all applications that use Keycloak.

keycloak
Create a project for the SSO
$ oc new-project sso
Deploy a custom Keycloak instance
$ git clone https://github.com/ravichalla/sso
$ cd sso/
$ oc new-build --binary --name keycloak
$ oc start-build keycloak --from-dir=. --follow
$ oc new-app keycloak
$ oc expose svc/keycloak
(Optional) Enable Readiness probe
$ oc set probe dc/keycloak --readiness --get-url=http://:8080/auth

###Use API Management to take control of your microservices
APIcast (https://github.com/3scale/apicast) is an Open Source API Gateway whose main focus areas are high performance and extensibility. It is part of the Redhat 3scale API Management solution, and is used by hundreds of companies around the world to expose their APIs in a secure and controlled way.

Prerequistes
Install and enable SSO before using this module. SSO is used to issue the JWT tokens during the OpenID Connect (http://openid.net/connect/) over OAuth 2.0 standard code flow to enable the secure connection to the gateway.

Create a custom API Management build
$ git clone https://github.com/ravichalla/api-management
$ cd api-management/
$ oc new-build --binary --name api-management -e BACKEND_URL=http://127.0.0.1:8081
Specify the OpenShift domain
$ oc set env bc/api-management OS_SUBDOMAIN=<OPENSHIFT-DOMAIN>

#Using CDK3
$ oc set env bc/api-management OS_SUBDOMAIN=$(minishift ip).nip.io

#Example: OS_SUBDOMAIN=192.168.64.11.nip.io
$ oc set env bc/api-management OS_SUBDOMAIN=192.168.64.11.nip.io
Create a custom API Management instance
$ oc start-build api-management --from-dir=. --follow
$ oc new-app api-management
$ oc expose svc/api-management --name api-bonjour
$ oc expose svc/api-management --name api-hola
$ oc expose svc/api-management --name api-ola
$ oc expose svc/api-management --name api-aloha
(Optional) Enable Readiness probe
$ oc set probe dc/api-management --readiness --get-url=http://:8081/status/ready
Tell frontend to enable API Management
\$ oc set env dc/frontend ENABLE_THREESCALE=true
Test the API Management Integration
After enabling the API Management use case, you will see a new Tab (or two if you haven’t previously enabled SSO) containing the new scenario.

Click in the API Management tab and you will see a new panel with the information of your deployed services.

Click the Refresh results link, if you are not logged in, you will see the services failing with an "Unauthorized" error.

Log in using the link in the top right corner using the user/user credentials.

After the page refresh click again in the API Management tab.

Click the Refresh Results link to call the managed API endpoints. You will see the same results as calling them from the browser tab.

Click the link again 6 more times in less than a minute.

Did you notice that now the bonjour service failed with a HTTP 429 Too Many Requests error? The 3scale APIcast allows you to gate (secure or protect) your endopoint without the need to change or add new endpoint in your code. You can proxy your service and add policies, like enabling CORS, doing URL Rewrite or edge rate limiting with a simple configuration.

###Use case: Configuring your application
hola microservice uses DeltaSpike Configuration to obtain the phrase that is displayed.

DeltaSpike allows the configuration to come from different ConfigSources (System properties, Environment variables, JNDI value, Properties file values). The microservices needs a configuration value called "hello" to display the provided value. If no value is informed, the default value (Hola de %s) is used.

We will use 2 different approaches to configure "hola" microservice:

Environment variables

Openshift ConfigMaps

Environment variables
Update the DeploymentConfig:

\$ oc set env dc/hola hello="Hola de Env var"

# To unset the env var

\$ oc set env dc/hola hello-
ConfigMaps
As you’ve seen, the definition of environment variables causes the DeploymentConfig to recreate the pods with that value set. OpenShift 3.2 allows the use of ConfigMaps

Create a ConfigMap from an existing file:
$ cd hola/
$ oc create configmap translation --from-file=translation.properties
\$ oc get configmap translation -o yaml
Mount a volume from ConfigMap
Here we are creating a mounted volume in all pods that reads the file.

\$ oc patch dc/hola -p '{"spec":{"template":{"spec":{"containers":[{"name":"hola","volumeMounts":[{"name":"config-volume","mountPath":"/etc/config"}]}],"volumes":[{"name":"config-volume","configMap":{"name":"translation"}}]}}}}'
Update the microservice to read the file
The POD needs a Java System Property to inform the location of the file.

\$ oc set env dc/hola JAVA_OPTIONS="-Dconf=/etc/config/translation.properties"
After the deployment, check that the config is present

$ oc get pods -l app=hola
$ oc rsh hola-?-?????
sh-4.2$ cat /etc/config/translation.properties
sh-4.2$ exit
Update the ConfigMap and wait the POD to get updated.

\$ oc edit configmap translation
Note
The synchronization between the ConfigMap and the mounted volume takes some time to be performed

###Use case: Deployment Patterns
Blue/Green deployment
Blue/Green deployment is a technique where two different environments/versions (Blue and Green) of the same application are running in parallel but only one of them is receiving requests from the router at a time.

To release a new version, the router is updated to switch between the "Blue" environment and point to the "Green" environment.

greendeployment
We can use 2 different approaches to perform Blue/Gren deployment:

Changing the Routes

Changing the Services

Blue/Green deployment using Routes
This strategy is based on Openshift deployment examples and it uses the API-Gateway project, but it could be used with any other project:

Modify the service.

\$ cd /api-gateway

\$ vim src/main/java/com/redhat/developers/msa/api_gateway/CamelRestEndpoints.java

# replace .transform().body(List.class, list -> list)

# by .transform().body(List.class, list -> list.stream().map(res -> "UPDATED - " + res).collect(java.util.stream.Collectors.toList()))

Create a "new" version of the application using a different name..

$ oc new-build --binary --name=api-gateway-blue -l app=api-gateway-blue
$ mvn package && oc start-build api-gateway-blue --from-dir=. --follow
\$ oc new-app api-gateway-blue -l app=api-gateway-blue,hystrix.enabled=true
Enable Readiness probe

\$ oc set probe dc/api-gateway-blue --readiness --get-url=http://:8080/health
Switch the route to the "new" application.

\$ oc patch route/api-gateway -p '{"spec": {"to": {"name": "api-gateway-blue" }}}'

# To return to the "old" application

\$ oc patch route/api-gateway -p '{"spec": {"to": {"name": "api-gateway" }}}'
(Optional) Remove the "old" version.

$ oc delete is,bc,dc,svc api-gateway
$ oc delete builds -l build=api-gateway
Blue/Green deployment using Services
This approach is more suitable for Services with internal access only. It will modify the existing Kubernetes Service to point to the new DeploymentConfig

Modify the service.

\$ cd /bonjour

\$ vim lib/api.js

# go to line 32

# replace const sayBonjour = () => `Bonjour de ${os.hostname()}`;

# by const sayBonjour = () => `Version 2 - Bonjour de ${os.hostname()}`;

Create a "new" version of the application using a different name.

$ oc new-build --binary --name=bonjour-new
$ npm install && oc start-build bonjour-new --from-dir=. --follow
\$ oc new-app bonjour-new
Enable Readiness probe

\$ oc set probe dc/bonjour-new --readiness --get-url=http://:8080/api/health
Switch the Service to the "new" application.

\$ oc patch svc/bonjour -p '{"spec":{"selector":{"app":"bonjour-new","deploymentconfig":"bonjour-new"}}}'

# To return to the "old" application

\$ oc patch svc/bonjour -p '{"spec":{"selector":{"app":"bonjour","deploymentconfig":"bonjour"}}}'
Canary deployments
Canary deployments are a way of sending out a new version of your app into production that plays the role of a “canary” to get an idea of how it will perform (integrate with other apps, CPU, memory, disk usage, etc). Canary releases let you test the waters before pulling the trigger on a full release.

canarydeployment
This strategy is based on Openshift deployment examples and it uses the Bonjour project, but it could be used with any other project:

Modify the service.

\$ cd /bonjour

\$ vim lib/api.js

# go to line 32

# replace const sayBonjour = () => `Bonjour de ${os.hostname()}`;

# by const sayBonjour = () => `Version 2 - Bonjour de ${os.hostname()}`;

Create a "new" version of the application using a different name..

$ oc new-build --binary --name=bonjour-new
$ npm install && oc start-build bonjour-new --from-dir=. --follow
\$ oc new-app bonjour-new
Enable Readiness probe

\$ oc set probe dc/bonjour-new --readiness --get-url=http://:8080/api/health
Apply a common label (svc=canary-bonjour) for both versions

$ oc patch dc/bonjour -p '{"spec":{"template":{"metadata":{"labels":{"svc":"canary-bonjour"}}}}}'
$ oc patch dc/bonjour-new -p '{"spec":{"template":{"metadata":{"labels":{"svc":"canary-bonjour"}}}}}'
Update the Service to point to the new label

\$ oc patch svc/bonjour -p '{"spec":{"selector":{"svc":"canary-bonjour","app": null, "deploymentconfig": null}, "sessionAffinity":"ClientIP"}}'
Simulate multiple clients

\$ oc scale dc/bonjour --replicas=3

# Simulate 10 new clients and note that 25% of the requests are made on bonjour-new.

\$ for i in {1..10}; do curl http://bonjour-helloworld-msa.`minishift ip`.nip.io/api/bonjour ; echo ""; done;

# Simulate the same client

$ curl -b cookies.txt -c cookies.txt  http://bonjour-helloworld-msa.$(minishift ip).nip.io/api/bonjour

# To change the credentials, delete the cookies file

\$ rm cookies.txt
To return to the original state

$ oc scale dc/bonjour --replicas=1
$ oc delete all -l app=bonjour-new
\$ oc patch svc/bonjour -p '{"spec":{"selector":{"svc":null ,"app": "bonjour", "deploymentconfig": "bonjour"}, "sessionAffinity":"None"}}'
