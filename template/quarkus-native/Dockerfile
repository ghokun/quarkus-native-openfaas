# Builder image
FROM oracle/graalvm-ce:20.1.0-java11 as builder
RUN gu install native-image

# Package function as a native binary
RUN mkdir /app
WORKDIR /app
ADD mvnw /app/
ADD .mvn /app/.mvn/
ENV GRAALVM_HOME=$JAVA_HOME
ADD function /app/function
RUN ./mvnw -f /app/function/pom.xml package -Pnative

# Watchdog image
FROM openfaas/of-watchdog:0.7.7 as watchdog


FROM frolvlad/alpine-glibc:alpine-3.10
RUN apk add --no-cache ca-certificates
RUN addgroup -S app && adduser -S -g app app
RUN apk --no-cache add ca-certificates curl

WORKDIR /root/
RUN mkdir -p /home/app
WORKDIR /home/app
COPY --from=builder /app/function/target/*-runner /usr/bin/function
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog
USER app

ENV cgi_headers="true"
ENV fprocess="function -Xmx64m"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:9090"

ENV exec_timeout="20s"
ENV write_timeout="25s"
ENV read_timeout="25s"

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]