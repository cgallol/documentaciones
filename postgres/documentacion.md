# POSTGRES

Especialmente para linux.

## Introduccion

Si es otra version muy anterior o no se instala bien ver instalacion script pagina principal postgres.

Usar manual de terminal:

```sh
    man psql
```

## Configuracion

Rutas para configurar si es necesario.

```bash
    sudo nano /etc/postgresql/14/main/postgresql.conf
```

```bash
    sudo nano pg_hba.conf
```

### Verificar Procesos

```sh
    pgrep -u postgres -fa -- -
```

```sh
    ps aux | grep postgres
```

### Versiones instaladas

```sh
    dpkg -l | grep postgres
```

### Restaurar postgres

Servicio

```sh
    sudo service postgresql restart
```

```sh
    sudo systemctl restart postgresql
```

### Problemas de permisos

Entrar en la ruta `pg_hba.conf` y modificarla

```sh
    # ANTES
    # "local" is for Unix domain socket connections only
    local   all             all                                     peer
```

```sh
    # DESPUES
    # "local" is for Unix domain socket connections only
    local   all             all                                     md5
```

Luego restaurar servicio de postgres

## Inicio

Los **nombres de usuario** son por ejemplo root , postgres, ... ,para ingresar con uno de ellos hacer:

```sh
    sudo -i -u NOMBRE_USUARIO
```

### Ingresar por defecto a consola psql

Por defecto se ingresa con usuario postgres

Alternativa 1

```sh
    sudo -io postgres
    psql
```

Alternativa 2

```sh
    sudo -u postgres psql
```

Alternativa 3 usando puerto

```sh
    sudo -u postgres psql -p 5432
```

## Creacion de usuarios desde fuera

### Usuario para **postgres**:

```sh
    sudo -iu postgres
    createuser user_name -P
```

### Crear usuarios de forma iterativa:

```sh
    sudo -iu postgres
    createuser --interactive --pwprompt
```

Iniciar con un nombre de usuario (_user_name_):

```sh
    sudo -iu postgres
    createuser --interactive user_name
```

## Ingresar con usuario creado

Se necesita una base de datos por defecto hay una que se llama postgres.

```sh
    sudo -iu postgres
    psql -U user_name -d database_example
```

```sh
    sudo -iu postgres
    psql database_name user_name
```

```sh
    sudo -iu postgres
    psql -h localhost -U user_name database_name
```

## Comandos dentro de PSQL

> #postgres:

### Crear usuario

> CREATE USER = CREATE ROLE + LOGIN PERMISSION

```js
    CREATE USER user_name PASSWORD 'password_name';
```

```js
    CREATE USER user_name WITH ENCRYPTED PASSWORD 'password_name';
```

### Crear base de datos

```js
    CREATE DATABASE database_name;
```

### Crear base de datos con propietario

```js
    CREATE DATABASE _database_ WITH OWNER _usuario_;
```

### Crear Rol

```js
    CREATE ROLE name_user;
```

Con una opcion:

```js
    CREATE ROLE name_user WITH _permisos ;
```

### Añadir permisos a un rol

```js
    ALTER ROLE rol_name WITH _permisos;
```

permisos disponibles:

-   SUPERUSER **|** NOSUPERUSER
-   CREATEDB **|** NOCREATEDB
-   CREATEROLE **|** NOCREATEROLE
-   INHERIT **|** NOINHERIT **_determinar si el rol heredará los privilegios de los roles de los que es miembro_**.
-   LOGIN **|** NOLOGIN **_permitir que el rol inicie sesión_**
-   REPLICATION **|** NOREPLICATION **_determinar si el rol es un rol de replicación_**
-   BYPASSRLS **|** NOBYPASSRLS **_determinar si el rol pasa por alto una política de seguridad de nivel de fila (RLS)._**
-   PASSWORD 'password' **|** PASSWORD NULL **_cambiar la contraseña del rol._**
-   CONNECTION LIMIT limit **_especificar el número de conexiones simultáneas que puede realizar un rol, -1 significa ilimitado._**
-   VALID UNTIL 'timestamp' **_establezca la fecha y la hora a partir de las cuales la contraseña del rol dejará de ser válida._**

### Renombrar rol

```js
    ALTER ROLE rol_name RENAME TO new_usuario;
```

### Eliminar rol

```js
    DROP ROLE rol_name;
```

### Crear un rol grupal

```js
    CREATE ROLE <groupname>;
```

### Asignar un grupo a un rol

```js
    GRANT <groupname> TO <role>
```

### Eliminar un grupo a un rol

Quita del grupo a ese rol

```js
    REVOKE <groupname> FROM <role>
```

### Crear esquemas

```js
    CREATE SCHEMA <schema>;
```

### Permiso a rol para conectarse a la base de datos

```js
    GRANT CONNECT ON DATABASE <database> TO <rol>;
```

### Permiso de privilegio de uso de esquemas

```js
    GRANT USAGE ON SCHEMA <schema> TO <rol>;
```

Si desea permitir que este rol cree nuevos objetos como tablas de este esquema, utilice el siguiente SQL en lugar del anterior:

```js
    GRANT USAGE, CREATE ON SCHEMA <schema> TO <rol>;
```

El siguiente paso es conceder acceso a las tablas. Como se mencionó en la sección anterior, la concesión puede realizarse en tablas individuales o en todas las tablas del esquema. Para tablas individuales, utilice el siguiente SQL:

```js
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE <table1>, <table2> TO <rol>;
```

Para todas las tablas y vistas del esquema, utilice el siguiente SQL:

```js
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA myschema TO readwrite;
```

### Dar todos los pribilegios.

Sobre una base de datos

```js
    GRANT ALL PRIVILEGES ON DATABASE database_name TO user_name;
```

### NOTA

Con los roles implementados, se simplifica el proceso de creación de usuarios. Simplemente crea el usuario y concédele uno de los roles existentes. Estas son las instrucciones SQL para este proceso:

```js
    CREATE USER <usuario> WITH PASSWORD 'secret_passwd';
    GRANT <read_rol> TO <usuario>;
```

Esta instrucción SQL otorga a myuser1 los mismos permisos que el rol de solo lectura.

Del mismo modo, puede conceder acceso de lectura y escritura a un usuario otorgando el rol readwrite

## Esquema public y rol public

Cuando se crea una nueva base de datos, PostgreSQL crea de forma predeterminada un esquema denominado public y concede acceso en este esquema a un rol de backend denominado public. A todos los usuarios y roles nuevos se les concede de forma predeterminada el rol public y, por lo tanto, pueden crear objetos en el esquema public.

PostgreSQL utiliza un concepto de ruta de búsqueda. La ruta de búsqueda es una lista de nombres de esquema que PostgreSQL comprueba cuando no se utiliza un nombre calificado del objeto de base de datos. Por ejemplo, cuando selecciona de una tabla denominada “mytable”, PostgreSQL busca esta tabla en los esquemas enumerados en la ruta de búsqueda. Elige la primera coincidencia que encuentra. De forma predeterminada, la ruta de búsqueda contiene los siguientes esquemas:

> postgres=# show search_path;
> search_path
>
> "$user", public
> (1 row)

El nombre **$user** se refiere al nombre del usuario que ha iniciado sesión actualmente. De forma predeterminada, no existe ningún esquema con el mismo nombre que el nombre de usuario. Por lo tanto, el esquema **public** se convierte en el esquema predeterminado siempre que se utiliza un nombre de objeto no calificado. Por este motivo, cuando un usuario intenta crear una nueva tabla sin especificar el nombre del esquema, la tabla se crea en el esquema **public**. Como se mencionó anteriormente, de forma predeterminada, todos los usuarios tienen acceso para crear objetos en el esquema **public** y, por lo tanto, la tabla se ha creado correctamente.

Esto se convierte en un problema si intenta crear un usuario de solo lectura. Incluso si restringe todos los privilegios, los permisos heredados a través del rol **public** permiten al usuario crear objetos en el esquema **public**.

Para solucionarlo, se debe revocar el permiso de creación predeterminado en el esquema **public** desde el rol **public** mediante la siguiente instrucción SQL:

```js
    REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

Asegúrese de ser el propietario del esquema public o de formar parte de un rol que le permita ejecutar esta instrucción SQL.

La siguiente declaración revoca la capacidad del rol público de conectarse a la base de datos:

```js
    REVOKE ALL ON DATABASE mydatabase FROM PUBLIC;
```

Esto garantiza que los usuarios no puedan conectarse a la base de datos de forma predeterminada a menos que se conceda explícitamente este permiso.

La revocación de los permisos del rol public afecta a todos los usuarios y roles existentes. Los usuarios y roles que deberían de poder conectarse a la base de datos o crear objetos en el esquema público deben recibir los permisos explícitamente antes de revocar los permisos del rol public en el entorno de producción.
