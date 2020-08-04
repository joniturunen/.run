# Docker notes

> 4.8.2020 /JT

These are mostly from talks/podcasts by [Bret Fischer](https://www.bretfisher.com/about/).

## About Security

[Check this list](https://github.com/BretFisher/ama/issues/17)

### Docker bench security

Check security and other (apparmor etc)

[GitHub repo of Docker Bench Security](https://github.com/docker/docker-bench-security)

### Check USER privileges for running containers 

Docker tends to run programming language specific containers as root if not taken care of. Chekc node.js etc. for dockerfiles about users they create and use them.
Bret had talk about Node.js in DockerCon19 and has [example in GitHub](https://github.com/BretFisher/dockercon19/blob/master/1.Dockerfile) about using `USER`. 

__Note__ that you have to use chown in `COPY` and `RUN mkdir` commands to take care of file permissions for that selected user. 

### Shift left sec; With scanning

Code repo scanning, Image scanning. 

- AquaSec [aquasec](https://www.aquasec.com) - **Trivy**
- Snyk - [snyk.io](https://snyk.io)

Check the Bret's list for more. 


