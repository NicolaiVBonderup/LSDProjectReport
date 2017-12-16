# Large Systems Development Exam Report

**Group 6**

Dennis Michael Rønnebæk

Ebbe Vig Nielsen

Emil Rosenius Pedersen

Rune Vandall Zimsen

Nicolai Vikkelsø Bonderup

## Introduction

This report is written for the Large Systems Development course, on the Software Development top-up bachelor's degree at Copenhagen Business Academy.

The intention of the report is to detail our experiences with the design, implementation and maintenance of both our own HackerNews system, as well as the monitoring of another group's system.

Throughout this report, we will describe our choice of methodologies, architecture and tools, as well as our experiences with the hand-over of a system to a separate group, along with the maintenance of the system in question.

## 1. Requirements, Architecture, Design & Process

### 1.1 System Requirements

##### Purpose of the system

The purpose of the system is to provide the user with a social news website similar to Reddit. The social news network allows users to post articles related to computer science and entrepreneurship. Furthermore, it allows registered users to create their own posts and participate in discussions by leaving replies on other people's posts and comments.

##### Scope of the system

The idea is to build a system that should mimic a number of selected functionalities of [https://hackernews.com](https://hackernews.com/). A full list of features will be provided further down.

##### Objectives and success criteria of the project

1. The system should be available to anyone with an internet connection and a browser from a publicly visible server
2. The system should have an uptime of at least 95% over the course of a 2 month period
3. The system should be able to retrieve messages, even when the backend and database is down.

##### Functional requirements

- Web based application
- All users can read content
- Registered users can post/comment on/up-vote/spam-flag articles
- Registered users with a karma score of 500+ can down vote articles
- All content must be persisted in a database
- All articles must be time stamped
- The system has to have a REST API

##### Nonfunctional requirements

- **Usability**

  The system will be a clone of Hackernews, and as such must have a simple graphical user interface that can be operated through most modern browsers. The simplicity of the graphical user interface is key, as we do not presume user familiarity with any similar systems, and will not be providing any sort of training in its usage.

  The system must provide error messages to the user if an error has occurred, with a descriptive message to inform the user what went wrong.

  The system must present the different stories that have been posted in an orderly, compact yet understandable manner.

  The comments on any given story, along with the nested comments, must be presented with proper formatting and indentation to make a comment thread easy to follow and read.

- **Reliability**

  After going live, the system must have a minimum uptime of 95%. Along with the system uptime, whenever the system does have downtime, whether it be unplanned or due to maintenance, there must be a mechanism in place that will cache incoming content, which will then forward the buffered content to the system once it has gone live again.

  We do not expect for the system to be incredibly popular, so we have fairly low expectations for concurrent user numbers.

- **Performance**

  The system will handle all incoming messages as soon as possible. There will be no data loss, and in case of server up-time, messages received in this period will be posted as soon as the system is up and running in a prioritized FIFO manner.

  As the system is a leisure-based product, the level of criticality is fairly low. Still, the system should be rather responsive, as the application itself is fairly simple, and other products of the same type are pretty responsive.

- **Supportability**

  The system must be properly documented, such that post-release maintenance can be performed in a cost-effective manner. There must be proper test documentation and regression suites, such that additions and changes can be performed more efficiently.

  Maintenance of the system is handled by the development team. The team that receives the system will be able to report errors as issues on the GitHub repository.

- **Implementation**

  The hardware platform will be a remote server operated by a third party company, and as such, neither the development team nor the users will have complete control over the server. As such, if the platform wherein the server is located has issues, the development team will not be capable of restoring the system until those issues have been resolved.

- **Interface**

  The system will two different interfaces; a web-based application with a GUI, operable by normal users who wish to use the system, and a REST API that will allow an external program to publish stories and comments to the system without operating the GUI.

- **Packaging**

  The system will be installed on a remote server by the development team, after which the address and any required credentials are handed over to the users, along with the location of the project's repository, such that they can establish issues found in the system that must be corrected.

- **Legal**

  The system will be published on GitHub under the MIT license. The development team is fully responsible for system failures, and are required to correct failures in a timely manner.

### 1.2 Development Process

#### Feature development

##### Branching

While the developers are working on new features, they will be working in separate version control branches reserved for a single use case. If the use case is spanning towards different components, the use case is divided into subtasks, which are then worked on in separate branches and ultimately merged into the given feature branch. This allows us to restrict ourselves to merge only finished features into the development environment, which can then be manually tested by the person responsible for the quality assurance of the software before they are released to production.

The name of a feature branch for a given use case, must match the ID of the use case in our project management tool. This gives the team an overview over which branches are tied to which use cases, and makes it intuitive for developers to find the related use case in the project management tool, which allows for communication, finding the definition of the acceptance tests, and how to test the implemented functionality in a review situation.

#### Releasing features to the staging environment

Once a developer is finished developing an entire feature, a merge request to the staging environment is initiated, which initiates the review process. In this review, the focus is mainly on the technical aspect of the code, while still keeping the acceptance requirements in mind. The review process is as follows:

- **The first step** is performed automatically, which checks whether feature is automatically mergable with the staging environment. If the system reports, that the merge will result in conflicts, the conflicts must be fixed in the feature branch before it can pass the review process.
- **The second step** is performed by automatically triggering a deployment to our continuous integration software, which will deploy the commit to a build server, and run a configuration of unit tests, integration tests and e2e-tests. The build must pass with a code zero, meaning that the application was successfully deployed and all tests passed.
- **The third step** is performed by a developer, who is assigned to review the task. In this step, the reviewer will test that the code functions on his machine as per the use case specification, and will also review the source code line-by-line, going through a number of technical requirements such as modularity, performance, ease of reading, and code standards.

If the reviewer adds any comments, changes or additions in the review these issues are to be resolved, before the merge request is re-initiated. This is often done via pair-programming, to allow sharing of knowledge and discussion of the changes.

Once the review has passed, the code is merged into the staging branch, where it resides until the project manager and quality assurance decides to deploy a build to production.

#### Releasing features to production

When a feature is considered shippable, a release-to-production request can be initiated. Releasing features into production is done by initiating an additional merge request onto the production environment. This will trigger a build process similar to the one towards the staging environment, but this time, the focus is on the quality assurance and requirements specification.

- **The first step** is again performed automatically, which checks whether feature is automatically mergable with the production environment. If the system reports, that the merge will result in conflicts, the conflicts must be fixed in the development branch before it can pass the review process. Although this will almost never happen, the possibility is still there.
- **The second step** is again performed by automatically triggering a deployment to our continuous integration software, which will deploy the commit to a build server, and run a configuration of unit tests, integration tests and e2e-tests. The build must again pass with a code zero, meaning that the application was successfully deployed and all tests passed.
- **The third step** is performed by the project manager and the person responsible for quality assurance. In a real world example, the product owner and other key figures would be a part of this review. In this review, the functionality of the features are taken into consideration, making sure they live up to the requirements specification.

Once the review has passed, the code is merged into the production branch, where it triggers one last build with the continuous integration software. This might be seen as a bit overkill, but it is there to ensure that the features released into production are 200% tested. If the build passes with a code zero, and automatic deployment process is initiated.

#### Deployment process

Once the deployment process is initiated and the continuous integration build passes with a code zero, GitHub will automatically query a web hook, which notifies our cloud infrastructure, that a new production build is ready for release.

Here we use AWS CodeDeploy, to automatically handle the deployment process for us, using a blue/green deployment strategy. With this deployment strategy, every time we reploy a new revision, we provision a set of 3 replacement instances residing in a new automatically scalable instance group. On each instance CodeDeploy downloads and installs the latest version of our application, based on our configuration in each subsystem. If everything goes as planned, CodeDeploy performs a health check of each instance, determining whether the replacement instances are ready for deployment.

CodeDeploy then reroutes load balancer traffic from the existing set of instances running the previous version of our application to the set replacement instances running the new version. After traffic is rerouted to the new instances, the original instances are terminated after 15 minutes.

Blue/green deployments allow us to test the new application version before sending production traffic to it. If there is an issue with the newly deployed application version, we can roll back to the previous version faster than with in-place deployments, and additionally, the instances provisioned for the blue/green deployment will reflect the most up-to-date server configurations since they are new.

### 1.3 Software Architecture

##### Introduction

Our production environment consists of three subsystems. A front-end web application built with the Typescript MVC framework - Angular 4, and a REST API/back-end application built with PHP 7.0, and a relational database system. The system is cloud based, and utilizes Amazon Web Services for all of it's server infrastructure.

##### Automatically scalable instance groups

The application subsystems reside in it's own automatically scalable instance group, which by default contains three EC2 instances that are distributed across three different subnets in three different data centers. The EC2 instances are responsible for running the application code, which is stored on persistent block level storage volumes. The total number of EC2 instances running our application code is 6, and 12 while the deployment is in process on both subsystems consecutively.

##### Load balancing

In front of each application subsystem is a dynamic load balancer, which allows us to scale horizontally in any subsystem layer across the entire system, rapidly deploy new application versions with zero downtime, automatically handle backup plans in case of power outages and protect us against nature catastrophes. If one instance is somehow terminated unexpectedly, another instance is immediately deployed with our application and rerouted through the load balancer.

##### Database management

Our database is stored on a separate AWS RDS instance, which is only available inside our virtual private cloud. RDS makes makes it easy to set up, operate, and scale a relational database in the cloud.

### 1.4 Software Design

#### Backend

We wanted to create a very flexible design that was easy to change, so we agreed upon using modular components and follow SOLID rules when writing the code. That way we could ensure that methods and classes stayed simple and flexible, and easy to understand for anyone that didn't write the code. 

We wanted to create a RESTful API with 3 different set layers behind it (Controllers, Facades and Access) and middleware when needed. The middleware should take care of authentication and validation, and then the Controllers should handle the request (eg. get query parameters or get body). The facade layer should work as a buffer layer so that methods with multiple options will be delegated from here. The Access layer handles all contact to our database.

We also sketched out the Entity models we thought we were gonna need. What we came up with was

- Post
- Comment
- User

#### Database

To get an idea of how we wanted to structure our database we created an E/R diagram

![](https://imgur.com/QL1HNRp.png)

### 1.5 Software Implementation

#### Backend

In most cases we managed to follow our software design. We had to add a couple of extra classes to handle Helges script, and we found out that we were lacking some properties on the post entity so they also had to be added. Other than that we kept to our plans, and simply delegated Helges posts to another route as we had to add a JSON Web Token to the user trying to post a post. Later we completely ignored the user from Helges script and ended up creating a test user. We did this as it caused an unnecessary strain on our server by having to find the user and then log in with the user, and we had already seen that it worked. Instead we could now just send the test users JWT, which we hardcoded.

During the later stages of the development we had a few major bugs that we had to fix. The first one was pretty critical as it would lay our servers down. The issue was due to the way we were handling Helges posts and comments. We took them all in as posts which meant that the comments were posted without a title. In our database setup we needed a title, and so we wrote "This is a test". The problem originated when we then created a slug from that title. The slug we would create the first time would be "this-is-a-test", and then a slug with a similar name would throw a PDO exception. This would trigger a renaming of the slug with a postfix value based of the number of slugs with identical names. Due to the fact that we were simultaneously handling multiple requests this would trigger a race condition when a lot of comments where posted in a row. Furthermore the slug wasn't indexed therefore causing an even bigger delay on the query. When monitoring this problem we could slowly see the number of database connection increase until we reached more than we could handle, and we had to restart the server. The fix was easy as we just added a uid to the slug from the time it was created instead of using the postfix value every time a duplicate was found.

The other major bug came when we started having a lot of posts in our database. We were using pagination to only show 30 posts at a time on the frontend, but we weren't using an index on our OrderBy value, causing us to have to sort all our posts every time we wanted to extract a post. This was fixed by using an indexed value instead.

We would probably never have encountered the first error in a real environment but us handling the comments as posts in Helges script made it happen. The second error was not noticeable when testing on a local environment but was easily seen as soon as the quantity of data increased. 

#### Database

The database was implemented just as planed, and the only changes was adding a few tuples to post.

## 2. Maintenance & SLA Status

### 2.1 Hand-over

Once we established contact with group C, we created a shared Messenger chat group including every member of each group, such that we could communicate more easily. 

The handover of the system consisted primarily of group C providing us with the URL for their system, wherein we could also find a link directing us to their API documentation. We agreed to submit issues to the GitHub repository for the frontend application of their system.

Initially, the handover of the system we were to operate hit a few hitches, as we were provided with the URL for a test server, rather than the live server. As such, several issues were submitted detailing bugs present only on the test server. When group C were incapable of reproducing the issues we filed, we eventually discovered the underlying error, after which our group began operating their proper live system. 

The quality of the API documentation was sufficiently high, although slightly unfinished in some areas. Being that it properly detailed the routes for each endpoint, the HTTP methods assigned to each route, as well as the structure of the request, we were capable of thoroughly testing their endpoints through direct Postman calls with relative ease.

We were not provided with documentation detailing the exact requirements for the system, and as such, were unable to properly assess whether the system followed requirements or not.

### 2.2 Service-Level Agreement

We were never provided with a service-level agreement from group C. This is possibly due to an early mix-up, wherein both groups had mistakenly sent their SLA's to the group they were operating, rather than the group who were operating their own system. However, although we eventually realized this mistake, and spoke to group C about it, we were never provided with an SLA to discuss.

Later on, we discovered their SLA located in a folder in one of their repositories connected to the system. However, the document looked more like a placeholder, wherein the only agreement stated the system's total uptime must not fall under 95%. All other points on the SLA seemed more like descriptions for how the SLA should be written, rather than actual agreements.

Attached below is the SLA that we discovered:

---

# Service Level Agreement

## Introduction
This document includes the service-level agreement between Group B and Group C, wherein we will outline the mutual SLA for the monitoring of the Hacker News system developed and maintained by Group C.

The SLA will include agreed-upon metrics for which the HackerNews system will be maintained by Group B, particularly with regards to the average uptime of the system under observation, mean response time of the system, mean uptime, total uptime and number of requests on a number of different endpoints.

## Up
Up is a simple metric that lets you know whether the service is up or down. It should never be down unless the system is under maintenance. I everything is working it will say Yes.

## Uptime
The total uptime calculated in days.

## Avg uptime (coming soon)
In accordance with the original assignment specification, the minimum acceptable percentile of system uptime must be held at or above 95% of total system lifespan.


## Number of requests (200) from /latestn endpoint
A graph showing all status code 200 requests hitting the /latest endpoint.

---

### 2.3 Maintenance and Reliability

Being that we had no SLA, we had no solid criteria for the maintenance of the system. We also were not provided with dashboard access in proper time, as the dashboard was causing issues on their end. Whether these issues were ever solved is unknown to us.

Generally, maintenance consisted of periodical visits to their live application, testing the different functionalities of the application and observing any errors that would occur. We observed the application to be generally unstable, wherein half of our visits were met with complete unresponsiveness from their server. 

The login and signup functionality of the application was continually broken throughout the duration of the operation, even when the remainder of the application was functional. As such, we were never able to use the application to directly post stories or comments.

Eventually, we created a suite of API tests, which would directly call their service endpoints. These calls were only partially successful, as many of them would be met with either a timeout error, or status codes 404 and 500.

Most issues submitted to the developers were handled within a few days. We communicated with the group several times through chat and in person to discuss the issues, and whether or not they had been resolved. Several issues found by our group were already submitted as issues by members of group C, and as such, we refrained from submitting them ourselves.

## 3. Discussion

### 3.1 Technical Discussion

To summarize the first part, we made a serious effort to make a thorough, well-planned solution to the given project description. The planning process went well, wherein we weighed our options, and decided on a different technology than what was part of our curriculum. Primarily, the technical issues we met in the process were unforeseen tasks. One of the most significant of these issues was the input from the external script, which sent huge amounts of requests over a long period of time. 

Handling these issues turned out to be some of the best learning experiences we've had throughout the semester. For instance, we had a problem with pagination in our system, also described in part 1.5, where the solution was making proper usage of indexing in the database, relating back to theoretical knowledge we had gained in the previous semester. 

At the planning stage of the project we felt well-prepared, but as it usually happens, during the execution we ran into issues that we had not properly prepared for. However, analyzing these issues has been very beneficial, as we can take the knowledge we've gained and make use of it going forward.

The second part with maintaining the HackerNews application for another group went different than we expected, as this part was much more loosely structured. What worked well in this part was the communication through Messenger, which when reflecting over the process probably wasn't the whole idea with functioning as operation team for an application. The SLA should have been more guide of how to speak and communicate with the developing application team, rather than a guide for what a SLA is.

### 3.2 Group Work Reflection & Lessons Learned

Looking back, we feel that many of the challenges that were presented to us throughout the project were challenges that we had already met in our previous semester. This included the usage of continuous integration, continuous delivery and database management, and as such, we feel that instead of learning new skills, our primary gain was further understanding and experience with these tools.

Since we had already tried working on a project hosted on a DigitalOcean server before, we instead decided to make use of Amazon Web Services, being that AWS is such a popular hosting choice in the industry. While the experience gained feels quite valuable, this approach was also significantly more expensive. While we do not regret it, the cost is high enough to where we likely will not use AWS for a school project again.

Likely the most important things we learned throughout the project could be best described threefold: 

- We learned how to use AWS, which is experience that will likely be beneficial in the future. 
- We gained experience with working with a large application, and the maintenance of said application on a live server.
- We encountered issues which made it clearer for us that we should put more focus on more expansive test scenarios. Properly stress-testing our system could have allowed us to identify several severe bottlenecks before they became an issue after our application went live.

## Conclusion

In the end, we feel that this project not only allowed us to improve our skills with technologies we already were familiar with, but also allowed us to expand our horizons by exploring alternatives to the tools we were otherwise provided. Although we also encountered numerous difficulties throughout the project, we feel that we came away from them with valuable lessons learned, as well as a deeper knowledge of things we still can improve on in the future.