# Cuestionario: Migración de bases de datos en MariaDB

## 1. ¿Por qué se hace un respaldo antes de migrar?

Se realiza un respaldo antes de migrar para conservar una copia de seguridad de la base de datos original y evitar la pérdida de información si ocurre algún error durante el proceso. También permite recuperar los datos en caso de que la migración falle o la información se dañe.

En nuestra práctica utilizamos `mariadb-dump` para generar un archivo `.sql` que contiene la información de la base de datos.

```bash
mariadb-dump -u root -p --databases biblioteca > respaldo_previo.sql
```

---

## 2. ¿Para qué sirve la opción `--single-transaction`?

La opción `--single-transaction` permite realizar el respaldo dentro de una sola transacción, obteniendo una copia consistente de los datos en el momento en que inicia el respaldo, sin bloquear las tablas en motores compatibles como InnoDB.

En nuestra práctica se utilizó para generar un respaldo más seguro y consistente durante el proceso de migración.

```bash
mariadb-dump -u root -p --single-transaction --routines --triggers --events biblioteca > biblioteca_migracion.sql
```

---

## 3. ¿Qué diferencia hay entre exportar con y sin `--databases`?

La diferencia es que al utilizar `--databases` se incluye el nombre de la base de datos y las instrucciones necesarias para crearla en el archivo de respaldo. Sin esta opción, se exporta el contenido de la base de datos indicada, pero no necesariamente se incluyen las instrucciones para crear la base de datos.

**Con `--databases`:**

```bash
mariadb-dump -u root -p --databases biblioteca > respaldo.sql
```

Incluye la base de datos y permite que el respaldo contenga la instrucción para crearla.

**Sin `--databases`:**

```bash
mariadb-dump -u root -p biblioteca > respaldo.sql
```

Exporta las tablas y su contenido, sin incluir la instrucción para crear la base de datos mediante `CREATE DATABASE`.

---

## 4. ¿Qué dos pruebas permiten confirmar que los datos migraron sin pérdidas?

Las dos pruebas que utilizamos en nuestra práctica fueron:

**1. Conteo de registros (`COUNT(*)`):** permite comprobar que la cantidad de registros de las tablas originales sea igual a la cantidad de registros de las tablas migradas.

```sql
SELECT COUNT(*) FROM biblioteca.autores;
SELECT COUNT(*) FROM biblioteca_nueva.autores;

SELECT COUNT(*) FROM biblioteca.libros;
SELECT COUNT(*) FROM biblioteca_nueva.libros;
```

**2. Comparación de checksum (`CHECKSUM TABLE`):** permite comparar los valores de comprobación de las tablas originales y las migradas para verificar que su contenido coincida.

```sql
CHECKSUM TABLE biblioteca.autores, biblioteca.libros;
CHECKSUM TABLE biblioteca_nueva.autores, biblioteca_nueva.libros;
```

Al obtener la misma cantidad de registros y los mismos valores de checksum, podemos comprobar que las tablas comparadas conservaron sus datos durante la migración.

---

## 5. Si el destino estuviera en otro servidor, ¿qué paso adicional sería necesario?

Sería necesario establecer una conexión remota con el servidor destino, asegurándonos de tener su dirección IP o nombre de host, el puerto de conexión, un usuario con permisos y acceso de red habilitado.

Después, transferiríamos el archivo de respaldo SQL al servidor destino y realizaríamos la importación en la base de datos correspondiente.

Por ejemplo, podríamos utilizar `scp` para copiar el respaldo a otro servidor:

```bash
scp biblioteca_limpia.sql usuario@IP_SERVIDOR:/ruta/destino/
```

- `scp`: copia archivos mediante una conexión SSH.
- `biblioteca_limpia.sql`: archivo que se va a transferir.
- `usuario`: usuario del servidor destino.
- `IP_SERVIDOR`: dirección IP del servidor remoto.
- `/ruta/destino/`: ubicación donde se guardará el archivo.

Posteriormente, en el servidor destino, importaríamos el respaldo utilizando MariaDB.

```bash
mariadb -u root -p biblioteca_nueva < biblioteca_limpia.sql
```

De esta manera, la migración podría realizarse entre dos servidores diferentes, en lugar de hacerlo dentro del mismo equipo.