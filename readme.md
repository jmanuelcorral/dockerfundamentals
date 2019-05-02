# Empaquetando apps

- imagen (vendría a ser una clase, es la definición)
- contenedor (la instancia de la imagen, es tiempo de ejecución)

## Ficheros interesantes a conocer en un desarrollo con docker

- dockerfile
- docker-compose
- dockerignore

## Comandos en un dockerfile tiempo de construccion de imagen

|Comando|Explicacion|
|---|---|
|FROM|Primera linea de un dockerfile (Obligatoria), de que imagen "heredamos"|
|MAINTAINER|Quien mantiene esto|
|ADD|Copia un fichero desde el host a la imagen|
|ENV|Setea una variable de entorno en el contenedor|
|EXPOSE|Abre un puerto en los contenedores linkados|
|ONBUILD|Comando que solo será lanzado si esta imagen es utilizada como base para otra imagen|
|RUN|Ejecuta un comando y salva el resultado en una nueva capa|
|USER|Setea el usuario por defecto en ejecución en el contenedor|
|VOLUME|Crea un volumen que puede ser compartido entre contenedores o con la maquina HOST|
|WORKDIR|Setea el directorio por defecto en el contenedor|

## Comandos en un dockerfile tiempo de ejecución

|comando|Explicacion|
|---|---|
|ENTRYPOINT|El comando que se ejecutará primero|
|CMD|El comando se ejecutará después del entrypoint|

### ¿EntryPoint o CMD?

Dockerfiles Debe de existir al menos un EntryPoint o un CMD en un dockerfile.

EntryPoint debería de ser utilizado si el container lo estamos utilizando como un ejecutable final.

CMD debería ser utilizado como una forma de definir argumentos por defecto en un Entrypoint, o para ejecutar un comando ad-hoc en un contenedor.

CMD Se puede override ( ejecutando un docker run contenedor [comando] ).

### Layers en docker

Cada imagen es un conjunto de capas, partiremos de que en tiempo de container o tiempo de ejecucion, todas son readonly menos la última.
Una capa es 
El hecho de utilizar capas, y hashearlas, permite a docker "reaprovechar" parte de las imagenes, para construir otras imagenes.

## ejemplo de dockerfile con node

Ejemplo completo [aqui](https://github.com/jmanuelcorral/cherryprint/blob/master/pdfGenService/dockerfile)

```yml
FROM node:10-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

CMD [ "npm", "start" ]
```

## Ejemplo de dockerfile con .net

```yml
FROM mcr.microsoft.com/dotnet/core/sdk:2.2
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet build --no-restore -c Release -o /app
ENTRYPOINT ["dotnet", "miappmolona.dll"]
```

## Best practices al elegir base image

Usar [alpine](https://nickjanetakis.com/blog/the-3-biggest-wins-when-using-alpine-as-a-base-docker-image)

imagenes de Microsoft netcore sdk y netcore (han vuelto a cambiar):
para desarrollar: [sdk](https://hub.docker.com/_/microsoft-dotnet-core-sdk/)
para publicar: [runtime](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/)

## MULTI STAGE 

Tenemos una aplicación SPA, por lo que necesito mi tooling de Frontend y mi tooling de Backend.

```yml
ARG NODE_IMAGE=node:8.11
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS dotnet-build
WORKDIR /src

FROM ${NODE_IMAGE} as node-build
WORKDIR /web
COPY src/ .
RUN npm install
RUN npm run build:prod

FROM dotnet-build as publish
WORKDIR /src/wwwroot
COPY --from=node-build /web/wwwroot .
WORKDIR /src
COPY . .
WORKDIR /src/src/Web/WebSPA
RUN dotnet publish -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebSPA.dll"]

``