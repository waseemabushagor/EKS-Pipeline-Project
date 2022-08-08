<!-- PROJECT TITLE -->
<div align="center">
  <h3 align="center">Pipeline Project</h3>
</div>



<!-- TABLE OF CONTENTS -->
<div align="center">
  <details>
    <summary>Table of Contents</summary>
    <ol>
      <li><a href="#about">About The Project</a></li>
      <li><a href="#built-with">Built With</a></li>
      <li><a href="#getting-started-and-assumptions">Getting Started and Assumptions</a></li>
      <li><a href="#prerequisites">Prerequisites</a></li>
      <li><a href="#installation">Installation</a></li>
      <li><a href="#usage">Usage</a></li>
      <li><a href="#roadmap">Roadmap</a></li>
      <li><a href="#license">License</a></li>
      <li><a href="#acknowledgments">Acknowledgments</a></li>
    </ol>
  </details>
</div>


<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://github.com/waseemabushagor/pipeline/blob/master/images/screenshot.PNG)

Key Concepts:
* This project uses a CI/CD pipeline which extensively focuses on code quality.
* This project leverages the power of ec2(s) and EKS to support the long-term viability of the deployment.
* This project has a blue/green deployment strategy which focuses on a clean, clear, and consistent user experience without hindering development.

This project listens to a webhook from github, sent on push(es) to an Amazon Web Services (AWS) Elastic Compute Cloud (ec2) instance. This t2.medium ec2 instance runs Jenkins which controls a Continuous Integration / Continuous Deployment (CI/CD) pipeline. The t2.medium was necessary for the overall health of the Jenkins instance, whereas the cheaper t2.micros can often hang on pipeline operations and become unreachable. This Jenkins pipeline runs a series of built-in tests, including Sonarqube code linting, code smells, vulnerability scans, and bug analyses. Jenkins then builds a docker image of the project and pushes it to Docker Hub. This pipeline then creates a deployment, ingress, and service to a remote Elastic Kubernetes Service (EKS) cluster through a single YAML file. To recreate this, your own EKS instance's credentials need to be properly provisioned for a service account in AWS Identity and Access Management (IAM) roles & users (eks* and cloudaccess* are recommended for creation + deployment configurations like ours). Finally, the code is deployed in a blue/green fashion, meaning that by changing the deployment but not the service and ingress we can change the outward appearance of our app while having very different internal functions.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

* [![Docker][Docker.com]][Docker-url]
* [![DockerHub][DockerHub.com]][DockerHub-url]
* [![Jenkins][Jenkins.io]][Jenkins-url]
* [![GitHub][GitHub.com]][GitHub-url]
* [![Kubernetes][Kubernetes.io]][Kubernetes-url]

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->

### Prerequisites

Once your ec2 instance (t2.medium) is up and running, this user data script of commands will install jenkins, git, docker, give permissions for jenkins to use docker:
  ```sh
  sudo yum update –y
  sudo yum install -y yum-utils
  sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo -y
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  sudo yum upgrade -y
  sudo yum upgrade
  # Add required dependencies for the jenkins package
  sudo yum install java-11-openjdk -y
  sudo yum install jenkins -y
  sudo systemctl daemon-reload
  sudo systemctl enable jenkins
  sudo systemctl start jenkins
  sudo yum install git -y
  sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
  sudo yum install docker-ce docker-ce-cli containerd.io -y
  sudo systemctl start docker
  sudo systemctl enable docker.service
  sudo usermod -a -G docker jenkins
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword

  ```


### Installation

The user account permissions used were maximal to get the target setup. Please don't use these in production, the authors intended use case is exclusively for minimum viable product (MVP) setup only.

1. Create a new user in your AWS IAM dashboard, and select a JSON permissions policy. The policy below contains the _maximum permissions_ necessary for this task. Use them at your own risk (please secure your instances).
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "eks_administrator",
              "Effect": "Allow",
              "Action": [
                  "eks:*"
              ],
              "Resource": "*"
          },
          {
              "Sid": "cloudformation_administrator",
              "Effect": "Allow",
              "Action": [
                  "cloudformation:*"
              ],
              "Resource": "*"
          }
      ]
  }
  ```

2. Set this repository up with a webhook of your jenkins url with the addition of '/github-webhook/'. Reference https://hevodata.com/learn/jenkins-github-webhook/

3. On your AWS IAM dashboard navigate to the user you set up with the custom JSON access policy in step 1 above. Then, in that user's section entitled 'security credentials', click 'create access key'. This will generate a key specific to this user for this use case. Do not lose it, and keep it somewhere safe. 

export AWS_PROFILE=user

4. SSH back into your jenkins instance and in the command line, type 'aws configure'. In the following fields, fill in the user account we just created access keys for above in step 3. This will give your instance permissions to access and remote into your soon-to-exist EKS cluster.

5. Finally, create your EKS cluster using the following commands from the command line of the ec2 instance that jenkins is running on. 
  ```sh
  eksctl create cluster --name pipeline-project --version 1.22 --region us-east-1 --nodegroup-name linux-nodes --node-type t2.micro --nodes 1
  aws eks --region us-east-1 update-kubeconfig --name pipeline-project
  kubectl get all
  ```

6. Now whenever anyone makes a push to the GitHub repository where your webhook is, it will be entered into the CI/CD pipeline, examined, and deployed on your local EKS cluster.
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage

This pipeline is a fault-tolerant implementation of blue/green deployment 
off of a CI/CD pipeline. With a thorough understanding of the Jenkinsfile,
any reasonably experienced developer can fork this repo to examine and
deploy their own code to an EKS cluster after sonarqube evaluates it.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [x] Add Changelog
- [x] Add back to top links
- [ ] Add Additional Templates w/ Examples
- [x] Add "components" document to easily copy & paste sections of the readme

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/2206-devops-batch/ERMS-Project2?style=for-the-badge
[contributors-url]: https://github.com/2206-devops-batch/ERMS-Project2/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/2206-devops-batch/ERMS-Project2?style=for-the-badge
[forks-url]:https://github.com/2206-devops-batch/ERMS-Project2.git
[stars-shield]: https://img.shields.io/github/stars/2206-devops-batch/ERMS-Project2?style=for-the-badge
[stars-url]: https://github.com/2206-devops-batch/ERMS-Project2/stargazers
[issues-shield]: https://img.shields.io/github/issues/2206-devops-batch/ERMS-Project2?style=for-the-badge
[issues-url]: https://github.com/2206-devops-batch/ERMS-Project2/issues
[license-shield]: https://img.shields.io/github/license/2206-devops-batch/ERMS-Project2?style=for-the-badge
[license-url]: https://github.com/2206-devops-batch/ERMS-Project2/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.PNG
[Docker.com]:https://img.shields.io/badge/Docker-white?style=for-the-badge&logo=docker&logoColor=30B2F5
[Docker-url]:https://www.docker.com/
[DockerHub.com]:https://img.shields.io/badge/DockerHub-white?style=for-the-badge&logo=dockerhub&logoColor=30B2F5
[DockerHub-url]:https://hub.docker.com/
[Jenkins.io]:https://img.shields.io/badge/Jenkins-white?style=for-the-badge&logo=jenkins&logoColor=black
[Jenkins-url]:https://www.jenkins.io/
[GitHub.com]:https://img.shields.io/badge/GitHub-white?style=for-the-badge&logo=github&logoColor=black
[GitHub-url]:https://github.com/
[Kubernetes.io]:https://img.shields.io/badge/Kubernetes-white?style=for-the-badge&logo=kubernetes&logoColor=004DFF
[Kubernetes-url]:https://kubernetes.io/
