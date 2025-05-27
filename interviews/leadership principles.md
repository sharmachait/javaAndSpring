## Customer Obsession
before writing the code for the tool, i sat down with my manager and the billing team to decide the level of autonomy they required, then we decided that it really mattered to them and required a full fledged solution divorced from the CICD change and release management process
RDL reports in house tool
## Ownership
Durable functions
RDL reports in house tool
Mongo Schema migrator
## Invent and simplify
First start with CICD pipeline RDL reports in house tool. DBA access over the data suorces
## Are right a lot
Mongo Schema migrator
requirement RSA encryption for JWT in the legacy monolithic application for some new data that we get from user service, but Spring versions before Spring Security 5.2 do not have built-in support for RSA encryption for JWT tokens. and we already had a microservice with those ersions that was returning the data to us, so i pointed out that we should do the RSA encryption in that microservice it self
## Learn and be curious
implmented k6 operator districuted testing framework
upgraded our postgres from 15 to 16.8 
we had a script that was being used to grant privileges to users, so in 15 version we were granting postgres role, but thats not possible in 16, so i took the initiative to fix that script granted permission to read create update, and created functions to drop things and gave execute permissions on that functions, and those are being used by all other teams now cross functionally
## Insist on the highest standards
new project bare bones cicd, dockerize and update the image in the helm chart, no code coverage report
i took the initiative to implement the code coverage in the CICD while it was not of the highest priority for the deliverables, infact it only hinders the deadline, but is necessary for high standard products
jacoco maven surefire
## Think big
before writing the code for the tool, i sat down with my manager and the billing team to decide the level of autonomy they required, then we decided that it really mattered to them and required a full fledged solution divorced from the CICD change and release management process
RDL reports in house tool
## Bias for action
mongo schema migrator, i could have asked permission for it pitched the idea get some story points assigned for it first, but i implemented the solution and then pitched it. because it wouldnt hurt, and i wanted to do it none the less
## Furgality
we were using microsoft hosted agents for CICD, i took the initiative too reduce our teams expenditure by deploying agents on the under utilized non prod clusters, but it wasnt so easy, because we were using docker commands to build our applications but kubernetes cluster dont have dockerd, so even DIND and docker sock binding wouldnt work, so we made a custom base image for the agents that had the docker binaries on it and used the microsoft provided agent binaries on top of that. agents are not very expensive, but we did save some money since our non prod clusters were severely under utilized 
## Earn trust
leaders are attentive, speak candidly and and are self-critical
## Dive Deep
sometimes we need the senior engineer to junior tasks
after issues are mitigated we can dive deep even in hind sight
postgresql
## Have backbone, disagree and commit
Mongo Schema migrator    
we were asked to create a job that would run scripts checked in. i proposed a full on migration frame work using migrate-mongo. but the idea was not picked up. the issue was that it was for a new project and right now we were only required to create indexes in the database. In mongo its fine to create collections and indexes even if they already exists. it is handled and no error is thrown but that not how the azure cosmos db work, which i soon figured out, so if a script ran twice, which the solution i was asked to implement did, it ran all the scripts again, azure cosmos db threw an error. so i read the scripts that were being checked in and sure enough there were no checks for existing collections and indexes. so a schema migrator framework would help as it wouldn't let a script thats already ran run again. so i took it on my own and implemented a migrate-mongo on my own, and instead of simply running the scripts using mongosh in a job pod, i ran the scripts using migrate-mongo
## Deliver Results
vague you know we should do this everything, so i would like to take an example that doesnt fit perfectly but what i learned from it, or how i was able to over come a challenge to do that
sonar qube docker image no comments on PR
## Strive to be Earths Best Employer
leaders create a safer, more productive env. lead with empathy. and care about their employees growth
teach the intern about setting up cicd for fileshares
## Success and Scale Bring Broad Responsibility
leaders create more than they consume and sleep thinking about doing more





Hi, I’m Kanhaiya Bhayana, a Software Engineer with over 2 years of 
experience at Incedo. Currently, I’m working with one of our key clients, 
AssetMark, on their flagship platform, EWM, or e-Wealth Manager. 
This platform empowers financial advisors to efficiently manage households, 
clients, and related data.

During my tenure, I’ve had the opportunity to contribute to AssetMark’s transition from a monolithic to a microservices architecture. One of my significant achievements was working on the Business Assessment Tool (BAT) module, which is now live and successfully serving its purpose. This project allowed me to strengthen my skills in software design and microservices implementation.

Presently, I’m working on another crucial feature—the Client Section 
module—which involves enabling advisors to organize client preferences 
into groups.