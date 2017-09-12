# Introduction
There is an instruction how to produce project to [Maven Central][0] via [Sonatype][1].
## Prepare
If You do it in first time then need prepare your machine and create issue on [Sonatype][1]
### Create ticket on Sonatype JIRA
For publish project to [Maven Central][0] need [Sign up][2] or [Sign in][3] on Sonatype JIRA.
After press `Create` button on top the main page. 

There is an example how must look issue:
![screenshot of sample](/maven/resources/sonatype/sonatype_create_issue.png "Create Issue example")

After a while, an employee Sonatype create a set of repositories for You and close ticket.

### Prepare pom.xml
There is minimum for `pom.xml` file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>${GROUP_ID}</groupId>
    <artifactId>${ARTIFACT_ID}</artifactId>
    <version>${PROJECT_VERION}-SANPSHOT</version>
    <packaging>jar</packaging>

    <name>${PROJECT_NAME}</name>
    <description>${PROJECT_DESCRIPTION}</description>
    <url>${PROGECT_SITE}</url>

    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
        </license>
    </licenses>

    <scm>
        <!--<url>https://github.com/git_name/project_name</url>-->
        <url>${PROJECT_URL}</url>
        <!--<connection>scm:git:https://github.com/git_name/project_name.git</connection>-->
        <connection>scm:${PROJECT_CONNECTION_URL}</connection>
        <!--<developerConnection>scm:git:https://github.com/git_name/project_name.git</developerConnection>-->
        <developerConnection>scm:${PROJECT_CONNECTION_URL}</developerConnection>
    </scm>

    <developers>
        <developer>
            <id>owner</id>
            <name>${DEVELOPER_NAME}</name>
            <email>${DEVELOPER_EMAIL}</email>
            <timezone>UTC+3</timezone>
        </developer>
    </developers>

    <parent>
        <groupId>org.sonatype.oss</groupId>
        <artifactId>oss-parent</artifactId>
        <version>7</version>
    </parent>

    <dependencies>
    </dependencies>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <profiles>
        <profile>
            <id>sign-artifacts</id>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <configuration>
                        </configuration>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.0.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>package</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <version>2.10.4</version>
                <executions>
                    <execution>
                        <id>attach-javadocs</id>
                        <phase>package</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### Configure GnuPg
GnuPg is required to sign artifacts at the time of sending to Sonatype Repository.
On Linux GnuPg contains in the standart repositories. For Windows you can [download][4] from official website.

After installing GnuPg need generate and send key to server:
  * Generate key:
  
    `$ gpg --gen-key`
  * Show keys list:
  
    `$ gpg --list-keys`
    
    After this command you wiil see some like:
      ```
      pub   2048R/1A89FE67 2013-02-26
      uid                  Vasiliy Pupkin <vp@example.com>
      sub   2048R/D9725D51 2013-02-26
       ```
  * Send your public key to server:
  
    `$ gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 1A89FE67`
    
    Where `1A89FE67` is short print from your public key.
  
### Configure Maven settings
Add to Maven `settings.xml` file:

  * Sonatype snapshots server
    ```xml
      <servers>
        ...
            <server>
              <id>sonatype-nexus-snapshots</id>
                <username>${SONATYPE_NAME}</username>
                <password>${SONATYPE_PASSWORD}</password>
            </server>
        ...
      </servers>	
    ```
  * Sonatype staging server
    ```xml
      <servers>
        ...
            <server>
              <id>sonatype-nexus-staging</id>
                <username>${SONATYPE_NAME}</username>
                <password>${SONATYPE_PASSWORD}</password>
            </server>
        ...
      </servers>	
    ```
  * Git server
    ```xml
      <servers>
        ...
            <server>
                <id>github.com</id>
                <username>${GITHUB_NAME}</username>
                <password>${GITHUB_PASSWORD}</password>
            </server>
        ...
   	 	</servers>	
    ```
  * Gpg profile
    ```xml
      <profiles>
        ...
            <profile>
                <id>gpg-passphrase</id>
              <activation>
                      <activeByDefault>true</activeByDefault>
                </activation>
                <properties>
                      <gpg.passphrase>${GNUPG_PASSWORD}</gpg.passphrase>
                </properties>
            </profile>
        ...
      </profiles>

    ```
### Release
For release project to [Maven Central][0] needs make 2 things:
#### Deploy
For deploying your project to [Sonatype][1]:
  * `$ mvn deploy`
  * `$ mvn release:prepare`
  * `$ mvn release:perform`
#### Release to Maven
  1. Go to [Soantype Repository][5]
  2. Login as on [Sonatype][1]
  3. Open `Staging Repositories` in `Build Promotion` Tab
  4. Select your staging repository (your_artifact_id-10XX)
  5. Click `Close`
  6. Click `Release`
  
![screenshot of sample](/maven/resources/sonatype/sonatype_repository.png "Repository example")

If You release in first time then need left comment for employees on [Sonatype][1] on created issue. After some time
you will get access to sync with [Maven Central][0]. This need make just once.

[0]: https://search.maven.org/
[1]: https://issues.sonatype.org/
[2]: https://issues.sonatype.org/secure/Signup!default.jspa
[3]: https://issues.sonatype.org/login.jsp?os_destination=%2Fsecure%2FSignup%21default.jspa
[4]: https://www.gpg4win.org/
[5]: https://oss.sonatype.org/
