
```
$ docker service scale hello-docker-stack_web=5
hello-docker-stack_web scaled to 5
```

If everything went OK, there should be 5 instances of the service runnning. :sunglasses: 
You can check the status with `docker service ls` 



```
$ docker service logs -f hello-docker-stack_web
```

Your are now seeing on the console the logs from all the instances. If you try to access multiple time to the service you will eventyally see how all the containers forward the logs to you console. Isn't that cool? 

Finally, lets take the app down with docker stack rm:

```
$ docker stack rm hello-docker-stack_web
```


A docker-compose.yml file is a YAML file that defines how Docker containers should behave in production.

In the root of the proyect, you will find a `docker-compose.yml` file describing our first service. 

```YAML
version: "3.3"
services:
  # We only have one service so far that we will name "web"
  web:
    # Image name should be the same that we created on previous section.
    image: hello-docker
    # On this part we define the strategy for scheduling containers. 
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

So, this `docker-compose.yml` file tells `Docker` to do the following:

* Use the image we created in previous section **hello-docker**
* Run 3 instances of that image as a service called web, limiting each one to use, at most, 10% of the CPU (across all cores), and 50MB of RAM.
* Immediately restart containers if one fails.
* Map port 80 on the host to web’s port 80.
* Instruct web’s containers to share port 80 via a load-balanced network called webnet. (Internally, the containers themselves will publish to web’s port 80 at an ephemeral port.)
* Define the webnet network with the default settings (which is a load-balanced overlay network).

Now let’s run it. Go to the `./3-running-services` folder of the project and execute the following command: 

```
$ docker stack deploy -c docker-compose.yml hello-docker-stack
```

Magic ✨🐳

Our single service stack is running 3 container instances of our deployed image on one host. Let’s investigate.




Let's kill one of the worker nodes and see how docker re-schedules its containers: in `play-with-docker` just hit the delete button in any of the worker nodes. If running locally just `docker-machine rm worker2`

Now `docker service ps pinger` repeatedly to see how some of the pop up in the other nodes automatically. How cool is that?

You now have a resilient, distributed application running in a docker swarm cluster ✨


Docker swarms run tasks that spawn containers. Tasks have state and their own IDs:
```
docker service ps <service>
```

You can run `curl http://localhost` several times in a row, or go to that URL in your browser and hit refresh a few times. Either way, you’ll see the container ID change, demonstrating the load-balancing; with each request, one of the 5 replicas is chosen, in a round-robin fashion, to respond.

