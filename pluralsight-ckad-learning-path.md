# [Certified Kubernetes Application Developer (CKAD)](https://www.pluralsight.com/paths/certified-kubernetes-application-developer-ckad-2023)

## Certified Kubernetes Application Developer: Application Design and Build

## Define, Build, and Modify Container Images

The first competency listed under "Application Design and Build" is simply _Define, build and modify container images_, which is pretty vague; however, the top 3 skills that come up again and again are:

* Build images from Dockerfiles, and by extension, debug build issues
* Name, tag, re-tag images, and pull/push images to image registries
* Save images as OCI archives

### Building

```shell
docker image build -t name:optional-tag .
```

Builds an [OCI image](https://github.com/opencontainers/image-spec/blob/main/spec.md) using the `Dockerfile` (this is optional if the file is named simply `Dockerfile` and is present in the build context `.`, which is the current director, otherwise the `-f` flag tells Docker which file to use).

*NOTE:* It's always wise in the exam scenario to check your work after you've completed a step/task. In the case of building an image the `ls` command lists all local images

```shell
docker image ls

REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    ee301c921b8a   7 months ago   9.14kB
```

It _is_ also possible to take a running container that may have been modified while running and create an image from it (*NOTE:* the author mentions this as a possible anti-pattern):

```shell
docker commit <container> <image-name>
```

### Re-naming/tagging Images

Say, for example, you build an image without using your username on Docker Hub; in this case, you'll need to rename the image:

```shell
docker image tag <old-image-name> <new-image-name>
```

If you're using the Docker CLI, you don't need to include the registry prefix to the image name (e.g. `docker.io/username/image-name:optional-tag`)

*NOTE:* If you run `docker image ls` you'll see both the old and the new images appear in the list, and both will share the same image ID.

### Pushing/Pulling Images

According to this course, it's unlikely the exam will ask you to push an image; however, it's almost certain it will ask you to pull an image.

```shell
docker image pull name:optional-tag
```

### Saving Images

In the event you're asked to save an image as a tar file:

```shell
docker save name:optional-tag --output name.tar
```

Or, gzipped:

```shell
docker save name:optional-tag | gzip > name.tar.gz
```

### Exam Scenarios

Things to note about the exam:

* The exam environment contains multiple K8s clusters, but each question will highlight which cluster to use and provide the command to connect to it
* Each cluster contains multiple namespaces, and it's **your responsibility to read the question correctly and use the right namespace**

The course repo (https://github.com/nigelpoulton/ckad) contains some scenarios you can use to test yourself on the content from the module.