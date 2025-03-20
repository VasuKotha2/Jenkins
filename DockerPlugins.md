The Docker Pipeline Plugin provides a docker global variable in Jenkins Pipelines. Here are the functions available:

| Function                              | Description                                           |
|-------------------------------------- | ----------------------------------------------------- |
| docker.build(imageName, args)	        | Builds a Docker image from a Dockerfile.              |
| docker.image(imageName)	            | References an existing Docker image.                  |
| docker.image(imageName).pull()	    | Pulls the latest version of an image from a registry. |
| docker.image(imageName).push(tag)	    | Pushes the image to the registry with an optional tag.|
| docker.image(imageName).tag(newTag)	| Tags an image with a new name.                        |

