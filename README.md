#!/bin/bash

# Script para instalar y configurar un contenedor XAMPP en Docker

# Verifica si Docker está instalado
if ! command -v docker &> /dev/null
then
    echo "Docker no está instalado. Por favor, instala Docker primero."
    exit 1
fi

# Nombre del contenedor
CONTAINER_NAME="mi-xampp"

# Puerto para SSH
SSH_PORT=6022

# Puerto para HTTP
HTTP_PORT=6080

# Directorio local para los archivos web
LOCAL_WWW_DIR="/ruta/local/www"

# Verifica si el directorio local existe, si no, lo crea
if [ ! -d "$LOCAL_WWW_DIR" ]; then
    echo "Creando directorio local para archivos web..."
    mkdir -p "$LOCAL_WWW_DIR"
fi

# Descarga la imagen de XAMPP
echo "Descargando la imagen de XAMPP..."
docker pull tomsik68/xampp

# Verifica si ya existe un contenedor con el mismo nombre
if [ "$(docker ps -aq -f name=^/${CONTAINER_NAME}$)" ]; then
    echo "Ya existe un contenedor con el nombre $CONTAINER_NAME. Eliminándolo..."
    docker stop $CONTAINER_NAME
    docker rm $CONTAINER_NAME
fi

# Crea y ejecuta el contenedor XAMPP
echo "Creando y ejecutando el contenedor XAMPP..."
docker run --name $CONTAINER_NAME -p $SSH_PORT:22 -p $HTTP_PORT:80 -d -v $LOCAL_WWW_DIR:/www tomsik68/xampp

# Verifica si el contenedor se está ejecutando
if [ "$(docker ps -q -f name=^/${CONTAINER_NAME}$)" ]; then
    echo "El contenedor XAMPP se ha creado y está en ejecución."
    echo "Puedes acceder a la interfaz web de XAMPP visitando http://localhost:$HTTP_PORT/dashboard/"
    echo "Para acceder al contenedor via SSH: ssh root@localhost -p $SSH_PORT (contraseña: root)"
else
    echo "Hubo un problema al crear el contenedor XAMPP."
fi

echo "Proceso completado."
