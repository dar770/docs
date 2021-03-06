---
description: Check out the examples explaining how to use databases and background services in VM based environments and with Docker containers.
---

# Using Databases and Services

Semaphore offers a virtual machine (VM) based environment, and a containerized
Docker environment for running your CI/CD pipelines.

## Using databases and background services in VM based environments

In the VM based images, like `ubuntu1804`, use the [sem-service utility][sem-service]
to manage database engines and other services on Semaphore.

Let's say that your CI build needs Redis and PostgreSQL:

``` yaml
# .semaphore/semaphore.yml

version: 1.0

agent:
  machine:
    type: e1-standard-2    # Linux machine type with 2 vCPUs, 4 GB of RAM
    os_image: ubuntu1804   # The Ubuntu 18.04 OS image.

blocks:
  - name: "Test"
    task:
      jobs:
        - name: Tests
          commands:
            - sem-service start redis
            - sem-service start postgres
            - createdb -U postgres -h 0.0.0.0 myapp_database
            - echo 'running tests'
```

Since you have unrestricted access to the job's environment, other options for
running services include installing native packages with `sudo apt-get install`.

## Using databases and background services with Docker containers

Semaphore allows you to run your jobs in a Docker environment, where you can
start multiple containers. The first container will be used to run your commands,
while the rest of the containers will be booted up and linked.

Let's say that your CI build needs Redis and PostgreSQL:

``` yaml
# .semaphore/semaphore.yml

version: v1.0
name: Docker Based Builds

agent:
  machine:
    type: e1-standard-2

  containers:
    - name: main
      image: 'registry.semaphoreci.com/ruby:2.6'

    - name: db
      image: 'registry.semaphoreci.com/postgres:9.6'
      env_vars:
        - name: POSTGRES_PASSWORD
          value: keyboard-cat

    - name: cache
      image: 'registry.semaphoreci.com/redis:5.0'

blocks:
  - name: "Hello"
    task:
      jobs:
      - name: Hello
        commands:
          # install postgres and redis clients
          - apt-get -y update && apt-get install postgresql-client redis-tools

          # create a database by connecting to 'db' container
          - PGPASSWORD="keyboard-cat" createdb -U postgres -h db -p 5432 -e hello

          # list key in redis container by connecting to the cache container
          - redis-cli -h cache KEYS *
```

We used the Semaphore hosted [Postgres](/ci-cd-environment/semaphore-registry-images/#postgres) and [Redis](/ci-cd-environment/semaphore-registry-images/#redis)
images to start your services.

!!! info "Semaphore convenience images redirection"
	Due to the introduction of [Docker Hub rate limits](/ci-cd-environment/docker-authentication/), if you are using a [Docker-based CI/CD environment](/ci-cd-environment/custom-ci-cd-environment-with-docker/) in combination with convenience images Semaphore will **automatically redirect** any pulls from the `semaphoreci` Docker Hub repository to the [Semaphore Container Registry](/ci-cd-environment/semaphore-registry-images/).	

### Using services and test data across blocks

Note that, since all jobs run in isolated environments, the services that you
start in one job are not automatically available in other jobs.
The isolation of jobs from each other within their block also means that
services are not shared across blocks or pipelines.

To use a service or populate test data in all parallel jobs within a block,
specify that in the task [prologue][prologue]. Repeat the same steps in the
definition of each block as needed.

## Next steps

Almost every project has dependencies, and we can save a lot of time by
installing them once and reusing them from a cache. Let's learn how to do that
in [the next section][next].

[prologue]: https://docs.semaphoreci.com/reference/pipeline-yaml-reference/#prologue
[next]: https://docs.semaphoreci.com/guided-tour/caching-dependencies/
[sem-service]: https://docs.semaphoreci.com/ci-cd-environment/sem-service-managing-databases-and-services-on-linux/
