# Jenkins

## How artifacts get stored in Nexus.
Steps for Configuring Nexus with Maven in Jenkins DSL Pipeline
1. Set Up Nexus Repository:
- Before proceeding with configuring your Maven project, ensure your Nexus repository is properly set up:
- Nexus repository needs to be configured to store your Maven artifacts (like .jar, .war, .pom).
  - Log into Nexus.
  - Create a Maven hosted repository (e.g., `maven-releases`, `maven-snapshots`).
  - For each repository, get the repository URL (you will need this for Jenkins to deploy artifacts).
- `Example Nexus URLs:`
   - `https://nexus.company.com/repository/maven-releases/`
   - `https://nexus.company.com/repository/maven-snapshots/`

2. Configure `pom.xml`
- Your `pom.xml` file tells Maven where to deploy the built artifact (e.g., JAR, WAR, etc.) once the build completes. The main section that you’ll need to add is `distributionManagement.`
  a. Distribution Management in pom.xml:
  - In the pom.xml, specify the Nexus repositories under the distributionManagement section.
  ```xml
  <distributionManagement>
    <!-- Release repository -->
    <repository>
        <id>nexus-releases</id>
        <url>https://nexus.company.com/repository/maven-releases/</url>
    </repository>

    <!-- Snapshot repository -->
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>https://nexus.company.com/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
  ```
 - Explanation:
   - `id`: This should be a unique identifier for each repository, which will also be used in settings.xml for authentication.
   - `url`: The URL pointing to your Nexus repository, which will either be the releases or snapshots repository depending on the version of your artifact.
     
**Note: You must configure both a release repository and a snapshot repository if you plan to deploy both stable and development versions.**
       
3. Configure Credentials in `settings.xml`, which we will get in Jenkins by the plugin `config file provider`
- Since Nexus requires authentication (username and password), Maven needs to be informed of how to authenticate when deploying artifacts. This is done through the `settings.xml` file.
- Manage Jenkins - Manage Files (Plugin - Config File Provider) - here we will get `settings.xml`
- In that setting.xml, we need to provide the "ID" and "username-password" of the Nexus Maven-hosted repository (e.g., `maven-releases`, `maven-snapshots`).
**Note- `maven-releases` for production servers and `maven-snapshots` for lower envirenment server.**
```xml
<servers>
    <server>
        <id>nexus-releases</id> <!-- This must match the id in pom.xml -->
        <username>${env.NEXUS_USERNAME}</username>
        <password>${env.NEXUS_PASSWORD}</password>
    </server>
    <server>
        <id>nexus-snapshots</id> <!-- This must match the id in pom.xml -->
        <username>${env.NEXUS_USERNAME}</username>
        <password>${env.NEXUS_PASSWORD}</password>
    </server>
</servers>
```
4. Create a stage for artifact deployment. `mvn deploy`



