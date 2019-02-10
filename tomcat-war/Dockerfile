# syntax = docker/dockerfile:experimental
# Step 1 Build jar and jre
FROM amazonlinux:2 as builder

WORKDIR /work

# install java, maven
RUN yum install -y tar wget gzip

RUN wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"   "http://download.oracle.com/otn-pub/java/jdk/11.0.2+9/f51449fcd52f4d52b93a989c5c56ed3c/jdk-11.0.2_linux-x64_bin.rpm"
RUN rpm -ivh /work/jdk-11.0.2_linux-x64_bin.rpm

ADD https://www-eu.apache.org/dist/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz /work
RUN tar zxvf apache-maven-3.6.0-bin.tar.gz

ENV JAVA_HOME /usr/java/latest
ENV PATH $PATH:/work/apache-maven-3.6.0/bin

# build app
ADD pom.xml /work/pom.xml
ADD src /work/src
RUN --mount=type=cache,target=/root/.m2 mvn clean install

# build jre
RUN jlink \
    --verbose \
    --module-path "$JAVA_HOME/jmods" \
    --add-modules java.base,java.logging,java.xml,jdk.unsupported,java.sql,java.naming,java.desktop,java.management,java.security.jgss,java.instrument,jdk.management \
    --compress 2 \
    --no-header-files \
    --no-man-pages \
    --strip-debug \
    --output /work/jre 

## Step 2 run
FROM amazonlinux:2

RUN amazon-linux-extras install tomcat8.5 -y \
    && rpm -ev --nodeps java-1.8.0-openjdk java-1.8.0-openjdk-headless systemd\
    && rm -rf /var/cache/yum/* \
    && yum clean all

WORKDIR /opt/app
ENV JAVA_HOME /opt/jre
ENV JAVACMD $JAVA_HOME/bin/java
COPY --from=builder /work/target/spring-boot-sample-war-2.1.2.RELEASE.war /var/lib/tomcat/webapps/ROOT.war
COPY --from=builder /work/jre /opt/jre

ENV JAVA_HOME /opt/jre
ENV PATH $PATH:/opt/jre/bin

EXPOSE 8080
CMD /usr/libexec/tomcat/server start

