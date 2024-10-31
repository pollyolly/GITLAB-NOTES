### GITLAB-NOTES
#### Gitlab CI/CD
Project Directory
```vim
$cd project_application/
$touch projet_application/.gitlab-ci.yml
```
.gitlab-ci.yml
```vim
variables: #GLOBAL VARIABLES ACCESSIBLE INSIDE JOB
  IMAGE_NAME: nanajanashia/demo-app
  IMAGE_TAG: python-app-1.0

stages: #Organize JOB execution from top (test) to bottom (deploy)
  - test   #first to execute; test stage
  - build  #second to execute; build stage
  - deploy #third to execute; deploy stage
 #- testonly; Add more stages here

run_tests: #job name; #any name of the job
  stage: test #test stage
  image: python:3.9-slim-buster #Docker image name
  before_script: #This scrip is executed before the SCRIPT below
    #update first the server
    - apt-get update && apt-get install make
  script: #script to execute
    - make test
    - mkdir test_folder
    - touch test_folder/touch test.md 
   #- #More script here
  artifacts: #Keep the files and do not delete after execution
    path: #path to the folder to keep
      - test_folder/


build_image: #Job name
  stage: build #Build stage
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  variables: #Local variables
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    #variables $REGISTRY_USER and $REGISTRY_PASS created in
    #https://gitlab.com/johnmarkroco05/suite-crm/-/settings/ci_cd
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASS 
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG


deploy:
  stage: deploy #deploy stage
  before_script:
    - chmod 400 $SSH_KEY
  script:
    #Login to server via SSH and Login to Docker then run Docker
    #StrictHostKeyChecking=no, auto accept any host keys no more prompts
    - ssh -o StrictHostKeyChecking=no -i $SSH_KEY root@161.35.223.117 "
        docker login -u $REGISTRY_USER -p $REGISTRY_PASS &&
        docker ps -aq | xargs docker stop | xargs docker rm &&
        docker run -d -p 5000:5000 $IMAGE_NAME:$IMAGE_TAG"
```
### Reference

[GitLab CI/CD Pipeline Tutorial for Beginners (2024)](https://www.youtube.com/watch?v=z7nLsJvEyMY)

[GitLab CI CD Tutorial for Beginners [Crash Course]](https://www.youtube.com/watch?v=qP8kir2GUgo)
