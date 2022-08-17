# Repository:

A repository is the central database of projects, source codes, dependencies, containers etc.
It serves the purpose of fetching data for all the fellow developers to their local environments.

## Repository Management:
Managing repositories is very crucial for an organization especially when it comes to storage costs, bandwidth costs, build times, stability etc.
All of this can be solved with Nexus repositories.

### What is Nexus Repository?
Nexus Repository OSS is an open source repository that supports many artifact formats, including Docker, Java, and npm. With the Nexus tool integration, jobs/pipelines in your toolchain can publish and retrieve versioned apps and their dependencies by using central repositories that are accessible from other environments.

### Why to use Nexus Repository?
Nexus is a repository manager. It allows you to proxy, collect, and manage your dependencies so that you are not constantly juggling a collection of JARs. It makes it easy to distribute your software. Internally, you configure your build to publish artifacts to Nexus and they then become available to other developers.

### Benefits of Nexus Repository:

1. Speedy Builds
When you run your multi-module project in Maven, how do you think Maven knows if it needs to update plugins or snapshot dependencies? It has to make a request for each artifact it needs to test. Even if nothing has changed, if your project depends on a few SNAPSHOTs or if you don't specify plugin version, Maven might have to make tens to hundreds of requests to a remote repository. All of these requests over the public internet add up to real, wasted, time. I've seen complex builds cut build time by 75% after installing a local instance of Nexus. You are wasting time better spent coding waiting for your build to needlessly interrogate a remote Maven repository.

2. Bandwidth savings
The larger the organization, the more critical bandwidth savings can be. If you have thousands of developers regularly wasting good bandwidth to download the same files over and over again, using a repository manager to keep a local cache is going to save you a good deal of bandwidth. Even for smaller organizations with limited budgets for connectivity and IT operations, having to deal with a set of developers maxing out your connection to the internet to download the same things over and over again seems backwards.

3. Saving the bandwidth of Central Maven Repositories
Running the Central Maven repository is no short order. It ain't cheap to serve the millions of requests and Terabytes of data required to satisfy the global demand for software artifacts from the Maven Central repository. Something as simple as installing a repository manager at every organization that uses Maven would likely cut the bandwidth requirements for Central by at least half. If you have more than a couple developers using Maven, install a repository manager for the sake of keeping Central in business.

4. Predictability and Stability
Depending on Central for your day to day operations also means that you depend on having internet connectivity (and on the fact the Central will remain available 24/7). While we're confident in our ability to keep Central running 24/7, you should take some steps of your own to make sure that your development teams isn't going to be surprised by some network outage on either end. If you have a local repository manager, like Nexus, you can be sure that your builds will continue to work even if you lose connectivity.

5. Control and Auditing
We've all worked in places with a developer who might be more interested in experimenting than in working. It is unfortunate to say so, but there are often times when an architect, or an architecture group needs to establish some baseline standards which are going to be used in an organization. Nexus provides just this level of control.
Without a repository manager, you are going to have little control over what dependencies are going to be used by your development team.

6. Ability to Deploy 3rd-party Artifacts
Instead of hand-crafting some POMs, download Nexus and take the two or three minutes it is going to take to get your hands on a free, capable tool that can create such a repository from 3rd-party artifacts. Nexus provides an intuitive upload form which you can use to upload any random free-floating JAR that finds its way into your project's dependencies.

7. Ability to Host Internal Repositories
you are using Maven and everybody has an efficient build that builds every piece of code required from scratch.You can solve a problem like this by splitting projects up and using Nexus as an internal repository to host internal dependencies.

Ex: Consider a company that has 30 developers split into three groups of 10, each group focused on a different part of the system. Without an easy way to share internal dependencies, a group like this is forced either to create an ad hoc filesystem-based repository or to build the entire system in its entirety so that dependencies are installed in every developer's local repository. The alternative is to separate the projects into different modules that all have dependencies on artifacts hosted by a Nexus repository. Once you've done this, groups can collaborate by exchanging compiled snapshot and release artifacts via Nexus. In other words, you don't need to ask every developer to checkout a massive multi-module project that includes the entire organization's code. Each group within the organization can deploy snapshots and artifacts to a local Nexus instance, and each group can maintain a project structure which includes only the projects it is responsible for.

8. Ability to Host Public Repositories
If you are an open source project, or if you release software to the public, Nexus can be the tool you use to serve artifacts to external users.If you were using something like Nexus, which can be configured to expose a hosted repository to the outside world, you could use the packaging and assembly capabilities of Maven and the structure of the Maven repository to make a release that is more easily consumed. And, this isn't just for JAR files and Java web applications; Maven repositories can host any kind of artifact. Nexus, and Maven repositories in general, define a known structure for releases. If you are writing some Java library, publishing it to your own Nexus instance serving a public repository will make it easier for people to start using your code right away.


### How to use Nexus Repository:

1. Configuring your clients and projects to use your Nexus repository for maven:
Add this in your ~/.m2/settings.xml file. This will configure the credentials to publish to your hosted repos, and will tell your mvn to use your repo as a mirror of central:

        <?xml version="1.0" encoding="UTF-8"?>
            <settings xmlns="http://maven.apache.org/SETTINGS/1.1.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">

            <servers>
                <server>
                <id>nexus-snapshots</id>
                <username>admin</username>
                <password>admin123</password>
                </server>
                <server>
                <id>nexus-releases</id>
                <username>admin</username>
                <password>admin123</password>
                </server>
            </servers>

            <mirrors>
                <mirror>
                <id>central</id>
                <name>central</name>
                <url>https://nexus.organization-url/repository/ls-maven-group/</url>
                <mirrorOf>*</mirrorOf>
                </mirror>
            </mirrors>

            </settings>
        
        ...


And now configure your projects.
If you want only to download dependencies from Nexus, put this in the pom.xml:


        <project ...>
        
            ...

            <repositories>
                <repository>
                <id>maven-group</id>
                <url>https://nexus.lifesight.io/repository/ls-maven-group/</url>
                </repository>
            </repositories>
        </project>
