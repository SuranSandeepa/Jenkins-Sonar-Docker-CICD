# Jenkins CICD - SonarQube, Docker, GitHub Webhooks on AWS

This project sets up a CI/CD pipeline using Jenkins, SonarQube, Docker, and GitHub Webhooks. The infrastructure is hosted on AWS with three EC2 instances.

This pipeline automates:
- Code analysis using SonarQube.
- Continuous integration via Jenkins.
- Dockerized deployments.
- Automated triggers via GitHub Webhooks.

**Infrastructure Details**:
- 3 EC2 Instances (t2.medium, Ubuntu)
  - **Jenkins Server**: CI/CD tool
  - **SonarQube Server**: Code quality analysis
  - **Docker Server**: Dockerized applications
