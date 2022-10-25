FROM openjdk:8 AS BUILD_IMAGE
RUN apt update && apt install maven -y
WORKDIR /app
RUN git clone -b ci-jenkins https://github.com/Hemanth0467/vprofile-project.git
copy . .
RUN mvn -s settings.xml install

FROM tomcat:8-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE /var/lib/jenkins/workspace/Build/target/vprofile-4.2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
