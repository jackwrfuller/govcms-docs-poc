# GovCMS Platform

The GovCMS hosting platform is designed to host Drupal CMS websites and support continuous deployment. It uses containerisation to segregate projects and scale to meet traffic demands, as opposed to traditional shared, virtual or dedicated hosting.

## Main components

The key components of the GovCMS platform are described here in point form.

* [GitHub](https://github.com/govCMS) is used for public (open source) code where possible.
* GovCMS has its own private instance of Gitlab for CI and private repositories.
* Your project runs on individual Docker containers (for Nginx, PHP, MariaDB, etc). The base images are [hosted publicly](https://hub.docker.com/u/govcms).
* As a first principle, all projects run independently (they don't share services).
* Where possible, the same Docker images are the basis for local, testing and production containers.
* [Lagoon by amazee.io](https://lagoon.sh) is the collection of services that coordinates the moving parts in the cloud.
* [EKS by AWS](https://aws.amazon.com/eks/) is responsible for deploying Docker containers onto AWS, under instruction from Lagoon.
* Lagoon is **not** required for local development. Local development uses [Docker Compose](https://docs.docker.com/compose/).

## Diagram

This diagram gives an overview of how the elements of the GovCMS platform fit together.

```
 I                      site 1      site 2      site 3     site 4   <---+ Public facing websites,
 N                        ^           ^           ^           ^           each of which is a
 T                        |           |           |           |           segregated project on
 E                        |           |           |           |           the GovCMS platform
 R                   +---------------------------------------------+
 N                   |    Akamai Content Delivery Network (CDN)    |
 E                   |   (DDoS protection, speed + uptime boost)   |
 T                   +---------------------------------------------+
                          ^           ^           ^           ^
                          |           |           |           |
    +--------------------------------------------------------------+
 G  |                | php     | | php     | | php     | | php     |
 O  |                | +-----+ | | +-----+ | | +-----+ | | +-----+ |
 V  |    Lagoon      | nginx   | | nginx   | | nginx   | | nginx   |  <--------+
 C  |                | +-----+ | | +-----+ | | +-----+ | | +-----+ |           |
 M  |                | redis   | | redis   | |         | | solr    |           |
 S  |                +---------------------------------------------+           |
    |                |                                             |           |
 P  |                |   EKS (Kubernetes, Docker images)           |      ----------------------
 L  |                |                                             |      The same Docker images
 A  +--------------------------------------------------------------+      are used locally as in
 T  |                                                              |      the cloud. This way,
 F  |                  Amazon WebServices (AWS)                    |      developers get the most
 O  |             including RDS for mariadb databases              |      accurate possible
 R  |                                                              |      environment to test in.
 M  +--------------------------------------------------------------+      ----------------------
                                  |                                            |
                                  |                                            |
    +--------------------------------------------------------------+           |
    |              GitLab (privately hosted)                       |           |
    |       Testing, Continuous Integration (CI), some code        |           |
    +--------------------------------------------------------------+           |
                ^                                                              |
                |                                                              |
                |                    local1.com    local2.com   local3.com     |
 L      +---------------+          +-------------------------------------+     |
 O      |               |          | php       | | php      | | php      |     |
 C      | Local working |          | +-------+ | | +------+ | | +------+ |     |
 A      | files on      |          | mariadb   | | mariadb  | | mariadb  |  <--+
 L      | on your PC    | +----->  | +-------+ | | +------+ | | +------+ |
        |               |     |    | nginx     | | nginx    | | nginx    |
        +---------------+     |    | +-------+ | | +------+ | | +------+ |
       / + + + + + + + + \    |    | test      | | test     | | test     |
      / + + + + + + + + + \   |    +-------------------------------------+
      =====================   |    |      Docker (Docker Compose)        |
                              |    +-------------------------------------+
                              |
                              |        +----------------+
                              |        |                |
                              |        |  GitHub        |
                              |        |  (public)      |
                              +----->  |                |
                                       |  open source   |
                                       |  code where    |
                                       |  possible      |
                                       |                |
                                       | GovCMS distro  |
                                       | and scaffold   |
                                       |                |
                                       +----------------+

```

## Services

Some more information about the key services we use.

### Amazon EKS

[Amazon Elastic Kubernetes Service (Amazon EKS)](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html) is a managed Kubernetes service that eliminates the need to operate and maintain the availability and scalability of Kubernetes clusters in Amazon Web Services (AWS) and in your own data centers. [Kubernetes](https://kubernetes.io/docs/concepts/overview/) is an open source system that automates the management, scaling, and deployment of containerized applications.

### Kubernetes

[Kubernetes](https://kubernetes.io) is an open source orchestration system, originally created by Google, for automating the deployment, scaling, and management of containerized applications, eliminating many of the manual processes involved in running at scale.

Kubernetes will automatically resolve issues with any container, site or availability zone. Self-healing capabilities ensure automated recovery of individual sites, or in the most extreme case, entire physical data centers (availability zones).

### Lagoon

[Lagoon](https://docs.lagoon.sh) (from amazee.io) is made up of several interconnected open-source services working together on Kubernetes - at a high level, these services and tools include things like GraphQL API, RabbitMQ messaging, and webhook handlers. By using Docker Compose locally, when you push your repository Lagoon determines what images to deploy for you from the same configuration files, which means your remote hosting uses the same images as your local setup.

### GitLab

[GitLab](https://about.gitlab.com) is an open-source Git repository manager with issue management, version control, code review, monitoring, continuous integration and continuous deployment.

In the context of GovCMS hosting, 'GitLab' refers to the privately hosted instance of GitLab managed by the Department of Finance, not the public website 'https://www.gitlab.com'.

GitLab will also be used to manage the release and patch process – allowing precise control over the availability of GovCMS releases, with the ability to create on-demand test environments for agencies to test upcoming features and releases.

### GitHub

[GitHub](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git) is a cloud-based platform where you can store, share, and work together with others to write code.

In the context of GovCMS, the GovCMS distribution, scaffold and relevant tooling are provided to the public on GitHub. Anyone is free to download and use these to test GovCMS features and functionality, learn to build with GovCMS, and can be used in their own hosting environments if desired.
