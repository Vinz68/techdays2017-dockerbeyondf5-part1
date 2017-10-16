# Demo script TechDays 2017 - Docker Beyond F5 - LT to Cluster #


## Demo 0 ##
```
docker pull rvanosnabrugge/killerapp
docker run -d -p 4242:80 rvanosnabrugge/killerapp
```

## Demo 1: Why I really like Docker ##
In this first demo I want to show how Docker can really help the developer in his daily workflow, and at the same time walk through some concepts of Docker.  I start out with the question "Who has ever installed a SQL Server on a new Machine". "And how long did that take?" 

The idea then, is to start a SQL Server Management Studio and connect to localhost. It fails, so I start a command line and show how I don't have any containers running and start up a SQL Server

```
docker login
docker pull store/microsoft/mssql-server-linux:2017-latest
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Welkom1234!" -e "MSSQL_PID=Standard" -p 1433:1433 --name sqlserver -d store/microsoft/mssql-server-linux:2017-latest
```


Then connect again in SQL Management Studio and create the Nortwind Database

Stop the container, and commit the changes, explaining the layers

`docker commit sqlserver repo/northwind`

`docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Welkom1234!" -d -p 1433:1433 --name sqlserver-rvo repo/northwind` 

## Demo 2: Move your app to a container ##
In this demo I want to move my existing, dotnet core, Self Hosted application towards a container. I open the solution with only the dotnetcore stuff in it, and show how it looks. Then, I add Docker support. But what does that do? It adds files. Docker-Compose, a new Project, Docker File.

I open the Docker file and tell about the layers that you see here. That stuff is added to a container, and that when the container is run, the application is started on a port.

I also explain that the Dockerfile is responsible for describing what is inside your container. That you can use this file to build a container image and that you can distribute that image.

I run the docker command in Visual Studio, and show how you can debug in the container.. So that does also work flawless on the command line ? No !

I open a command line and run the command

```
cd demo
docker build -t repo/techdays-vote-vs . 
```
it fails, because it cannot find the files it needs. Show the Dockerfile and show how the source is an argument that is filled by VS.

```
del ".\vote-net\VotingApp-Net\.dockerignore"
```

Run the following commands on the commandline.
```
dotnet restore
dotnet publish -o publish
docker build -t repo/techdays-vote-vs --build-arg source=./publish .  

docker run -d -p 5000:80 repo/techdays-vote-vs
```

We can also use it for running tests in a container

Start up the VoteTestUI Solution
Run the tests with standard Chrome

```
docker run -d -p 4444:4444 --network techdays  selenium/standalone-chrome 

docker run -d -p 5000:80 --network techdays --name votenet repo/techdays-vote-vs
```

Change the web config setting and tell how you can use the container to run your tests..

Run the tests again 

## Demo 3: Setting up the pipelines ##
Now we have teh containers, it is time to set up the Bakery. You can do different things. You can set up a build for a Composition, or you can set up a build for a container. If you release the application as a whole you can do that. You can of course also set up a pipeline for a specific container and only use that one.

We will set up a build for the composition. But then? Where do you store it. In the container registry. Login to Docker Hub how a Container registry have been set up. 

In the pipeline push the images to Docker Hub. Show how you can pull the images from there..

## Demo 4: Setting up the cluster and doing a release ##
Show a little of the cluster, how it is built and how it works. Open a command line and show Kubectl. The command line tool for Kubernetes. 

`kubectl proxy`

Open the browser and start the image we just created in the build pipeline 

Then open the release pipeline and show how you created yml files that can be deployed. Run the pipeline and tell about how this is an pipeline for dev and production

Then modify the vote app, to show green in stylesheet and text (another branch) . Check-in, trigger the build and release and show the rolling update. Run the load test as part of the pipeline.


`kubectl scale --replicas=1 deployments/dep-voting-web --namespace development`

# Links #
* [Azure Container Registry registration in Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#using-azure-container-registry-acr)
* [Full Screen Mario](https://kotori.me/mario/)
* [Full Screen Mario Docker Container](https://dockerdemos.github.io/fullscreenmario/)
* [Road To ALM - Blog Rene van Osnabrugge](https://roadtoalm.com)
* [Page Object Model Selenium](https://automatetheplanet.com/page-object-pattern/)
* [Docker Voting app](https://github.com/dockersamples/example-voting-app)
* [Environment pattern replacement](https://docs.docker.com/engine/reference/builder/#environment-replacement)
* [selenium Chrome on Docker](http://underthehood.meltwater.com/blog/2016/11/09/using-docker-with-selenium-server-to-run-your-browser-tests/)