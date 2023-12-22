# [Certified Kubernetes Application Developer (CKAD)](https://www.pluralsight.com/paths/certified-kubernetes-application-developer-ckad-2023)

## Certified Kubernetes Application Developer: Application Design and Build

### Define, Build, and Modify Container Images

The first competency listed under "Application Design and Build" is simply _Define, build and modify container images_, which is pretty vague; however, the top 3 skills that come up again and again are:

* Build images from Dockerfiles, and by extension, debug build issues
* Name, tag, re-tag images, and pull/push images to image registries
* Save images as OCI archives

#### Building

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

#### Re-naming/tagging Images

Say, for example, you build an image without using your username on Docker Hub; in this case, you'll need to rename the image:

```shell
docker image tag <old-image-name> <new-image-name>
```

If you're using the Docker CLI, you don't need to include the registry prefix to the image name (e.g. `docker.io/username/image-name:optional-tag`)

*NOTE:* If you run `docker image ls` you'll see both the old and the new images appear in the list, and both will share the same image ID.

#### Pushing/Pulling Images

According to this course, it's unlikely the exam will ask you to push an image; however, it's almost certain it will ask you to pull an image.

```shell
docker image pull name:optional-tag
```

#### Saving Images

In the event you're asked to save an image as a tar file:

```shell
docker save name:optional-tag --output name.tar
```

Or, gzipped:

```shell
docker save name:optional-tag | gzip > name.tar.gz
```

#### Exam Scenarios

Things to note about the exam:

* The exam environment contains multiple K8s clusters, but each question will highlight which cluster to use and provide the command to connect to it
* Each cluster contains multiple namespaces, and it's **your responsibility to read the question correctly and use the right namespace**

The course repo (https://github.com/nigelpoulton/ckad) contains some scenarios you can use to test yourself on the content from the module.

### Understand Jobs and Cron Jobs

Jobs are the way to run a specific number of pods **to completion.** Jobs provides additional mechanisms like retries, backoffs, cleanup, etc. Jobs are managed by the Jobs Controller, which monitors jobs from the control plane.

Example job manifest:

```yaml
### START JOB TEMPLATE ###
apiVersion: batch/v1
kind: Job
metadata:
  name: ckad1
spec:
  activeDeadlineSeconds: 10
  ttlSecondsAfterFinished: 120
  completions: 5
  parallelism: 1
  backoffLimit: 4
### END JOB TEMPLATE ###
### START POD TEMPLATE ###
  template:
    spec:
      restartPolicy: Never
      ### CONTAINER TEMPLATE ###
      containers:
      - name: ctr
        image: apline:latest
        command: ['sh', '-c' 'echo "Let\'s smash the CKAD!" && sleep 60']
### END POD TEMPLATE ###
```

Items to note in the above manifest example:

* `completions: 5`: stipulates the job should spin up 5 pods
* `parallelism: 1`: stipulates that only 1 pod should run at a time
* `backoffLimit: 4`: stipulates that if there's an error, the job should try to restart the pod a maximum of 4 times using an exponential backoff
* `restartPolicy: Never`: the value here could also be `OnFailure`, which would mean the node tries to spin up the pod as many times as is allowed by `backoffLimit`; however, `Never` stipulates that failed pods remain, which **is extremely helpful when troubleshooting as it means the pod's logs are available after failure**
* `activeDeadlineSeconds: 10`: stipulates the time after which a pod should be terminated if it's still running
* `ttlSecondsAfterFinished: 120`: stipulates the time after which a job should be terminated if it's still running

Cron jobs are basically jobs that are set to run on a defined schedule, which is defined by cron notation (e.g. `0 4 * * 6`, which specifics 04:00 every Sunday). See [docs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) for details.

Cron jobs work on the timezone set on the K8s API server (i.e. control plane) and wakes up every 10 seconds and checks for new tasks. If 100+ schedules for a specific job, it'll throw an error and won't run any more of a job. If you specify `startingDeadlineSeconds: 300`, the cron job controller will check that period, and not the last 10 seconds, for missed schedules.

Setting `concurrencyPolicy: Allow | Forbid | Replace` determines how the cron job controller handles missed jobs:

* `Allow`: allow missed schedules to run in parallel
* `Forbid`: new schedules will not be run if a current job is running
* `Replace`: if a current job is running at the time when the controller checks for missed schedules, the new job will replace the existing

**NOTE:** The exam will often specify a specific namespace to use for a question, so be sure you're memorized how to set that context:

```shell
kubectl config set-context --current --namespace=<NAMESPACE>
```


