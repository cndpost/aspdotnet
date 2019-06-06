ASP.NET core learning notes.  

Most of the materials are taken from https://asp.net


1. The steps to build a microservices using dotnet core:

    1.0) On windows, install VS 2017 or better. 
    1.1) On Linux. such as Redhat, do:
          
          yum install rh-dotnet22 -y

    1.2) check current installed version of dotnet by: 
    
          $>dotnet

    1.3) create a microservice project template:
        
        dotnet new webapi -o myMicroservice --no-https
        cd myMycroservice
        dotnet run

        then you will be able to see the service at:

          http://localhost:5000/api/Values


        You can expand more microservices and api routings manully from above template files

    1.4) Build a docker image, push to registry and then use azure kubernest service to pull the image 
         and launch the service in desired number of instances

         1.4.1) type following command to check if docker is installed and also daemon is running:
         
                  docker --version

         1.4.2) cd myMicroservice

              a)  paste following contents to file "Dockerfile":


                    FROM microsoft/dotnet:2.2-aspnetcore-runtime AS base
                    WORKDIR /app

                    FROM microsoft/dotnet:2.2-sdk AS build
                    WORKDIR /src
                    COPY myMicroservice.csproj myMicroservice/
                    RUN dotnet restore myMicroservice/myMicroservice.csproj
                    WORKDIR /src/myMicroservice
                    COPY . .
                    RUN dotnet build myMicroservice.csproj -c Release -o /app

                    FROM build AS publish
                    RUN dotnet publish myMicroservice.csproj -c Release -o /app

                    FROM base AS final
                    COPY --from=publish /app .
                    ENTRYPOINT ["dotnet", "myMicroservice.dll"]


              b) using following command to build the docker image:
                  
                    docker build -t mymicroservice .

              c) using following command to check if the image is built and exist:

                    docker image ls

              d) using following command to test the image that it works locally:
                      
                      docker run -it --rm -p 3000:80 mymicroservice

                    If it works, it should be like the previous command "dotnet run" in 1.3 and you have a service at

                       http://localhost:5000/api/Values


     1.5 deploy the above docker image as instances on azure

            1.5.1)   push the docker image just built to docker registry first:

                     docker login
                     docker tag mymicroservice [YOUR DOCKER USERNAME]/mymicroservice
                     docker push [YOUR DOCKER USERNAME]/mymicroservice
       
            1.5.2)   login to azure:

                     a) az login

                     b) install azure kubernetest service:

                         az aks install-cli

                     c) create a resource group:

                         az group create --name myMicroserviceResources --location westus

                     d) create a AKS cluster:

                        az aks create --resource-group myMicroserviceResources --name myMicroserviceCluster --node-count 1 --enable-addons        
                            http_application_routing --generate-ssh-keys

                     e) Run the following command to download the credentials to deploy to your AKS cluster:

                        az aks get-credentials --resource-group myMicroserviceResources --name myMicroserviceCluster

 
                     f) cd myMicroservice

                        enter following content into file : deploy-myMicroservice.yaml


                        ---
                        apiVersion: apps/v1beta1
                        kind: Deployment
                        metadata:
                        name: mymicroservice
                        spec:
                        replicas: 1
                        template:
                            metadata:
                            labels:
                                app: mymicroservice
                            spec:
                            containers:
                            - name: mymicroservice
                                image: [YOUR DOCKER ID]/mymicroservice:latest
                                ports:
                                - containerPort: 80
                                env:
                                - name: ASPNETCORE_URLS
                                value: http://*:80
                        ---
                        apiVersion: v1
                        kind: Service
                        metadata:
                        name: mymicroservice
                        spec:
                        type: LoadBalancer
                        ports:
                        - port: 80
                        selector:
                            app: mymicroservice
                                            


                             
                        
                  g)   launch the instances using kubernetes:

                       kubectl apply -f deploy-myMicroservice.yaml


                  h)    check the service is up and running:

                      kubectl get service mymicroservice --watch


                   i) now your service is up and running at following URL in azure:

                       http://[YOUR EXTERNAL IP ADDRESS]/api/Values.


                       This [Your external IP] will be listed when you run above command in h) to check the service

                   j) you can run following command to adjust the scale you needed for performance:

                        kubectl scale --replicas=2 deployment/mymicroservice
                           

              

2. The steps to build a more generic webapi:

   2.1 use VS 2017 to create a project using template:
   
   

    From the File menu, select New > Project.
    Select the ASP.NET Core Web Application template. Name the project TodoApi and click OK.
    In the New ASP.NET Core Web Application - TodoApi dialog, choose the ASP.NET Core version. Select the API template and click OK. Do not select Enable Docker Support.


   
     build and run the project, then you can see the web app runs at following URL:

     https://localhost:<port>/api/values, where <port> is a randomly chosen port number.
   
   
   
     2.2 More features see the tutorial at https://asp.net


     2.3 to test you can use postman to do testing for all the get/put/post/delete and other methods
   
   
   
   
   
   
   
   
   
   
   
   
    "New ->Project; New ASP.NET Core Web Application; 

