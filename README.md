## Image Manager containerized

Let's build this thing! In this folder run:

```
docker build -t img-mgr .
```

Validate that the image is working by running it using:

```
docker run -dp 5000:5000 img-mgr
```

Open your web browser to `http://localhost:5000`. It doesn't have permissions to upload the files to S3, this is to be expected.

You can kill the container by finding the container id `docker ps`. Once the container ID is known use that to `docker kill CONTAINER_ID` where CONTAINER_ID is the one printed out from `docker ps`.

### Docker compose

[What is a docker-compose.yml](https://docs.docker.com/compose/)? TL;DR allows the management of running more than one container as well as the network, env var and resource mapping.

Elastic Beanstalk makes use of these config files to run your containers. You can make use of them on your machine, too! Try it now:

```
docker compose up
```

Now pull up your web browser and navigate to `http://localhost:80`

control-c break to shut it down.

### ECR

Within the AWS web console create an Elastic Container Registry. The name of the registry can be that of the application, so we'll name it `img-mgr`. All of the other values can be taken as default for this private repo.

Find the "Push commands" for your repo. This will provide guidance on how to get your local image pushed up to this repo.

Push your image up (it's already been built)


### Update docker compose file for EB

You've pushed your docker image up to ECR, update your `docker-compose.yml` to reference **YOUR** ECR repo. For example mine is *267738384267.dkr.ecr.us-west-2.amazonaws.com/img-mgr* find and use yours as the value of `image`

### .ebextensions

Within this folder you'll find `resources.config`. Folder name is important, the files within do not need to be named anything in particular other than ending in `.config`. EB will source this folder for config files that should be mashed together to set up your environment to your spec. This can include your own CloudFormation, environment variables and even shell scripts.

Take a look `resource.config` you'll find

* the S3 bucket that img-mgr requires to store its images
* the IAM policies for access to the S3 Bucket
* IAM policy for access to ECR
* The S3 bucket name is mapped to an environment variable that then is consumed by the `docker-compose.yml` file.

### Package your EB app config

Zip up this directory with your minor changes to `docker-compose.yml`

```
$ zip -r img-mgr.zip .
  adding: Dockerfile (deflated 42%)
  adding: README.md (deflated 51%)
  adding: upload_bucket.json (deflated 82%)
  adding: .ebextensions/ (stored 0%)
  adding: .ebextensions/resources.config (deflated 69%)
  adding: docker-compose.yml (deflated 17%)
```

Ensure that `.ebextensions/` folder is at the root of your zip file!

### Launch in Beanstalk

Within the AWS webconsole, create a new container Elastic Beanstalk application. Choose to upload your newly created `img-mgr.zip` and launch it. This will take a while, so open another AWS web console and watch the cloudformation resources roll out.
