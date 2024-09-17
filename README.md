
# Guía de Configuración de Replicación Maestro-Esclavo en MySQL

Esta guía describe el proceso para configurar la replicación Maestro-Esclavo en MySQL versión 8.4, asumiendo que el servidor maestro ya está en funcionamiento. La guía está dividida en dos secciones: Configuración del Maestro y Configuración del Esclavo.

---

## Configuración del Maestro

### 1. Editar el Archivo de Configuración de MySQL (Maestro)

Abre el archivo `my.ini` ubicado en:
```
C:\ProgramData\MySQL\MySQL Server 8.4\my.ini
```

Agrega las siguientes líneas bajo la sección `[mysqld]`:
```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-do-db=parking
```

### 2. Reiniciar el Servicio MySQL

Ejecuta los siguientes comandos para detener y arrancar el servicio MySQL:
```bash
net stop mysql
net start mysql
```

### 3. Crear el Usuario Replicator

Inicia sesión en MySQL como el usuario root:
```bash
mysql -u root -p
```

Luego ejecuta los siguientes comandos para crear el usuario `replicator` y otorgarle los privilegios necesarios:
```sql
CREATE USER 'replicator'@'%' IDENTIFIED BY 'Oem2024*';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;
```

### 4. Preparar el Maestro para la Replicación

Bloquea las tablas y muestra los logs binarios:
```sql
FLUSH TABLES WITH READ LOCK;
SHOW BINARY LOGS;
```

Anota la salida, ya que necesitarás esta información para configurar el esclavo. Ejemplo de salida:
```
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       154 | No        |
+------------------+-----------+-----------+
```

### 5. Respaldar la Base de Datos

Realiza un volcado de la base de datos que deseas replicar:
```bash
mysqldump -u root -p --databases parking > C:\backup\parking.sql
```

Después de completar el respaldo, desbloquea las tablas:
```sql
UNLOCK TABLES;
```

---

## Configuración del Esclavo

### 1. Editar el Archivo de Configuración de MySQL (Esclavo)

Abre el archivo `my.ini` en el servidor esclavo ubicado en:
```
C:\ProgramData\MySQL\MySQL Server 8.4\my.ini
```

Agrega las siguientes líneas bajo la sección `[mysqld]`:
```ini
[mysqld]
server-id=2
relay-log=relay-log
```

### 2. Reiniciar el Servicio MySQL

Ejecuta los siguientes comandos para detener y arrancar el servicio MySQL:
```bash
net stop mysql
net start mysql
```

### 3. Restaurar el Respaldo en el Esclavo

Restaura el volcado de la base de datos creado en el maestro al servidor esclavo:
```bash
mysql -u root -p < C:\backup\parking.sql
```

### 4. Configurar el Esclavo para Iniciar la Replicación

Inicia sesión en MySQL en el servidor esclavo:
```bash
mysql -u root -p
```

Ejecuta el siguiente comando para configurar la replicación en el esclavo, usando el archivo de log binario y la posición anotada anteriormente del maestro:
```sql
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='10.0.0.111',
SOURCE_USER='replicator',
SOURCE_PASSWORD='Oem2024*',
SOURCE_LOG_FILE='mysql-bin.000001',
SOURCE_LOG_POS=892,
GET_MASTER_PUBLIC_KEY=1;
```

### 5. Iniciar la Replicación

Ejecuta el siguiente comando para iniciar el proceso de replicación:
```sql
START REPLICA;
```

### 6. Verificar el Estado de la Replicación

Puedes verificar el estado de la replicación con:
```sql
START REPLICA STATUS\G;
```

Asegúrate de que el estado sea `Running` y que no haya errores.

---

## Conclusión

Has configurado exitosamente la replicación Maestro-Esclavo en MySQL. Asegúrate de monitorear el proceso de replicación periódicamente para garantizar la sincronización de datos entre los servidores maestro y esclavo.
