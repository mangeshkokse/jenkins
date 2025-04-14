# jenkins
## Stage-by-Stage Summary
```mathmetica

- Stage	                              = What it does
- Git                                 = Checkout	Clones the project from GitHub (main branch) using credentials
- Compile	                            = Compiles the Java code using Maven
- Test	                              = Runs the project's unit tests
- Trivy File System Scan	            = Scans the project directory for vulnerabilities (Trivy FS scan)
- SonarQube Analysis	                = Performs static code analysis with SonarQube
- Quality Gate	                      = Waits for SonarQube’s quality gate result (pass/fail)
- Build	                              = Packages the project into a .jar using Maven
- Publish Artifacts to Nexus	        = Publish built artifacts to Nexus repo using Maven
- Build and Tag Docker Image	        = Creates a Docker image for the app and tags it
- Docker Image Scan                   = Runs Trivy security scan on the Docker image
- Push Docker Image	                  = Pushes the image to DockerHub
- Deploy to Kubernetes	              = Deploys the app using kubectl apply to a K8s cluster
```
