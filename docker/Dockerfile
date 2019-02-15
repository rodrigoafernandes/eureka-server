FROM cvcbrasil/java:8u172-jre

RUN mkdir /opt/eureka-server

WORKDIR /opt/eureka-server

COPY ./target/eureka-server*.jar eureka-server.jar

SHELL ["/bin/bash","-c"]

EXPOSE 8080
EXPOSE 5005

ENTRYPOINT java ${ADDITIONAL_OPTS} -jar eureka-server.jar