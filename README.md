# Escenario Técnico: Sistema "BiblioGest 2.0"
El sistema debe gestionar el inventario físico de una biblioteca. El modelo consta de:

SOCIOS: (DNI, Nombre, Email, Telefono).
LIBROS: (ISBN, Titulo, Autor, Paginas).
EJEMPLARES: (ID_Ejemplar, ISBN, Estado). Relacionado con LIBROS.
PRESTAMOS: (Socio, Ejemplar, Fecha). Relacionado con SOCIOS y EJEMPLARES.

# 1: Definición de Estructura y Tipos de Datos

## 1. Código SQL para crear el esquema relacional definiendo claves primarias, foráneas y opciones de borrado.

```sql
CREATE TABLE SOCIOS (
    DNI VARCHAR2(9),
    Nombre VARCHAR2(100),
    Email VARCHAR2(100),
    Telefono VARCHAR2(15),
    PRIMARY KEY (DNI)
);

CREATE TABLE LIBROS (
    ISBN VARCHAR2(13),
    Titulo VARCHAR2(200),
    Autor VARCHAR2(100),
    Paginas NUMBER(4),
    PRIMARY KEY (ISBN)
);

CREATE TABLE EJEMPLARES (
    ID_Ejemplar NUMBER,
    ISBN VARCHAR2(13),
    Estado VARCHAR2(20),
    PRIMARY KEY (ID_Ejemplar),
    FOREIGN KEY (ISBN) REFERENCES LIBROS (ISBN) ON DELETE CASCADE
);

CREATE TABLE PRESTAMOS (
    Socio VARCHAR2(9),
    Ejemplar NUMBER,
    Fecha DATE,
    fecha_devolucion DATE,
    PRIMARY KEY (Socio, Ejemplar, Fecha),
    FOREIGN KEY (Socio) REFERENCES SOCIOS (DNI) ON DELETE CASCADE,
    FOREIGN KEY (Ejemplar) REFERENCES EJEMPLARES (ID_Ejemplar) ON DELETE CASCADE
);
```

## 2. Modificación de la tabla SOCIOS para incluir fechas y comparativa de tipos de datos.

```sql
ALTER TABLE SOCIOS ADD fecha_registro DATE;
ALTER TABLE SOCIOS ADD fecha_ultima_modificacion TIMESTAMP;
```

### Diferencias entre DATE y TIMESTAMP
El tipo DATE almacena la fecha y la hora exactas hasta el nivel de los segundos (año, mes, día, hora, minuto y segundo). 

El tipo TIMESTAMP es una extensión que incluye todo lo anterior pero añade fracciones de segundo (milisegundos, microsegundos) y permite almacenar información de la zona horaria.

### Diferencias entre CHAR y VARCHAR
CHAR almacena texto de longitud fija; si introduces un texto más corto que el límite, la base de datos rellena el espacio sobrante con espacios en blanco. 

VARCHAR (o VARCHAR2 en Oracle) almacena texto de longitud variable: solo ocupa el espacio de los caracteres que realmente se introducen.

**Justificación:** En este esquema, la mejor elección es definir el DNI como CHAR(9) porque siempre tiene exactamente 9 caracteres, asegurando un tamaño fijo. Para el resto de campos de texto (Nombre, Titulo, Autor, Email) se debe usar VARCHAR2 porque su longitud varía en cada registro, evitando así desperdiciar espacio de almacenamiento.

## 3. Restricciones para que el Estado del ejemplar solo admita valores específicos y el Email sea clave candidata.

```sql
ALTER TABLE EJEMPLARES ADD CONSTRAINT chk_estado_ejemplar CHECK (Estado IN ('Nuevo', 'Bueno', 'Deteriorado'));
ALTER TABLE SOCIOS ADD CONSTRAINT uq_email_socio UNIQUE (Email);
```

# 2: Objetos de la Base de Datos y Optimización

## 1. Creación de una vista llamada VISTA_PRESTAMOS_ACTIVOS que muestra el titulo del libro, el ID del ejemplar y el nombre del socio para préstamos sin devolver.

```sql
CREATE VIEW VISTA_PRESTAMOS_ACTIVOS AS
SELECT
    l.Titulo,
    e.ID_Ejemplar,
    s.Nombre
FROM PRESTAMOS p
JOIN EJEMPLARES e ON p.Ejemplar = e.ID_Ejemplar
JOIN LIBROS l ON e.ISBN = l.ISBN
JOIN SOCIOS s ON p.Socio = s.DNI
WHERE p.fecha_devolucion IS NULL;
```

## 2. Creación de un índice en la tabla LIBROS para agilizar la búsqueda por título.

```sql
CREATE INDEX idx_libros_titulo ON LIBROS (Titulo);
```

## 3. Definición de TABLESPACE en Oracle y justificación de la separación de archivos físicos.

### Definición:
Un TABLESPACE es una unidad lógica de almacenamiento en Oracle. Agrupa estructuras lógicas (como tablas, vistas o indices) y se relaciona directamente con uno o varios archivos físicos de datos (datafiles) en el disco duro del servidor.

### Buena práctica:
Separar las tablas y los índices en distintos tablespaces (y físicamente en distintos discos duros) permite a la base de datos realizar operaciones de entrada/salida (I/O) en paralelo. 

Mientras el sistema lee el indice en un disco, puede leer los datos de la tabla en el otro simultáneamente. Esto reduce los cuellos de botella en el disco, mejora drásticamente el rendimiento de las consultas y facilita las tareas de mantenimiento si un indice se corrompe.

# 3: Administración y Herramientas

## 1. Gestión de seguridad y privilegios (Creación de usuarios, asignación y revocación de permisos, y eliminación de usuarios).

```sql
CREATE USER super_admin IDENTIFIED BY 'contraseña';
GRANT ALL PRIVILEGES TO super_admin;

CREATE USER tecnico_inventario IDENTIFIED BY 'contraseña_tecnico';
GRANT SELECT, UPDATE ON EJEMPLARES TO tecnico_inventario;

REVOKE UPDATE ON EJEMPLARES FROM tecnico_inventario;

DROP USER tecnico_inventario;
```

## 2. Comandos necesarios para conectar a la consola de MySQL y Oracle.

```sql
mysql -u nombre_usuario -p
sqlplus nombre_usuario/contraseña
```

## 3. Identificación de las características de las herramientas gráficas y de consola.

| Herramienta | CLI (Consola) | Web | Escritorio (GUI) | Multi-extensión | Nativo Oracle | Nativo MySQL |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **MySQL Workbench** | | | X | | | X |
| **phpMyAdmin** | | X | | | | X |
| **SQL Developer** | | | X | | X | |
| **Visual Studio Code** | | | X | X | | |
| **mysql / sqlplus** | X | | | | X | X |

### HECHO POR JOSE AGUILERA ORTIZ