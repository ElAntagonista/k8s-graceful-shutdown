## Overview
This is a playground to test graceful shutdown options for a php application running in a kubernetes cluster.
Insipiration for this came from - https://blog.palark.com/graceful-shutdown-in-kubernetes-is-not-always-trivial/
The Kubernetes deployment consists of a simple php application running with php-fpm and an nginx container both running in the same pod.
The laravel application randomly sleeps between 20 miliseconds and 850 miliseconds to simulate some work being done.

## Prerequisites
- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Tilt](https://docs.tilt.dev/install.html)
- [Ctlptl](https://github.com/tilt-dev/ctlptl)
**Optional**
- [Vegeta](https://github.com/tsenart/vegeta) for load testing


## Setup
1. The nginx container has been set with the following preStop hook:
```yaml
preStop:
    exec:
    command:
        - sh
        - '-c'
        - sleep 5 && /usr/sbin/nginx -s q
```
2. The php-fpm container has been set with the following preStop hook:
```yaml
lifecycle:
    preStop:
        exec:
        command:
            - sh
            - '-c'
            - sleep 5 && kill -SIGQUIT 1
```
3. The php-fpm docker image has been built with the following config setting up the `process_control_timeout` to 20s: 
```
RUN sed -i 's/;process_control_timeout = 0/process_control_timeout = 20s/' /usr/local/etc/php-fpm.conf
```

## Running the example
1. Start the local kubernetes cluster with `ctlptl apply -f kind.yaml`
2. Start the tilt server with `tilt up`
3. Open the tilt dashboard by pressing spacebar
4. Monitor the tilt dashboard to see when all components are ready

After following these steps, the php application will be available at http://localhost:80 exposed by an nginx ingress controller.


## Testing the graceful shutdown
Test 1
1. Open up a terminal and run `echo "GET http://localhost:80" | vegeta attack -duration=30s | vegeta report --type=text` 
This will hit the application with 50 requests per second for 30 seconds from 10 vegeta workers.
2. In a separate terminal, run `kubectl rollout restart deployment/my-app`

After the vegeta attack is finished, you should see 100% or close to 100% success rate of all requests made to the application.

Test 2
1. Inspect the distribution of the pods by running and note the node that hosts 2 out of the 3 pods.
2. Run the vegeta attack again and in a separate terminal run `kubectl drain <host> --ignore-daemonsets` 
3. After the vegeta attack is finished, you should see 100% or close to 100% success rate of all requests made to the application.

## Notes
- Explicit update of the STOPSIGNAL in both php-fpm and nginx was not needed. It might be needed for different versions.
- The preStop hook on both containers had the biggest effect on the success rate of the requests during rollout/node drain.
- The `process_timeout_control` setting in php-fpm had an effect especially when increasing the response time duration.
