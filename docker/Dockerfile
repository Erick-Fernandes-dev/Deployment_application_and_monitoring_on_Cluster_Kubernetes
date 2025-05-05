FROM tomcat:11.0.6-jdk21-temurin-jammy

COPY jolokia-agent-jvm-2.2.9-javaagent.jar /usr/local/tomcat/lib/jolokia-agent.jar

ENV CATALINA_OPTS="-javaagent:/usr/local/tomcat/lib/jolokia-agent.jar=port=8778,host=0.0.0.0"

EXPOSE 8080 8778


RUN rm -rf /usr/local/tomcat/webapps/*

COPY jenkins.war /usr/local/tomcat/webapps/ROOT.war
