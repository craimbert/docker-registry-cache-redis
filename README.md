# docker-registry-cache-redis
How to set up a Docker Registry v2 using redis for caching

##Run redis container
````
$ docker run --name cache-redis -d -p 6379:6379 redis
````

##Set up the IP of the host
If you are using docker-machine, use the IP of that docker-machine; or just use "localhost" otherwise
````
MY_IP=$(docker-machine ip $(docker-machine active))
````
##Run Docker Registry v2 container
For official documentation about registry configuration -> go [here](https://docs.docker.com/registry/configuration/).
<br>Example:
* specify redis as the caching method:  define env var REGISTRY_STORAGE_CACHE_BLOBDESCRIPTOR=redis
* define redis minimal conf: define env var REGISTRY_REDIS_ADDR=$MY_IP:6379
````
$ docker run -d -p 5000:5000 \
	--restart=always \
	--name registry \
	-e REGISTRY_STORAGE_CACHE_BLOBDESCRIPTOR=redis \
	-e REGISTRY_REDIS_ADDR=$MY_IP:6379 \
	registry:2
````  	
##Look at the setup redis-registry running
````
$ docker logs -f registry
````
###Startup
In the startup logs you can see: 
````
level=info msg="using redis blob descriptor cache"
````
=> redis enabled as caching storage
###Push an image to the regisry
From another terminal, push an image to the docker registry:
````
$ docker tag httpd:latest localhost:5000/httpd
$ docker push localhost:5000/httpd
````
In the logs there is: 
````
level=info msg="redis: connect 192.168.99.100:6379"
````
 => connection succesful with redis server

##Inspect redis DB: run the redis cli-client & query the k/v store
````
$ docker run --rm -it \
	  --link cache-redis:redis \
	  redis \
	  sh -c 'exec redis-cli -h "$REDIS_PORT_6379_TCP_ADDR" -p "$REDIS_PORT_6379_TCP_PORT"'
	
172.17.0.107:6379> info keyspace
# Keyspace
db0:keys=15,expires=0,avg_ttl=0

172.17.0.107:6379> keys *
 1) "blobs::sha256:f3509cb04d70473ffa680744de59bd4e29fd1ae274114a4ed4415721c0e714d6"
 2) "repository::httpd::blobs::sha256:f8efbffe7b954b520805da80ce0cce94e3834482c384c25c8851db98696e7f70"
 3) "repository::httpd::blobs::sha256:5b3039cd40c998628a5754fb63d6570b03b658352990b5d2017b4b97db2455dd"
 4) "repository::httpd::blobs::sha256:94118c67a40b1e95e1f8a573c18b080dcbf9f93cbdad4ee00da83ba44122727c"
 5) "blobs::sha256:f8efbffe7b954b520805da80ce0cce94e3834482c384c25c8851db98696e7f70"
 6) "repository::httpd::blobs::sha256:f3509cb04d70473ffa680744de59bd4e29fd1ae274114a4ed4415721c0e714d6"
 7) "repository::httpd::blobs"
 8) "blobs::sha256:a7ff98debbdc22c6a717d4a86bfb99fcd0d4f8534a533ed94089dddaf5c4a94a"
 9) "repository::httpd::blobs::sha256:fffb6eca3dac77bd31f9de1409593b05a63ecfc7204637bdc183263a8379f38e"
10) "blobs::sha256:fffb6eca3dac77bd31f9de1409593b05a63ecfc7204637bdc183263a8379f38e"
11) "blobs::sha256:94118c67a40b1e95e1f8a573c18b080dcbf9f93cbdad4ee00da83ba44122727c"
12) "repository::httpd::blobs::sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
13) "blobs::sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
14) "repository::httpd::blobs::sha256:a7ff98debbdc22c6a717d4a86bfb99fcd0d4f8534a533ed94089dddaf5c4a94a"
15) "blobs::sha256:5b3039cd40c998628a5754fb63d6570b03b658352990b5d2017b4b97db2455dd"
````
--> SUCCESS: the Docker registry used redis as a caching system
