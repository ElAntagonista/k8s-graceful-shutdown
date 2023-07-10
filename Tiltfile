allow_k8s_contexts("kind-nginx-test")
default_registry("localhost:5005")

docker_build("localhost:5005/php-fpm", context="./app", dockerfile="./app/Dockerfile", extra_tag="0.1")
docker_build("localhost:5005/nginx-test",  context="./nginx",  dockerfile="./nginx/Dockerfile", extra_tag="0.1")

k8s_yaml("ingress.yaml")
k8s_yaml("k8s.yaml")
        