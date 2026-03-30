##### Dockerfile ##### 
## Builder Stage ##
FROM dart AS build
WORKDIR /app
COPY pubspec.* /app/
RUN dart pub get --no-precompile
COPY app.dart /app/
RUN dart compile exe app.dart -o app

## Run stage ##
FROM debian:stable-slim
COPY --from=build /app/app /app/app
CMD ["/app/app"]