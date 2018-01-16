Elasticsearch JNA Build
=======================

This project exists in order to build a jna jar which supports
platforms supported by elastic.  It builds native linux bits for jna
in vagrant, and uses the provided native bits for mac and windows.


Publish to Maven Central (Normal case)
--------------------------------------

In the default case, we track with upstream JNA.  This is when the
local version exactly matches net.java.dev/jna.

First set shell variables for convenience:

    # match whatever's in build.gradle
    export VERSION=4.5.1

Note that when upgrading the jna version, the `upstreamVersion` variable 
in the build file should be updated too.

Ensure you have the OSSRH Sonatype staging environment set up in
`~/.m2/settings.xml`:

    <server>
      <id>sonatype-nexus-staging</id>
      <username>user</username>
      <password>pass</password>
    </server>

Then build:

    % vagrant up && gradle assemble && vagrant halt

In order for OSSRH to accept the package, it must have sources and
javadoc jars as well.  We don't have any automation here to build
them, so simply reuse from net.java.dev/jna:

    % curl -L -o build/jna-${VERSION}-sources.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/${VERSION}/jna-${VERSION}-sources.jar
    % curl -L -o build/jna-${VERSION}-javadoc.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/${VERSION}/jna-${VERSION}-javadoc.jar

Now upload to OSSRH, signing with the Elasticsearch key (it will
prompt for passphrase, or if desperate you can use -Dgpg.passphrase
but please try to avoid):

    % mvn gpg:sign-and-deploy-file \
        -DgroupId=org.elasticsearch \
        -DartifactId=jna \
        -DgeneratePom=false \
        -Dpackaging=jar \
        -DrepositoryId=sonatype-nexus-staging \
        -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
        -DpomFile=build/distributions/jna-${VERSION}.pom \
        -Dfile=build/distributions/jna-${VERSION}.jar \
        -Dsources=build/jna-${VERSION}-sources.jar \
        -Djavadoc=build/jna-${VERSION}-javadoc.jar \
        -Dgpg.keyname=D88E42B4


Publish to Maven Central (With only local modifications)
--------------------------------------------------------

Sometimes we may need to publish a jar that has local modifications
when there hasn't been an upstream release.  See for example
[4.4.0-1][example-release].  The instructions are almost identical,
but we introduce a `suffix`.

Set shell variables for convenience:

    # Match whatever's in build.gradle
    export VERSION=4.4.0

    # Start at 1, but this may be incremented to whatever makes sense
    export SUFFIX="-2"

Then build the artifact and pom as usual, but supply `version.suffix`
property:

    % vagrant up && gradle assemble -Dversion.suffix=$SUFFIX && vagrant halt

In order for OSSRH to accept the package, it must have sources and
javadoc jars as well.  We don't have any automation here to build
them, so simply reuse from upstream net.java.dev/jna:

    % curl -L -o build/jna-${VERSION}${SUFFIX}-sources.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/${VERSION}/jna-${VERSION}-sources.jar
    % curl -L -o build/jna-${VERSION}${SUFFIX}-javadoc.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/${VERSION}/jna-${VERSION}-javadoc.jar

Now upload to OSSRH, signing with the Elasticsearch key:

    mvn gpg:sign-and-deploy-file \
      -DgroupId=org.elasticsearch \
      -DartifactId=jna \
      -DgeneratePom=false \
      -Dpackaging=jar \
      -DrepositoryId=sonatype-nexus-staging \
      -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
      -DpomFile=build/distributions/jna-${VERSION}${SUFFIX}.pom \
      -Dfile=build/distributions/jna-${VERSION}${SUFFIX}.jar \
      -Dsources=build/jna-${VERSION}${SUFFIX}-sources.jar \
      -Djavadoc=build/jna-${VERSION}${SUFFIX}-javadoc.jar \
      -Dgpg.keyname=D88E42B4


[example-release]: http://search.maven.org/#artifactdetails%7Corg.elasticsearch%7Cjna%7C4.4.0-1%7Cjar
