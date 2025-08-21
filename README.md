#!/bin/bash

# Script para crear una imagen Docker personalizada de XAMPP Lite con puertos personalizados
# Autor: [Tu nombre]
# Fecha: [Fecha]

# Crear el Dockerfile
cat <<EOL > Dockerfile
# Usar una imagen base de Ubuntu
FROM ubuntu:20.04

# Instalar dependencias necesarias
RUN apt-get update && apt-get install -y \\
    wget \\
    tar \\
    curl \\
    openssh-server \\
    supervisor \\
    && rm -rf /var/lib/apt/lists/*

# Descargar XAMPP Lite
RUN wget https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/8.2.4/xampp-linux-x64-8.2.4-0-installer.run -O /tmp/xampp-installer.run

# Instalar XAMPP Lite
RUN chmod +x /tmp/xampp-installer.run && \\
    /tmp/xampp-installer.run --mode unattended --installer-language en && \\
    rm /tmp/xampp-installer.run

# Configurar SSH
RUN mkdir /var/run/sshd
RUN echo 'root:password' | chpasswd  # Cambia 'password' por tu contraseña deseada
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Configurar Supervisor para gestionar servicios
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Exponer puertos personalizados
EXPOSE 6022
EXPOSE 6033
EXPOSE 6080

# Iniciar servicios con Supervisor
CMD ["/usr/bin/supervisord"]
EOL

# Crear el archivo de configuración de Supervisor
cat <<EOL > supervisord.conf
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:apache]
command=/opt/lampp/lampp startapache

[program:mysql]
command=/opt/lampp/lampp startmysql
EOL

# Construir la imagen Docker
echo "Construyendo la imagen Docker..."
docker build -t xampp-lite-custom .

# Verificar que la imagen se construyó correctamente
if [ $? -eq 0 ]; then
    echo "Imagen Docker construida correctamente."
else
    echo "Error al construir la imagen Docker."
    exit 1
fi

# Ejecutar el contenedor con puertos personalizados
echo "Ejecutando el contenedor..."
docker run --name xampp-custom -d \
  -p 6022:22 \
  -p 6033:3306 \
  -p 6080:80 \
  xampp-lite-custom

# Verificar que el contenedor está en ejecución
if [ $? -eq 0 ]; then
    echo "Contenedor ejecutándose correctamente."
    echo "Accede a los servicios en:"
    echo " - SSH: localhost:6022 (usuario: root, contraseña: password)"
    echo " - MySQL: localhost:6033 (usuario: root, sin contraseña)"
    echo " - HTTP: http://localhost:6080"
else
    echo "Error al ejecutar el contenedor."
    exit 1
fi

# Configurar inicio automático del contenedor
echo "Configurando inicio automático del contenedor..."
docker update --restart unless-stopped xampp-custom

if [ $? -eq 0 ]; then
    echo "Inicio automático configurado correctamente."
else
    echo "Error al configurar el inicio automático."
    exit 1
fi
