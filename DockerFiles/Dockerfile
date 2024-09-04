FROM openjdk:17-jdk-slim

ENV TZ=Asia/Ho_Chi_Minh
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /app
COPY target/*.jar /app/

ENTRYPOINT exec java ${JAVA_OPTS} -jar /app/Master*.jar ${APP_OPTS}
