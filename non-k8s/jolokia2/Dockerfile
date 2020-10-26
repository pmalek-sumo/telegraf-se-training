FROM tomcat:jdk8-adoptopenjdk-openj9
ADD https://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.2/jolokia-jvm-1.6.2-agent.jar \
    /jolokia-jvm-1.6.2-agent.jar
ENV CATALINA_OPTS "-javaagent:/jolokia-jvm-1.6.2-agent.jar=host=0.0.0.0,port=8778"