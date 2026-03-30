##### Dockerfile ##### 
FROM gcc:12.2.0 AS build

COPY . /opt/test
WORKDIR /opt/test

RUN g++ -o app app.cpp

FROM ubuntu:bionic
WORKDIR /opt/test
COPY --from=build /opt/test ./
CMD ["./app"]