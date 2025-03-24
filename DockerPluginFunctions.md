The Docker Pipeline Plugin provides a docker global variable in Jenkins Pipelines. Here are the functions available:

# Image Management
| Function                              | Description                                           |
|-------------------------------------- | ----------------------------------------------------- |
| docker.build(imageName, args)	        | Builds a Docker image from a Dockerfile.              |
| docker.image(imageName)	            | References an existing Docker image.                  |
| docker.image(imageName).pull()	    | Pulls the latest version of an image from a registry. |
| docker.image(imageName).push(tag)	    | Pushes the image to the registry with an optional tag.|
| docker.image(imageName).tag(newTag)	| Tags an image with a new name.                        |

# Running Containers

| Function                                  | Description                                                           |
|------------------------------------------ | ----------------------------------------------------------------------|
| docker.image(imageName).run(args)         | Runs a container from an image and executes a command.                | 
| docker.image(imageName).inside(args)      | Runs a container in the background and executes steps inside it.      |
| docker.image(imageName).withRun(args) {}	| Runs a container and automatically stops it after the block execution.|

# Registry Management

1. docker.withRegistry(registryUrl, credentialsId) {}	-- Logs into a Docker registry and executes the block.
2. docker.withServer(server, credentialsId) {}	 -- Connects to a remote Docker server.

# Docker Server and Tools

1. docker.withTool(toolName) {}	-- Uses a specific Docker tool installation.
2. docker.node(runArgs) {}	-- Runs a Jenkins agent inside a Docker container.


# Docker Plugin Functions (For Jenkins Agents)

1. dockerTemplate(name, image, options, remoteFs, instanceCap, retentionStrategy, pullStrategy, environments) -- Defines a Jenkins agent template.
2. dockerTemplateBase(image, dnsString, network, dockerCommand, volumesFrom, volumes, environments, extraHosts, bindPorts, privileged, tty, macAddress, labelString, hostName, alwaysPullImage)	 -- Configures a base template for agents.

# Functions for Container Management

1. docker.createContainer(image, args)	-- Creates a Docker container.
2. docker.startContainer(containerId)	-- Starts a container.
3. docker.stopContainer(containerId)	-- Stops a running container.
4. docker.removeContainer(containerId)	-- Removes a container.
5. docker.listContainers()	-- Lists all running containers.


# 1. Docker Image Management Functions Examples

1. docker.build(imageName, args) - Build a Docker Image
```groovy

pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('my-app:latest', '-f Dockerfile .')
                }
            }
        }
    }
}

```
2. docker.image(imageName).pull() - Pull an Image from Registry

```groovy
pipeline {
    agent any
    stages {
        stage('Pull Docker Image') {
            steps {
                script {
                    docker.image('nginx:latest').pull()
                }
            }
        }
    }
}
```

3. docker.image(imageName).push(tag) - Push Image to Docker Hub

```groovy

pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "myrepo/my-app"
    }
    stages {
        stage('Build & Push Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                        docker.image(DOCKER_IMAGE).push('latest')
                    }
                }
            }
        }
    }
}

```
# 2. Running Containers Examples

1. docker.image(imageName).run(args) - Run a Container

```groovy
pipeline {
    agent any
    stages {
        stage('Run Container') {
            steps {
                script {
                    def container = docker.image('mysql:latest').run('-e MYSQL_ROOT_PASSWORD=root -d')
                    echo "Container started with ID: ${container.id}"
                }
            }
        }
    }
}

```
2. docker.image(imageName).inside(args) - Execute Commands Inside a Running Container

```groovy
pipeline {
    agent any
    stages {
        stage('Run Inside Container') {
            steps {
                script {
                    docker.image('maven:3.8.6').inside {
                        sh 'mvn --version'
                    }
                }
            }
        }
    }
}

```

3. docker.image(imageName).withRun(args) {} - Run a Container and Cleanup After Use

```groovy
pipeline {
    agent any
    stages {
        stage('Run Temporary Container') {
            steps {
                script {
                    docker.image('node:14').withRun('-e NODE_ENV=production') { c ->
                        sh 'npm install'
                    }
                }
            }
        }
    }
}


```

# 3. Docker Registry Management Examples

1. docker.withRegistry(registryUrl, credentialsId) {} - Authenticate to Docker Registry
```groovy
pipeline {
    agent any
    stages {
        stage('Docker Login & Push') {
            steps {
                script {
                    docker.withRegistry('https://my-registry.com', 'docker-credentials') {
                        docker.image('myrepo/my-app').push('latest')
                    }
                }
            }
        }
    }
}

```
2. docker.withServer(server, credentialsId) {} - Connect to Remote Docker Server
```groovy
pipeline {
    agent any
    stages {
        stage('Connect to Docker Server') {
            steps {
                script {
                    docker.withServer('tcp://192.168.1.100:2376', 'docker-server-credentials') {
                        docker.image('nginx').run()
                    }
                }
            }
        }
    }
}

```
# 4. Docker Tool Management Examples

1. docker.withTool(toolName) {} - Use a Specific Docker Tool
```groovy
pipeline {
    agent any
    stages {
        stage('Using Docker Tool') {
            steps {
                script {
                    docker.withTool('my-docker-tool') {
                        sh 'docker --version'
                    }
                }
            }
        }
    }
}

```
# 5. Docker Agent Management (Docker Plugin)

1. dockerTemplate(name, image, options, remoteFs, instanceCap, retentionStrategy, pullStrategy, environments) - Define a Jenkins

```groovy
dockerTemplate(
    name: 'my-agent',
    image: 'jenkins/inbound-agent',
    options: '--privileged',
    remoteFs: '/home/jenkins',
    instanceCap: 5,
    retentionStrategy: 'always'
)

```

2. docker.node(runArgs) {} - Run a Jenkins Build Agent Inside Docker

```groovy

pipeline {
    agent {
        docker {
            image 'maven:3.8.6'
            args '--entrypoint=""'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

```

3. Running a Service and Stopping it Later

```groovy

pipeline {
    agent any
    stages {
        stage('Start MySQL') {
            steps {
                script {
                    def container = docker.image('mysql:latest').run('-e MYSQL_ROOT_PASSWORD=root -d')
                    echo "Started MySQL container: ${container.id}"

                    // Perform some operations

                    docker.stopContainer(container.id)
                    echo "Stopped MySQL container"
                }
            }
        }
    }
}

```

# 6. Managing Docker Containers (Docker Plugin)

1. docker.createContainer(image, args) - Create a Container

```groovy
pipeline {
    agent any
    stages {
        stage('Create Container') {
            steps {
                script {
                    def container = docker.createContainer('nginx', '--name my-nginx -d')
                    echo "Container ID: ${container.id}"
                }
            }
        }
    }
}

```

2. docker.startContainer(containerId) - Start a Created Container

```groovy
pipeline {
    agent any
    stages {
        stage('Start Container') {
            steps {
                script {
                    def container = docker.createContainer('nginx', '--name my-nginx -d')
                    docker.startContainer(container.id)
                    echo "Container started with ID: ${container.id}"
                }
            }
        }
    }
}

```

3. docker.removeContainer(containerId) - Remove a Container

```groovy
pipeline {
    agent any
    stages {
        stage('Remove Container') {
            steps {
                script {
                    def container = docker.createContainer('nginx', '--name my-nginx -d')
                    docker.startContainer(container.id)
                    docker.removeContainer(container.id)
                    echo "Container removed"
                }
            }
        }
    }
}

```


there are potential improvements to be done here
