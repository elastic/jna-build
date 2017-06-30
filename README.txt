This project exists in order to build a jna jar which supports platforms supported by elastic.
It builds native linux bits for jna in vagrant, and uses the provided native bits for mac and windows.

Publish to Maven Central
------------------------

First set a shell variable for convenience:

    export PRERELEASE=2 # or whatever ordinal makes sense

Ensure you have the OSSRH Sonatype staging environment set up in
~/.m2/settings.xml:

    <server>
      <id>sonatype-nexus-staging</id>
      <username>user</username>
      <password>pass</password>
    </server>

Then build the artifact and pom:

    % vagrant up
    % gradle assemble generatePomFileForJnaPublication -Dprerelease=$PRERELEASE
    % vagrant halt

In order for OSSRH to accept the package, it must have sources and
javadoc jars as well.  We don't have any automation here to build
them, so simply reuse the upstream net.java.dev/jna ones:

    curl -L -o target/jna-4.4.0-${PRERELEASE}-sources.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/4.4.0/jna-4.4.0-sources.jar
    curl -L -o target/jna-4.4.0-${PRERELEASE}-javadoc.jar \
      https://oss.sonatype.org/content/repositories/releases/net/java/dev/jna/jna/4.4.0/jna-4.4.0-javadoc.jar

Now upload to OSSRH, signing with the Elasticsearch key (it will
prompt for passphrase):

    mvn gpg:sign-and-deploy-file \
      -DgroupId=org.elasticsearch \
      -DartifactId=jna \
      -DgeneratePom=false \
      -Dpackaging=jar \
      -DrepositoryId=sonatype-nexus-staging \
      -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
      -DpomFile=build/distributions/jna-4.4.0-${PRERELEASE}.pom \
      -Dfile=build/distributions/jna-4.4.0-${PRERELEASE}.jar \
      -Dsources=target/jna-4.4.0-${PRERELEASE}-sources.jar \
      -Djavadoc=target/jna-4.4.0-${PRERELEASE}-javadoc.jar \
      -Dgpg.keyname=D88E42B4
