## Stand up Content
1. CI-CD definition and principles - ELizabeth
2. setup tools jenkins, docker, conneting to GIT basic pipeline : Mwikali
3. advanced pipeline: vm deployment - Hussein


### CI-CD definition and principles

<!-- CI/CD Definition and Principles -->

What is CI/CD?

CI/CD stands for Continuous Integration and Continuous Delivery (or Continuous Deployment). It is a software development practice that automates the process of integrating code, testing it, and deploying it to users. CI/CD helps development teams deliver software faster, with fewer errors and greater reliability.

<!-- Continuous Integration (CI) -->

Definition:

Continuous Integration is the practice of frequently merging code changes into a shared repository. Every time code is added or updated, automated tools build the application and run tests to ensure everything works correctly.
Key Features
    • Developers commit code regularly.
    • Automated builds are triggered after each commit.
    • Automated tests detect bugs early.
    • Developers receive immediate feedback on errors.
Benefits
    • Finds bugs early.
    • Reduces merge conflicts.
    • Improves code quality.
    • Saves development time.

<!-- Continuous Delivery (CD) -->

Definition:
Continuous Delivery is the practice of automatically preparing software for release after it passes all tests. The software is always ready to be deployed, but deployment to production requires manual approval.
Benefits
    • Faster software releases.
    • Reduced deployment risks.
    • Reliable release process.
    • Easy rollback if problems occur.
      
<!-- Continuous Deployment -->

Definition:
Continuous Deployment is similar to Continuous Delivery, except that every successful change is automatically deployed to production without human intervention.
Benefits
    • Very fast releases.
    • Continuous updates for users.
    • Minimal manual work.

<!-- CI/CD Pipeline -->

A CI/CD pipeline is a series of automated steps that move software from development to production.

Typical pipeline:
    1. Developer writes code.
    2. Code is pushed to a Git repository.
    3. The application is automatically built.
    4. Automated tests are run.
    5. Code quality and security checks are performed.
    6. The application is packaged.
    7. It is deployed to testing or staging.
    8. Finally, it is deployed to production.

<!-- Principles of CI/CD -->

1. Frequent Code Integration
Developers should integrate their code regularly instead of waiting until the end of a project.
2. Automation
Building, testing, and deployment should be automated to reduce manual work and errors.
3. Continuous Testing
Every code change should be tested automatically to identify issues early.
4. Fast Feedback
Developers should receive immediate feedback when builds or tests fail.
5. Version Control
All source code, configuration files, and deployment scripts should be stored in a version control system such as Git.
6. Small, Incremental Changes
Developers should make small code changes frequently, making them easier to review, test, and fix.
7. Consistent Deployment
Deployments should follow the same automated process every time to ensure consistency across environments.
8. Monitoring and Improvement
Applications should be monitored after deployment to detect issues and improve future releases.

<!-- Advantages of CI/CD -->

    • Faster software development.
    • Early detection of bugs.
    • Better software quality.
    • Reduced deployment failures.
    • Faster feedback for developers.
    • Improved teamwork and collaboration.
    • More reliable software releases.

<!-- Common CI/CD Tools -->

    • Version Control: Git, GitHub, GitLab
    • CI Tools: Jenkins, GitHub Actions, GitLab CI/CD
    • Build Tools: Maven, Gradle, npm
    • Testing Tools: JUnit, Selenium
    • Containers: Docker
    • Orchestration: Kubernetes

<!-- Summary -->

CI/CD is a modern software development approach that automates the integration, testing, and deployment of applications. Continuous Integration (CI) focuses on regularly integrating and testing code, while Continuous Delivery/Deployment (CD) focuses on automating the release process. By following CI/CD principles, organizations can deliver high-quality software faster, more reliably, and with fewer errors.


### setup tools jenkins, docker, conneting to GIT basic pipeline
// Add content here

### advanced pipeline: vm deployment
// Add content here