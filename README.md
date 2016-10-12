# multi-container-dsl
DSL for defining multi-container Docker applications, with no vendor lock-in.

This project intends to provide an abstract DSL to describe an group of containers, how it's linked, resource limits, etc.. without binding to a specific runtime. Think of it like the docker-compose.yml file, but doesn't require Docker Compose/Swarm to run the defined set of containers. Adapters will be provided to convert the DSL to different vendor-specific configuration files like Docker Swarm/Composer, Kubernetes Pods definitions, Fleet units & AWS ECS Task definitions.

## Standard LEMP app
```javascript
{
  "name": "demo-lamp-app",
  "globalEnv": [
    "MYSQL_USERNAME=root",
    "MYSQL_PASSWORD=password"
  ],
  "mountPoints": [
    {
      "name": "docker-sock",
      "hostPath": "/var/lib/docker.sock",
      "containerPath": "/var/lib/docker.sock",
      "readOnly": true
    }
  ],
  "persistentVolumes": [
    {
      "name": "mysql-data",
      "meta": "diskType=ssd"
    },
    {
      "name": "redis-data",
      "meta": "diskType=spinning"
    }
  ],
  "containers": [
    {
      "name": "codebase",
      "dockerImage": "private-repository.com/codebase",
      "dockerImageTag": "latest",
      "memoryLimit": "8MB"
    }, 
    {
      "name": "php-fpm",
      "dockerImage": "php",
      "dockerImageTag": "7.1-fpm",
      "memoryLimit": "512MB",
      "volumesFrom": "codebase",
      "ports": [
        "9000:9000"
      ],
      "env": [
        "APP_CONFIG=1"
      ]
    }, 
    {
      "name": "nginx",
      "dockerImage": "nginx",
      "dockerImageTag": "latest",
      "memoryLimit": "512MB",
      "volumesFrom": "codebase",
      "ports": [
        "80:80"
      ]
    },
    {
      "name": "mysql",
      "dockerImage": "mysql",
      "dockerImageTag": "latest",
      "memoryLimit": "512MB",
      "volumesFrom": "mysql-data",
      "ports": [
        "3306:3306"
      ]
    },
    {
      "name": "redis",
      "dockerImage": "redis",
      "dockerImageTag": "latest",
      "memoryLimit": "64MB",
      "volumesFrom": "redis-data",
      "ports": [
        "6379:6379"
      ],
      "mounts": [
        "docker-sock"
      ]
    }
  ]
}
```
