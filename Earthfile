VERSION 0.6
FROM openjdk:8-jdk-alpine


ARG IMAGE_NAME=earthly-example
ARG VERSION=1.0.0


RUN apk add --update --no-cache gradle
WORKDIR /java-example

deps:
    COPY build.gradle ./
    RUN gradle build

build:
    FROM +deps
    COPY src src
    RUN gradle build
    RUN gradle install
    SAVE ARTIFACT build/install/java-example/bin AS LOCAL build/bin
    SAVE ARTIFACT build/install/java-example/lib AS LOCAL build/lib

docker:
    COPY +build/bin bin
    COPY +build/lib lib
    ENTRYPOINT ["/java-example/bin/java-example"]
    SAVE IMAGE --push ghcr.io/jgibbarduk/$IMAGE_NAME:$VERSION

integration-test:
    FROM earthly/dind:alpine
	COPY ./docker-compose.yml .
	RUN apk update
	RUN apk add postgresql-client
	WITH DOCKER --compose docker-compose.yml --load app:latest=+docker
		RUN while ! pg_isready --host=localhost --port=5432; do sleep 1; done ;\
			docker run --network=default_java/part6_default app | grep "Opened database successfully!"
	END

publish:
    FROM +docker
    SAVE IMAGE --cache-from=ghcr.io/jgibbarduk/$IMAGE_NAME:cache --push ghcr.io/jgibbarduk/$IMAGE_NAME:$VERSION