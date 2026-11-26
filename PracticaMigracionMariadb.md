# Práctica 2: Migración de una base de datos en MariaDB

## Objetivo

Migrar una base de datos completa (estructura, datos y vista) hacia una **base nueva** usando solo MariaDB desde la terminal, y **validar** que la migración quedó íntegra.

## Requisitos

- MariaDB instalado y en ejecución.
- Una terminal con acceso a un usuario administrador de MariaDB (en los ejemplos, `root`).
- Una carpeta de trabajo:

```bash
mkdir practica2 && cd practica2
```

> En versiones anteriores a MariaDB 10.5 los comandos se llaman `mysql` y `mysqldump`.

## Escenario

```
┌──────────────┐    mariadb-dump     ┌──────────────────┐
│  biblioteca  │ ──────────────────► │ biblioteca_nueva │
│   (origen)   │    archivo .sql     │    (destino)     │
└──────────────┘                     └──────────────────┘
```

> Para mantenerla sencilla, origen y destino están en el mismo servidor. Si tuvieras un segundo servidor, el proceso es idéntico: solo copias el archivo `.sql` y lo importas allá.

---

## Paso 1. Crear la base de datos de origen

```bash
mariadb -u root -p
```

```sql
CREATE DATABASE biblioteca;
USE biblioteca;

CREATE TABLE autores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    pais VARCHAR(50)
);

CREATE TABLE libros (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    anio INT,
    autor_id INT NOT NULL,
    FOREIGN KEY (autor_id) REFERENCES autores(id)
);

INSERT INTO autores (nombre, pais) VALUES
('Gabriel Garcia Marquez', 'Colombia'),
('Julio Cortazar', 'Argentina'),
('Octavio Paz', 'Mexico');

INSERT INTO libros (titulo, anio, autor_id) VALUES
('Cien anos de soledad', 1967, 1),
('El amor en los tiempos del colera', 1985, 1),
('Rayuela', 1963, 2),
('El laberinto de la soledad', 1950, 3);

CREATE VIEW v_libros_autor AS
SELECT l.titulo, l.anio, a.nombre AS autor
FROM libros l
JOIN autores a ON a.id = l.autor_id;

SELECT * FROM v_libros_autor;
```

**Resultado esperado:** la vista muestra los 4 libros con su autor.

Salir con `EXIT;`

## Paso 2. Registrar los datos del origen (para comparar después)

```bash
mariadb -u root -p -e "
SELECT COUNT(*) AS autores FROM biblioteca.autores;
SELECT COUNT(*) AS libros FROM biblioteca.libros;
CHECKSUM TABLE biblioteca.autores, biblioteca.libros;"
```

**Resultado esperado:** 3 autores, 4 libros y un valor de `Checksum` para cada tabla.

**Anota** estos resultados.

## Paso 3. Hacer un respaldo de seguridad antes de migrar

```bash
mariadb-dump -u root -p --databases biblioteca > respaldo_previo.sql
```

Comprobar que se creó:

```bash
ls -lh respaldo_previo.sql
```

> Regla de oro: **siempre** se respalda antes de migrar.

## Paso 4. Exportar la base de datos

Se exporta **sin** la opción `--databases` para que el archivo no cree ni use la base original y podamos importarlo en otra con distinto nombre:

```bash
mariadb-dump -u root -p --single-transaction --routines --triggers --events biblioteca > biblioteca_migracion.sql
```

## Paso 5. Crear la base de datos de destino

```bash
mariadb -u root -p -e "CREATE DATABASE biblioteca_nueva;"
```

## Paso 6. Importar en el destino

```bash
mariadb -u root -p biblioteca_nueva < biblioteca_migracion.sql
```

Si no aparece ningún error, la importación fue exitosa.

## Paso 7. Validar la migración

### 7.1 Comparar conteos y checksum

```bash
mariadb -u root -p -e "
SELECT COUNT(*) AS autores FROM biblioteca_nueva.autores;
SELECT COUNT(*) AS libros FROM biblioteca_nueva.libros;
CHECKSUM TABLE biblioteca_nueva.autores, biblioteca_nueva.libros;"
```

**Resultado esperado:** 3 autores, 4 libros y los **mismos valores de `Checksum`** del Paso 2.

### 7.2 Verificar que la vista funciona

```bash
mariadb -u root -p -e "SELECT * FROM biblioteca_nueva.v_libros_autor;"
```

**Resultado esperado:** los mismos 4 libros con su autor.

### 7.3 Verificar la integridad referencial

```bash
mariadb -u root -p -e "INSERT INTO biblioteca_nueva.libros (titulo, anio, autor_id) VALUES ('Prueba', 2020, 999);"
```

**Resultado esperado:** error de clave foránea. Esto confirma que la restricción también se migró.

### 7.4 Verificar la estructura

```bash
mariadb -u root -p -e "SHOW CREATE TABLE biblioteca_nueva.libros\G"
```

## Paso 8. Registrar el resultado

Completa esta tabla:

| Comprobación | Origen | Destino | ¿Coincide? |
|---|---|---|---|
| Filas en `autores` | 3 | | |
| Filas en `libros` | 4 | | |
| Checksum `autores` | | | |
| Checksum `libros` | | | |
| Vista `v_libros_autor` funciona | Sí | | |
| Clave foránea activa | Sí | | |

Si todo coincide, la migración fue exitosa.

---

## Responde las siguientes preguntas

1. ¿Por qué se hace un respaldo antes de migrar?
2. ¿Para qué sirve la opción `--single-transaction`?
3. ¿Qué diferencia hay entre exportar con y sin `--databases`?
4. ¿Qué dos pruebas permiten confirmar que los datos migraron sin pérdidas?
5. Si el destino estuviera en otro servidor, ¿qué paso adicional sería necesario?
