##### Dockerfile ##### 
FROM swift:5.5
WORKDIR /app
COPY . .

RUN apt-get update && apt-get install libsqlite3-dev
RUN swift package clean
RUN swift build
RUN mkdir /app/bin
RUN mv `swift build --show-bin-path` /app/bin

EXPOSE 8080
ENTRYPOINT ./bin/debug/Run serve --env local --hostname 0.0.0.0