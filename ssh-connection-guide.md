# 🔑 Guía práctica: Conexión a servidores con SSH en Ubuntu/Linux

## 📌 Introducción

SSH (**Secure Shell**) es un protocolo que permite conectarse de forma segura a otro computador a través de la red.

Con SSH puedes acceder a la terminal de un servidor remoto como si estuvieras trabajando directamente en él.  

El comando principal es:

```bash
ssh usuario@ip-del-servidor
```

- **usuario** → el nombre del usuario con el que entras  
  *(ej: `ubuntu`, `ec2-user`, `root`, etc.)*.

- **ip-del-servidor** → la dirección IP o dominio del servidor remoto.

Si el servidor usa una **llave privada** para autenticación (en lugar de contraseña), debes indicar la llave con la opción `-i`:

```bash
ssh -i ruta/de/la/llave.pem usuario@ip-del-servidor
```

## ⚙️ Procedimiento paso a paso

### 1. Verifica qué tipo de llave tienes

Mira el archivo que te pasaron:

- Si termina en .pem, ya sirve directo en Linux.
- Si termina en .ppk, hay que convertirlo porque ese formato es de Windows (PuTTY).

Ejecutas el siguiente comando:

```bash
file ~/Descargas/tu-llave
```

```text
# Ejemplos de salida:
"PuTTY private key"            → es .ppk (formato de Windows/PuTTY)  
"PEM RSA private key"          → válido para Linux  
"OPENSSH PRIVATE KEY"          → válido para Linux
```

### 2. Crear la carpeta de llaves (si no existe)

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

💡 .ssh es la carpeta estándar para guardar tus llaves SSH.
Los permisos 700 aseguran que solo tú puedes acceder a ella.

### 3. Guardar la llave privada en esa carpeta

Ejemplo si la llave está en Descargas:

```bash
cp ~/Descargas/mi-llave.pem ~/.ssh/
```

👉 Copia el archivo mi-llave.pem a la carpeta .ssh.

ℹ️ Puedes guardarla en otro sitio, pero ~/.ssh/ es el lugar más seguro.

### 4. Ajustar permisos de la llave

```bash
chmod 600 ~/.ssh/mi-llave.pem
```

🔒 SSH rechaza llaves privadas con permisos más abiertos. Esto asegura que solo tú puedes leer la llave. 

### 5. Conectarte al servidor

```bash
ssh -i ~/.ssh/mi-llave.pem usuario@ip-del-servidor
```

Ejemplo:

```bash
ssh -i ~/.ssh/mi-llave.pem ubuntu@203.0.113.10
```

- 👉 ssh = comando para conectarse.
- 👉 -i ~/.ssh/mi-llave.pem = ruta de la llave privada.
- 👉 usuario@ip-del-servidor = usuario con el que entras (ej: ubuntu, ec2-user, azureuser, etc.) y dirección del servidor.

### 6. Si la llave está en formato .ppk

El formato .ppk (de Windows/Putty) no funciona directo en Linux.

Primero instalamos un programa que convierte .ppk a un formato que Linux entienda:

```bash
sudo apt update && sudo apt install -y putty-tools
```

Luego convertimos la llave:

```bash
puttygen ~/.ssh/mi-llave.ppk -O private-openssh -o ~/.ssh/mi-llave-openssh
```
💡 Si la .ppk tenía passphrase, puttygen te la pedirá durante la conversión.

👉 puttygen = convierte la llave.
👉 -O private-openssh = le digo que la convierta a formato OpenSSH (el que entiende Linux).
👉 -o ... = dónde quiero guardar la nueva llave.



Después damos permisos a la nueva llave:

```bash
chmod 600 ~/.ssh/mi-llave-openssh
```

Y nos conectamos igual que antes, pero usando la nueva llave:

```bash
ssh -i ~/.ssh/mi-llave-openssh usuario@ip-del-servidor
```

### 7. Configuración avanzada con ~/.ssh/config

Para no tener que escribir -i ruta y usuario@ip cada vez, usamos un archivo de configuración.

```bash
nano ~/.ssh/config
```

Ejemplo con varios servidores:

```bash
# 🔧 Servidor de pruebas
Host pruebas
  HostName 203.0.113.10
  User ubuntu
  Port 22
  IdentityFile ~/.ssh/llave-pruebas.pem
  IdentitiesOnly yes

# 🔧 Servidor de producción
Host produccion
  HostName 45.76.120.50
  User ubuntu
  Port 22
  IdentityFile ~/.ssh/llave-produccion.pem
  IdentitiesOnly yes

# 🔧 Servidor de la empresa
Host empresa
  HostName empresa.midominio.com
  User azureuser
  Port 2222
  IdentityFile ~/.ssh/llave-empresa-openssh
  IdentitiesOnly yes
```

ℹ️ Con Host defines un alias que puedes usar en lugar de escribir IP + usuario + llave cada vez.

👉 Explicación de cada campo:

- **Host** → alias inventado para ese servidor (puedes ponerle cualquier nombre).
- **HostName** → la dirección IP o dominio real del servidor.
- **User** → el usuario con el que te conectas.
- **Port** → puerto SSH (por defecto 22).
- **IdentityFile** → ruta de la llave que corresponde a ese servidor.
- **IdentitiesOnly yes** → fuerza a usar solo esa llave y no otras.

Guarda el archivo y dale permisos correctos:

```bash
chmod 600 ~/.ssh/config
```

### 8. Conexión usando alias

Ahora puedes conectarte con comandos simples:

```bash
ssh pruebas
ssh produccion
ssh empresa
```

👉 Ya no necesitas escribir -i ni la IP manualmente.

### 9. Verificar configuración

Para comprobar qué configuración usa SSH para un alias:

```bash
ssh -G pruebas
```

👉 Esto muestra toda la configuración aplicada (usuario, IP, llave, puerto, etc.).

### 10. ⚠️ Errores comunes

- “Permissions are too open”

Arregla los permisos del archivo.

👉 Solución:

```bash
chmod 600 ~/.ssh/mi-llave.pem
```

- “Permission denied (publickey)”

👉 Revisa:

1. Que el usuario sea correcto (ubuntu, ec2-user, azureuser, etc.).
2. Que la llave configurada sea la que corresponde (-i ...) o el alias con IdentityFile.
3. Prueba modo verboso: ssh -vvv -i ~/.ssh/tu-llave usuario@host.

- Timeout o no conecta

👉 Verifica que el servidor permita conexiones SSH en el **puerto 22** (o el que corresponda) desde tu IP.

### 11. Consejitos rápidos

Mantén tus llaves en ~/.ssh/ con permisos 600 y no las compartas.

Si usas mucho la misma llave, puedes cargarla en el agente:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/tu-llave.pem
ssh usuario@host
```

## ✅ Conclusión

Con este procedimiento puedes conectarte a múltiples servidores de forma segura en Linux, ya sea usando llaves .pem o .ppk.

El archivo ~/.ssh/config te permitirá organizar mejor tus accesos y conectarte con un simple comando.
