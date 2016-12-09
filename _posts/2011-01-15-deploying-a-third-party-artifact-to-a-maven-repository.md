---
layout: post
title: Deploying a third party artifact to a Maven Repository
---
Specifically, I'm attempting today to deploy the Perforce P4Java api to the Sonatype core Maven repository. Sonatype has plenty of documentation on this process, but most of them make the assumption that you are building the artifacts from source, and that you are already familiar with maven. I'm a maven newbie and I obviously don't have the source. Here's a quick rundown of how I did.
My primary resources was actually [this](https://docs.sonatype.org/display/Repository/How+To+Generate+PGP+Signatures+With+Maven) page (A shout out to Brian Jackson for pointing me in the right direction!)

The page mostly covers how to set up a gpg key, which is fairly straight forward and there's plenty of documentation on the internet on how to do it. But before we do anything, we need a working pom.xml. Here's the one I used:

{% highlight xml %}
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.perforce</groupId>
  <artifactId>p4java</artifactId>

<packaging>jar</packaging>
  <name>P4Java Api</name>
  <version>2010.1.269249</version>
  <description>Pure Java api for communicating with Perforce</description>
  <url>http://www.perforce.com</url>
  <developers>
    <developer>
      <id>perforce</id>
      <name>Perforce</name>
      <email>support@perforce.com</email>
    </developer>
  </developers>

<licenses>
<license>
      <name>Copyright (c) 2009, Perforce Software, Inc.  All rights reserved.</name>
      <url>LICENSE.txt</url>
      <distribution>repo</distribution>
    </license>
  </licenses>
  <scm>
   <url>http://www.perforce.com/</url>
  </scm>
  <build>

<plugins>
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-gpg-plugin</artifactId>
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
</project>
{% endhighlight %}

I had to construct this from several examples in sonatype's documentation, as every example they gave was incomplete in one way or another. Once I was given access to com.perforce, I was able to upload to a staging repository, but first I needed to configure maven to authenticate against it. Here's my `~/.m2/settings.xml` file that accomplishes this authentication, since maven apparently isn't able to ask the user for credentials in the event they aren't available.

{% highlight xml %}
<settings>
  <servers>
    <server>
      <id>sonatype-nexus-staging</id>
      <username>rpetti</username>

<password>************</password>
    </server>
  </servers>
</settings>
{% endhighlight %}

In my case, there were not one, but 3 different files that I needed to push to staging. The Sonatype documentation does not give any information about how to accomplish this, only the one snippet from the page above:

	$ mvn gpg:sign-and-deploy-file
	> -DpomFile=target/myapp-1.0.pom
	> -Dfile=target/myapp-1.0.jar
	> -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/
	> -DrepositoryId=sonatype-nexus-staging

Fortunately, we can work with this. Maven constructs artifact names in a very strict fashion: `${artifactId}-${version}-${classifier}.${packaging}`. We can override these properties directly on the command line, allowing for multiple artifacts with a single pom. I had to upload a license, a javadoc jar, and the jar library itself, so I ran the following:

	$ mvn gpg:sign-and-deploy-file
	> -DpomFile=pom.xml
	> -Dfile=p4java.jar
	> -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/
	> -DrepositoryId=sonatype-nexus-staging

	$ mvn gpg:sign-and-deploy-file
	> -DpomFile=pom.xml
	> -Dfile=LICENSE.txt
	> -Dclassifier=license
	> -Dpackaging=txt
	> -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/
	> -DrepositoryId=sonatype-nexus-staging

	$ mvn gpg:sign-and-deploy-file
	> -DpomFile=pom.xml
	> -Dfile=p4java-javadoc.jar
	> -Dclassifier=javadoc
	> -Dpackaging=jar
	> -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/
	> -DrepositoryId=sonatype-nexus-staging

This automatically created a staging repository in my [sonatype oss](https://oss.sonatype.org/) account for com.perforce. From there it's just a matter of verifying the files, closing it, and releasing.

