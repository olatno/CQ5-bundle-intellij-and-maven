CQ5-bundle-intellij-and-maven
How to setup, build and auto deployed cq5 bundle with maven and Intellij idea


Before you diving in, please note the following:

1. I have written this post to remind myself of the process. My main motive behind this approach is the idea of separating business logic (jave code) from view (jsp) and using my favourite IDE to build CQ5 bundles. 

2. I have tested and deployed cq5 package into cq5.5 instance using followings:


	a. cq5.5

	b. maven 3.0.5

	c. Intellij IDEA 12.1.3

3. I used difference sources in order to achieve this.

	a. Using Vault to checkout project as specify in this article
		http://dev.day.com/docs/en/crx/current/how_to/how_to_use_the_vlttool.html

	b. Developing cq5 with Maven 
		http://dev.day.com/docs/en/cq/5-5/developing/developmenttools/developing-with-maven.html

	c. Managing Packages Using Maven
		http://dev.day.com/docs/en/cq/5-5/core/how_to/how_to_use_the_vlttool/vlt-mavenplugin.html

	e. Developing cq5 with Eclipse
		http://dev.day.com/docs/en/cq/5-5/developing/developmenttools/developing_with_eclipse.html


 4. How to go about these step by step. In this steps, we are going do the following:
	-Install and configure vault
	-Set up project structure in intellij, which include creating Main Project and 3 Project Modules
	-Update pom.xml for Main Project and the 3 Project Modules
	-Create and update java class
	-Create and update jsp file
	-Create and update filter.xml file
	-Create cq5 project in CDRXE and checkout the project into intellij idea

	a. Install vault tool, if you dont know how follow the example from day link
		http://dev.day.com/docs/en/cq/current/developing/developmenttools/developing_with_eclipse.html#Installing%20FileVault%20(VLT)

	b.Set up project structure in intellij by taking the following steps. What will are doing here is creating a single project with 3 project modules. The file hierarchy is
		myprojectname (hold project reactor to co-ordinate build process in one goal)
			myprojectname/parent (hold share pom.xml)
			myprojectname/core (hold our busness logic)
			myprojectname/ui (hold the view, jsp)

         we use Intellij idea to create the above project structure as follows.

		In intellij (if intellij idea already install), click on File =>New Project. In the New Project window, select maven module, enter project name, opportunity to change project location or take default and opportunity to add project SDK or take default. Click on next to next window and click on finish. WARN - Dont create from archtype just take default and click on finish. Open the myprojectname/pom.xml and copy and paste the Maven rector:

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId><!-- {groupId} --></groupId>
    <artifactId>reactor</artifactId>
    <packaging>pom</packaging>
    <version><!-- {version} --></version>
    <modules>
        <module>app</module>
        <module>parent</module>
        <module>core</module>   
    </modules>
    <name>Sample Project Reactor</name>

    <!-- settings for local deployment -->
  <properties>
        <depl.user><!-- {local instance user (e.g. admin)} --></depl.user>
        <depl.password><!-- {local instance password (e.g. admin)} --></depl.password>
        <depl.host>localhost</depl.host>
        <depl.port><!-- {local instance port (e.g. 4502)} --></depl.port>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.day.cq</groupId>
                <artifactId>cq-quickstart-product-dependencies</artifactId>
                <version>5.5.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

	<!-- Adobe Maven Repository allows the download CQ/CRX related artifacts -->
	<!-- Might not work for some artifact, that is, you have to download and create repository manully -->
    <profiles>
        <profile>
            <id>adobe-public</id>

            <activation>
                <activeByDefault>false</activeByDefault>
            </activation>

            <properties>
                <releaseRepository-Id>adobe-public-releases</releaseRepository-Id>
                <releaseRepository-Name>Adobe Public Releases</releaseRepository-Name>
                <releaseRepository-URL>http://repo.adobe.com/nexus/content/groups/public</releaseRepository-URL>
            </properties>

            <repositories>
                <repository>
                    <id>adobe-public-releases</id>
                    <name>Adobe Public Repository</name>
                    <url>http://repo.adobe.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>never</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>adobe-public-releases</id>
                    <name>Adobe Public Repository</name>
                    <url>http://repo.adobe.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>never</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
</project> 

	c. Create project module underneath your main project which was created above by following this step:

		Click on File => New Module, highlight Maven module and name it parent. Click next to new window, accept default and click on finish.

	   Open myproject/parent/pom.xml file and save Maven decriptor:

	   <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId><!-- {groupId} --></groupId>
        <artifactId>reactor</artifactId>
        <version><!-- {version} --></version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <artifactId>parent</artifactId>
    <packaging>pom</packaging>
    <name>Sample Parent Project</name>
    <properties>
        <file.encoding>utf-8</file.encoding>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.felix</groupId>
                    <artifactId>maven-bundle-plugin</artifactId>
                    <version>2.1.0</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.felix</groupId>
                    <artifactId>maven-scr-plugin</artifactId>
                    <version>1.7.2</version>
                    <extensions>true</extensions>
                    <executions>
                        <execution>
                            <id>scr</id>
                            <goals>
                                <goal>scr</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <!-- Sling plugin dependency and config -->
                <plugin>
                    <groupId>org.apache.sling</groupId>
                    <artifactId>maven-sling-plugin</artifactId>
                    <version>2.1.0</version>
                    <extensions>true</extensions>
                    <configuration>
                        <usePut>true</usePut>
                        <user>${depl.user}</user>
                        <password>${depl.password}</password>
                        <slingUrl>http://${depl.host}:${depl.port}/cq5author/crx/repository/crx.default</slingUrl>
                        <slingUrlSuffix>/apps/hoc/install/</slingUrlSuffix>
                    </configuration>
                </plugin>
                <!-- VLT plugin dependency and config -->
                <plugin>
                    <!--<groupId>com.day.jcr.vault</groupId>
                 <artifactId>maven-vault-plugin</artifactId>
                 <version>0.0.10</version> -->
                    <groupId>com.day.jcr.vault</groupId>
                    <artifactId>content-package-maven-plugin</artifactId>
                    <version>0.0.20</version>
                    <extensions>true</extensions>
                    <configuration>
                        <group>Sample</group>
                        <requiresRoot>true</requiresRoot>
                        <install>true</install>
                        <verbose>true</verbose>
                        <packageFile>${project.build.directory}/${project.artifactId}-${project.version}.zip</packageFile>
                        <version>${project.version}</version>
                        <properties>
                            <acHandling>overwrite</acHandling>
                        </properties>
                        <userId>${depl.user}</userId>
                        <password>${depl.password}</password>
                        <targetURL>http://${depl.host}:${depl.port}/crx/packmgr/service.jsp</targetURL>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
    <dependencyManagement>

        <dependencies>
            <!-- CQ Dependencies -->
            <!-- OSGI Dependencies -->
            <dependency>
                <groupId>org.apache.felix</groupId>
                <artifactId>org.apache.felix.scr.annotations</artifactId>
                <version>1.6.0</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.osgi</groupId>
                <artifactId>org.osgi.compendium</artifactId>
                <version>4.1.0</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.osgi</groupId>
                <artifactId>org.osgi.core</artifactId>
                <version>4.1.0</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.sling</groupId>
                <artifactId>org.apache.sling.api</artifactId>
                <version>2.4.2</version>
                <scope>provided</scope>
            </dependency>

            <!-- Test dependencies -->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.8.2</version>
                <scope>provided</scope>
            </dependency>
            <!-- add additional dependencies as required -->
            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>javax.servlet.jsp-api</artifactId>
                <version>2.2.1</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>jsp-api</artifactId>
                <version>2.1</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>3.0.1</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

	d. Create project module underneath your main project which was created above by following this step:

		Click on File => New Module, highlight Maven module and name it core. Click next to new window, accept default and click on finish.
	
		Open myproject/core/pom.xml file and save Maven decriptor:

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId><!-- {groupId} --></groupId>
        <artifactId>parent</artifactId>
        <relativePath>../parent/pom.xml</relativePath>
        <version><!-- {version} --></version>
    </parent>

    <artifactId>core</artifactId>
    <packaging>bundle</packaging>

    <name>Sample Core Bundle</name>

    <dependencies>
        <dependency>
            <groupId>com.day.cq.wcm</groupId>
            <artifactId>cq-wcm-api</artifactId>
            <version>5.5.0</version>

        </dependency>
        <dependency>
            <groupId>org.apache.sling</groupId>
            <artifactId>org.apache.sling.api</artifactId>
            <version>2.2.0</version>

        </dependency>
        <dependency>
            <groupId>com.day.cq</groupId>
            <artifactId>cq-commons</artifactId>
             <version>5.5.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
             <groupId>com.day.cq.wcm</groupId>
             <artifactId>cq-wcm-core</artifactId>
            <scope>provided</scope>
             <version>5.5.0</version>
         </dependency>
        <!-- add additional dependencies here -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-scr-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-bundle-plugin</artifactId>
                <extensions>true</extensions>
                <configuration>
                    <instructions>
                        <Bundle-Category>hoc</Bundle-Category>
                        <Embed-Directory>OSGI-INF/lib</Embed-Directory>
                        <Include-Resource>{maven-resources}</Include-Resource>
                        <Export-Package>com.bwin.cq5project.*;version=${project.version}</Export-Package>
                    </instructions>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <!-- define deployment steps when installPackages profile is selected -->

        <profile>
            <id>installPackages</id>
            <activation>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.sling</groupId>
                        <artifactId>maven-sling-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>generate-adapter-metadata</id>
                                <phase>process-classes</phase>
                                <goals>
                                    <goal>generate-adapter-metadata</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>install-bundle</id>
                                <goals>
                                    <goal>install</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

</project>

	Create a java package under core/src/main/java by right click java node and go to New => Package in the popup window enter package name.

	Right click on the newly created package, got to New => Java class in the popup window name the new class TestContentPath.
	
	Go to core/src/main/java/yourpackagename/TestContentpath, copy and paste the java code:

	public class TestContentpath {

    		private Page pagePath;

		public TestContentpath(Page pagePath){
        		this.pagePath = pagePath;
    		}

    		public String getContentPaths() {

        		String pathName = pagePath.getPath();

        		return pathName != null ? pathName : "--empty--";
    		}
	}

	e. Create project module underneath your main project which was first above by following this step:

		Click on File => New Module, highlight Maven module and name it app. Click next to new window, accept default and click on finish.

	Open myproject/app/pom.xml file and save Maven decriptor:

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId><!-- {groupId} --></groupId>
        <artifactId>parent</artifactId>
        <relativePath>../parent/pom.xml</relativePath>
        <version><!-- {version} --></version>
    </parent>

    <artifactId>app</artifactId>
    <packaging>content-package</packaging>

    <name>Sample Application package</name>

    <dependencies>
        <dependency>
            <groupId><!-- {groupId} --></groupId>
            <artifactId>parent</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- add additional dependencies -->
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>${basedir}/src/main/content/META-INF</directory>
                <targetPath>../vault-work/META-INF</targetPath>

            </resource>

            <resource>

                <directory>${basedir}/src/main/content/jcr_root</directory>

                <excludes>
                    <exclude>**/.vlt</exclude>
                    <exclude>**/.vltignore</exclude>
                    <exclude>**/*.iml</exclude>
                    <exclude>**/.classpath</exclude>
                    <exclude>**/.project</exclude>
                    <exclude>**/.DS_Store</exclude>
                    <exclude>**/target/**</exclude>
                    <exclude>libs/**</exclude>
                </excludes>
            </resource>
        </resources>

        <plugins>

            <plugin>
                <groupId>com.day.jcr.vault</groupId>
                <artifactId>content-package-maven-plugin</artifactId>
                <version>0.0.20</version>
                <extensions>true</extensions>
                <configuration>
                    <group>Sample</group>
                    <requiresRoot>true</requiresRoot>

                    <install>true</install>
                    <verbose>true</verbose>

                    <packageFile>${project.build.directory}/${project.artifactId}-${project.version}.zip</packageFile>

                    <version>${project.version}</version>
                    <properties>
                        <acHandling>overwrite</acHandling>
                    </properties>

                    <embeddeds>
                        <embedded>
                            <groupId><!-- {groupId} --></groupId>
                            <artifactId>core</artifactId>
                            <target><!-- {install path in the repository (e.g. /apps/myproject/install)} --></target>
                        </embedded>
                    </embeddeds>
                </configuration>
            </plugin>

        </plugins>
    </build>

    <profiles>
        <profile>
            <id>installPackages</id>
            <activation>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>com.day.jcr.vault</groupId>
                        <artifactId>content-package-maven-plugin</artifactId>
                        <version>0.0.20</version>
                        <executions>
                            <execution>
                                <id>install-package</id>
                                <goals>
                                    <goal>install</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

</project>

	Create a directory and name it "content" in app/src/main directory. Create a package and name it "vault" in ui/src/main/content/META-INF directory. In the vault directory create a file name filter.xml and copy and paste 

	<?xml version="1.0" encoding="UTF-8"?>
	<!-- Defines which repository items are generally included -->
		<workspaceFilter version="1.0">
    			 <filter root="<!-- {root folder of app files (e.g. /apps/myproject)} -->">
			<filter root="/libs/foundation" />


		</workspaceFilter>


	f. Open command and cd to cq5talib/ui/src/main/content directory and issue this vault command "vlt –-credentials admin:admin co http://localhost:4502/crx" . If command is successful, the file from cq5 instance project created will be available in your intellij. Once cq5 project is checkout to intellij, move /libs/foundation to components/contentpage and delete <filter root="/libs/foundation" /> in your filter.xml

<!--vlt -v --credentials admin:admin co --force http://localhost:4502/cq5author/crx/server/crx.default-->

On the left hand side of your intellij bring up the Maven Project, which will appear on third column of the IDE. In the maven project, expand Profiles and check adobe-public and intsallPackages. Go to Sample Project Reactor, expand Lifesysle select clean and click on the green arrow. select install and click on green arrow to package and auto install cq5 project. 

Below were the outputs from my intellij console. 
<!-- -->
"C:\Program Files\Java\jdk1.6.0_24\bin\java" -Dmaven.home=C:\apache-maven-3.0.5\apache-maven\src -Dclassworlds.conf=C:\apache-maven-3.0.5\apache-maven\src\bin\m2.conf -Didea.launcher.port=7534 "-Didea.launcher.bin.path=C:\Users\OlatunjiN\AppData\Roaming\JetBrains\IntelliJ IDEA 12.1.3\bin" -Dfile.encoding=UTF-8 -classpath "C:\apache-maven-3.0.5\apache-maven\src\boot\plexus-classworlds-2.4.jar;C:\Users\OlatunjiN\AppData\Roaming\JetBrains\IntelliJ IDEA 12.1.3\lib\idea_rt.jar" com.intellij.rt.execution.application.AppMain org.codehaus.classworlds.Launcher --fail-fast --strict-checksums install -P adobe-public,installPackages
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] Sample Project Reactor
[INFO] Sample Parent Project
[INFO] Sample Core Bundle
[INFO] Sample Application package
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Sample Project Reactor 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-install-plugin:2.3.1:install (default-install) @ reactor ---
[INFO] Installing C:\Users\OlatunjiN\cq5project\pom.xml to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\reactor\1.0-SNAPSHOT\reactor-1.0-SNAPSHOT.pom
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Sample Parent Project 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-install-plugin:2.3.1:install (default-install) @ parent ---
[INFO] Installing C:\Users\OlatunjiN\cq5project\parent\pom.xml to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\parent\1.0-SNAPSHOT\parent-1.0-SNAPSHOT.pom
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Sample Core Bundle 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ core ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ core ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-scr-plugin:1.7.2:scr (scr) @ core ---
[INFO] 
[INFO] --- maven-sling-plugin:2.1.0:generate-adapter-metadata (generate-adapter-metadata) @ core ---
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ core ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory C:\Users\OlatunjiN\cq5project\core\src\test\resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ core ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:9c6abc2:test (default-test) @ core ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-bundle-plugin:2.1.0:bundle (default-bundle) @ core ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ core ---
[INFO] Installing C:\Users\OlatunjiN\cq5project\core\target\core-1.0-SNAPSHOT.jar to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\core\1.0-SNAPSHOT\core-1.0-SNAPSHOT.jar
[INFO] Installing C:\Users\OlatunjiN\cq5project\core\pom.xml to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\core\1.0-SNAPSHOT\core-1.0-SNAPSHOT.pom
[INFO] 
[INFO] --- maven-bundle-plugin:2.1.0:install (default-install) @ core ---
[INFO] Installing com/day/cq5project/core/1.0-SNAPSHOT/core-1.0-SNAPSHOT.jar
[INFO] Writing OBR metadata
[INFO] 
[INFO] --- maven-sling-plugin:2.1.0:install (install-bundle) @ core ---
[INFO] Installing Bundle com.day.cq5project.core(C:\Users\OlatunjiN\cq5project\core\target\core-1.0-SNAPSHOT.jar) to http://localhost:8099/cq5author/crx/repository/crx.default/apps/hoc/install/ via PUT
[ERROR] Installation failed, cause: Conflict
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Sample Application package 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ app ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 3 resources to ../vault-work/META-INF
[INFO] Copying 5 resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ app ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- content-package-maven-plugin:0.0.20:check-signature (default-check-signature) @ app ---
[INFO] No signature defined. Skipping signature check.
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ app ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory C:\Users\OlatunjiN\cq5project\app\src\test\resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ app ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:9c6abc2:test (default-test) @ app ---
[INFO] No tests to run.
[INFO] 
[INFO] --- content-package-maven-plugin:0.0.20:package (default-package) @ app ---
[INFO] Embedding --- Embedded: groupId=com.day.cq5project,artifactId=core,filter=false,excludeTransitive=false,target=/apps/cq5project/install/ ---
[INFO] Embedding com.day.cq5project:core:jar:1.0-SNAPSHOT (from C:\Users\OlatunjiN\cq5project\core\target\core-1.0-SNAPSHOT.jar) -> jcr_root/apps/cq5project/install/core-1.0-SNAPSHOT.jar
[INFO] Building content-package: C:\Users\OlatunjiN\cq5project\app\target\app-1.0-SNAPSHOT.zip
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ app ---
[INFO] Installing C:\Users\OlatunjiN\cq5project\app\target\app-1.0-SNAPSHOT.zip to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\app\1.0-SNAPSHOT\app-1.0-SNAPSHOT.zip
[INFO] Installing C:\Users\OlatunjiN\cq5project\app\pom.xml to C:\Users\OlatunjiN\.m2\repository\com\day\cq5project\app\1.0-SNAPSHOT\app-1.0-SNAPSHOT.pom
[INFO] 
[INFO] --- content-package-maven-plugin:0.0.20:install (install-package) @ app ---
[INFO] Installing app (C:\Users\OlatunjiN\cq5project\app\target\app-1.0-SNAPSHOT.zip) to http://localhost:8099/cq5author/crx/packmgr/service.jsp
[INFO] app, 1.0-SNAPSHOT (17440 bytes)
[INFO]     Created : Thu, 13 Jun 2013 15:42:36 +0100 by olatunjin
[INFO]     Modified: -
[INFO]     Unpacked: -
[INFO] 
[INFO] Installing content...
Collecting import information... 
Installing node types... 
Installing privileges... 
Importing content... 
- /apps/cq5project/components/contentpage
- /apps/cq5project/components/contentpage/contentpage.jsp
- /apps/cq5project/templates/contentpage
- /apps/cq5project/templates/contentpage/jcr:content
U /apps/cq5project/install/core-1.0-SNAPSHOT.jar/jcr:content
U /apps/cq5project/install/core-1.0-SNAPSHOT.jar/jcr:content/jcr:data
saving approx 2 nodes... 
Package imported. 
Package installed in 22ms.
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] Sample Project Reactor ............................ SUCCESS [0.148s]
[INFO] Sample Parent Project ............................. SUCCESS [0.008s]
[INFO] Sample Core Bundle ................................ SUCCESS [1.330s]
[INFO] Sample Application package ........................ SUCCESS [0.692s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.676s
[INFO] Finished at: Thu Jun 13 15:42:36 BST 2013
[INFO] Final Memory: 13M/244M
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0 
<!-- -->
