# Deployment-of-Flask-Application-with-Redis

Objective: Learn to deploy a Flask application with Redis

Tools required: Docker Desktop and kubectl

Prerequisites: Must have a Docker account or create one at https://www.docker.com/ 


Steps to be followed:
1.	Creating a new directory and adding the required files
2.	Creating and tagging the Flask image
3.	Logging into Docker and pushing the Flask image
4.	Creating the Redis and Flask deployments
5.	Creating the Redis and Flask services
6.	Verifying the Flask application


Step 1: Creating a new directory and adding the required files

1.1	Create and navigate to the directory redis_flask using the following commands:
	 	 mkdir redis_flask 
   	 	cd redis_flask

1.2	Add the code given below to redis_flask/app.py:
    vi app.py

    from flask import Flask
    from redis import Redis

    app = Flask(__name__)
    redis = Redis(host='redis', port=6379)

    @app.route('/')
    def hello():
    count = redis.incr('hits')
    return 'Hello from Docker! I have been seen {} times.\n'.format(count)

    if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

 
1.3	Add the code given below to the Dockerfile:

	vi Dockerfile

	FROM python:3.4-alpine
 	ADD . /code
  	WORKDIR /code
  	RUN pip install -r requirements.txt
  	CMD ["python", "app.py"]

 
1.4	Add the following code to requirements.txt:

	vi requirements.txt

	flask
 	redis

 
Step 2: Creating and tagging the Flask image

2.1	Create a Flask app image using the following command:
	  	docker build -t flask_image .

 
2.2	Tag the image using the following command:
    		docker tag flask_image:latest <docker-id>/flask-image:flask_image_for_redis

 Note: Replace <docker-id> with your docker username


Step 3: Logging into Docker and pushing the Flask image

3.1	Log into Docker using the following command:
	   	 docker login

 

3.2	Push the Flask image to the Docker repository:
	   	 docker push <docker-id>/flask-image:flask_image_for_redis

 
Step 4: Creating the Redis and Flask deployments

4.1	Add the following code to the redis.yaml file:

vi redis.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}

 

4.2	Create the Redis deployment using the following command:
   		 kubectl create -f redis.yaml
 

4.3	Add the following code to the flask.yaml file:

	vi flask.yaml
  
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: flask
  name: flask
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: flask
    spec:
      containers:
      - image: 7337568/flask-image:flask_image_for_redis
        name: flask-image
        resources: {}
status: {}


Note: Replace the image repository with yours accordingly.


4.4	Create the Flask deployment using the following command:
	    	kubectl create -f flask.yaml


Step 5: Creating the Redis and Flask services

5.1	Add the following code to the redis-svc.yaml file:

	vi redis-svc.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: redis
  name: redis
spec:
  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    app: redis
status:
  loadBalancer: {}

5.2	Create the Redis service using the following command:
	    kubectl create -f redis-svc.yaml

 
5.3	Add the following code to the flask-svc.yaml file:

	vi flask-svc.yaml

	apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: flask
  name: flask
spec:
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: flask
status:
  loadBalancer: {}

 
5.4	Create the Flask service using the following command:
    		kubectl create -f flask-svc.yaml

 
Step 6: Verifying the Flask application

6.1	Check if the Flask app is working using the following commands:
	   	 kubectl get svc
	   	 curl 10.107.110.98:5000

Note: Use your Flask service cluster IP accordingly

 


