Overview
=========

This document is a walkthrough of the demo presented in this talk.

Requirements
------------
This demo uses the following software and cloud resources. 

* Trivy
* Docker Engine Client 20.10.8
* Docker Engine Server 20.10.8
* a clone of this repository
* Alibaba Container Registry to push container images
* GitHub Actions as CI/CD 

**Docker**

		➜ docker version
		Client:
		 Cloud integration: 1.0.17
		 Version:           20.10.8
		 API version:       1.41
		 Go version:        go1.16.6
		 Git commit:        3967b7d
		 Built:             Fri Jul 30 19:55:20 2021
		 OS/Arch:           darwin/amd64
		 Context:           default
		 Experimental:      true
		
		Server: Docker Engine - Community
		 Engine:
		  Version:          20.10.8
		  API version:      1.41 (minimum version 1.12)
		  Go version:       go1.16.6
		  Git commit:       75249d8
		  Built:            Fri Jul 30 19:52:10 2021
		  OS/Arch:          linux/amd64
		  Experimental:     false
		 containerd:
		  Version:          1.4.9
		  GitCommit:        e25210fe30a0a703442421b0f60afac609f950a3
		 runc:
		  Version:          1.0.1
		  GitCommit:        v1.0.1-0-g4144b63
		 docker-init:
		  Version:          0.19.0
		  GitCommit:        de40ad0

		  
The Flask application and Dockerfile
--------------
Navigate to the ```/app``` directory. This contains the sample Flask application, a requirements file and the Dockerfile manifest.

```Dockerfile```, ```app.py```, ```requirements.txt```


Building version 0 of the app and pushing the image to a Container Registry
--------------

Login to Alibaba Container Registry: ```docker login```

	docker login
	Authenticating with existing credentials...
	Login Succeeded


While still on the ```/app``` directory invoke the following command in the CLI:
**```docker build -t hadronp/ubuntu-flask:v0 .```** 

Substitute your DockerHub account name instead of **hadronp** eg. ```my_account/ubuntu-flask:v0```

	docker build -t hadronp/ubuntu-flask:v0 .
	Sending build context to Docker daemon  4.096kB
	Step 1/6 : FROM python:alpine3.7
	alpine3.7: Pulling from library/python
	c67f3896b22c: Already exists 
	8ea0547d7723: Pull complete 
	711fdf82ca0a: Pull complete 
	6002973f8486: Pull complete 
	08ec325072d0: Pull complete 
	Digest: sha256:8e083aa39117533772c0ecd4327403bb1acf9a9fc7ae14deac6fddeb8c28d2c4
	Status: Downloaded newer image for python:alpine3.7
	 ---> b9858a9efe1e
	Step 2/6 : COPY . /app
	 ---> bd100e61a5cb
	Step 3/6 : WORKDIR /app
	 ---> Running in 2d6e4f6a015b
	Removing intermediate container 2d6e4f6a015b
	 ---> 6aeafca5d304
	Step 4/6 : RUN pip install -r requirements.txt
	 ---> Running in 02fe15680891
	Collecting flask==1.0.2 (from -r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
	Collecting Werkzeug>=0.14 (from flask==1.0.2->-r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
	Collecting itsdangerous>=0.24 (from flask==1.0.2->-r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
	Collecting click>=5.1 (from flask==1.0.2->-r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
	Collecting Jinja2>=2.10 (from flask==1.0.2->-r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
	Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->flask==1.0.2->-r requirements.txt (line 1))
	  Downloading https://files.pythonhosted.org/packages/ac/7e/1b4c2e05809a4414ebce0892fe1e32c14ace86ca7d50c70f00979ca9b3a3/MarkupSafe-1.1.0.tar.gz
	Building wheels for collected packages: MarkupSafe
	  Running setup.py bdist_wheel for MarkupSafe: started
	  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
	  Stored in directory: /root/.cache/pip/wheels/81/23/64/51895ea52825dc116a55f37043f49be0939bcf603de54e5cde
	Successfully built MarkupSafe
	Installing collected packages: Werkzeug, itsdangerous, click, MarkupSafe, Jinja2, flask
	Successfully installed Jinja2-2.10 MarkupSafe-1.1.0 Werkzeug-0.14.1 click-7.0 flask-1.0.2 itsdangerous-1.1.0
	Removing intermediate container 02fe15680891
	 ---> 38ee633f943c
	Step 5/6 : EXPOSE 5000
	 ---> Running in d9482c3d9366
	Removing intermediate container d9482c3d9366
	 ---> 372d0a4fe4f9
	Step 6/6 : CMD python ./app.py
	 ---> Running in eea1d28f06eb
	Removing intermediate container eea1d28f06eb
	 ---> c74382bedf20
	Successfully built c74382bedf20
	Successfully tagged hadronp/alpine-flask:v0

Show the newly built image in the terminal:

	docker image ls | grep ubuntu-flask
	hadronp/alpine-flask                       v0                  c74382bedf20        2 minutes ago       91.3MB

Push the container image to DockerHub: ```docker push hadronp/ubuntu-flask:v0```

	docker push hadronp/ubuntu-flask:v0
	The push refers to repository [docker.io/hadronp/ubuntu-flask]
	d06a3aaf817a: Pushed 
	8f83f81506cc: Pushed 
	9d47dcf94482: Layer already exists 
	1926d47f169d: Layer already exists 
	66bd78ef2de1: Layer already exists 
	55ad70ba131b: Layer already exists 
	ebf12965380b: Layer already exists 
	v0: digest: sha256:e6af025f2023fce5834618598162ec1c356753dc234a6fb0f2f1d608038962d8 size: 1786


Spawn a single container from the ```hadronp/ubuntu-flask:v0``` image
----------------

```docker run --detach --publish 5000:5000 hadronp/ubuntu-flask:v0```

	docker run --detach --publish 5000:5000 hadronp/ubuntu-flask:v0
	e3467d02e10d46a59538f7a8e7990f55e1a157b7b5a4a58d27cb54e5c8cf4d9e
	Ξ demo/app-code git:(master) ▶ docker ps
	CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                    NAMES
	e3467d02e10d        hadronp/ubuntu-flask:v0   "/bin/sh -c 'python …"   27 seconds ago      Up 26 seconds       0.0.0.0:5000->5000/tcp   compassionate_snyder

On the terminal type the following to view this impossibly awesome app for the first time. ```curl http://127.0.0.1:5000```


Install ```trivy``` scanner
----------------
Scanner for vulnerabilities in container images, file systems, and Git repositories, as well as for configuration issues

Repository link: **```https://github.com/aquasecurity/trivy```**

Installing in Debian/Ubuntu systems would be:

	sudo apt-get install wget apt-transport-https gnupg lsb-release
	wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
	echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
	sudo apt-get update
	sudo apt-get install trivy


To scan the container we just built run:

	trivy image hadronp/ubuntu-flask:v0
	
To filter vulnerabilities via severity level:
   
	trivy image --severity=MEDIUM,HIGH,CRITICAL hadronp/ubuntu-flask:v0
	
To to ignore unfixable vulnerabilities run::
   
	trivy image --ignore-unfixed --severity=CRITICAL hadronp/ubuntu-flask:v0



Automating the vulnerability scan and integrating to the CI/CD pipeline
-------

For this demo we will use Github Actions to do the following:

* Create two pipelines to trigger on ```dev``` and ```main``` branch merge events
* Checkout the code for each branch
* Build the Docker image
* Use a Trivy Github Action to scan the image
* if everything goes well push the image to Alibaba Container Registry
* simulate a deployment by puling the image from the registry, run the image and then verify by invoking ```curl http://localhost:5000``` on the agent 


Releasing new versions of our awesome Flask application
-------

In an immutable infrastructure paradigm, container images are **recreated** from scratch for each change in configuration and code. This ensures that each deployment in development and production environments is consistent.

Assuming new code is pushed into version control and when Pull requests are merged the pipeline goes through the defined workflow of building, scanning, registry push and deployment

License
-------

MIT

Author Information
------------------

quantum34e@gmail.com
