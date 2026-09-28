# Semana 7 - Base de Datos: Implementación, Poblamiento y Consulta del Modelo Relacional (Holding Carpenter SPA)

Actividad sumativa individual desarrollada con **Oracle SQL Developer** y **SQL Developer Data Modeler**. Se implementa un Modelo Relacional normalizado mediante sentencias DDL (creación de tablas y restricciones), se incorporan nuevas reglas de negocio con `ALTER TABLE`, se pueblan las tablas usando `IDENTITY` y objetos `SEQUENCE`, y finalmente se generan informes con la sentencia `SELECT`.

## Contexto de negocio

El **Holding Carpenter SPA** agrupa a un conjunto de empresas y necesita una base de datos para administrar la información de su personal: datos personales, empresa en la que trabaja, domicilio (comuna y región), estado civil, género, títulos obtenidos, idiomas que domina y la jefatura directa de cada trabajador. A partir del Modelo Relacional entregado se construyó el script completo, organizado en cuatro casos:

| Caso | Descripción |
|------|-------------|
| **Caso 1** | Implementación del modelo: creación de tablas (de las más fuertes a las más débiles) con sus restricciones PK, FK y UN. |
| **Caso 2** | Modificación del modelo con `ALTER TABLE` para incorporar nuevas reglas de negocio (UN y CK). |
| **Caso 3** | Poblamiento de las tablas `REGION`, `COMUNA`, `IDIOMA` y `COMPANIA` usando `IDENTITY` y `SEQUENCE`. |
| **Caso 4** | Recuperación de datos: dos informes de simulación de renta promedio. |

## Contenido del repositorio

| Archivo / carpeta | Descripción |
|-------------------|-------------|
| `Semana7_Basededatos_Gabrielurra.sql` | Script completo (Casos 1 al 4), listo para ejecutar con **F5** en SQL Developer. |
| `Semana7_Basededatos_Gabrielurra.dmd` | Diseño del modelo en SQL Developer Data Modeler. |
| `Semana7_Basededatos_Gabrielurra/` | Carpeta de datos del diseño (necesaria para abrir el `.dmd`). |

## Entidades identificadas

Se implementaron **10 tablas**: `REGION`, `IDIOMA`, `ESTADO_CIVIL`, `GENERO` y `TITULO` como tablas catálogo (fuertes); `COMUNA`, que depende de `REGION` mediante una clave primaria compuesta; `COMPANIA`, que representa a las empresas del holding; `PERSONAL` como entidad central, con una relación recursiva para identificar a su encargado; y `TITULACION` y `DOMINIO` como entidades asociativas que resuelven las relaciones N:M entre `PERSONAL`–`TITULO` y `PERSONAL`–`IDIOMA`.

### REGION
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_region | NUMBER(2) IDENTITY (inicia en 7, incrementa de 2 en 2) | PK |
| nombre_region | VARCHAR2(25) | Obligatorio |

### IDIOMA
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_idioma | NUMBER(3) IDENTITY (inicia en 25, incrementa de 3 en 3) | PK |
| nombre_idioma | VARCHAR2(30) | Obligatorio |

### ESTADO_CIVIL
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_estado_civil | VARCHAR2(2) | PK |
| descripcion_est_civil | VARCHAR2(25) | Obligatorio |

### GENERO
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_genero | VARCHAR2(3) | PK |
| descripcion_genero | VARCHAR2(25) | Obligatorio |

### TITULO
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_titulo | VARCHAR2(3) | PK |
| descripcion_titulo | VARCHAR2(60) | Obligatorio |

### COMUNA
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_comuna | NUMBER(5) — poblado con `SEQ_COMUNA` (inicia en 1101, incrementa de 6 en 6) | PK |
| comuna_nombre | VARCHAR2(25) | Obligatorio |
| cod_region | NUMBER(2) | PK / FK → REGION |

### COMPANIA
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_empresa | NUMBER(2) — poblado con `SEQ_COMPANIA` (inicia en 10, incrementa de 5 en 5) | PK |
| nombre_empresa | VARCHAR2(25) | Obligatorio (único) |
| calle | VARCHAR2(50) | Obligatorio |
| numeracion | NUMBER(5) | Obligatorio |
| renta_promedio | NUMBER(10) | Obligatorio |
| pct_aumento | NUMBER(4,3) | Opcional |
| cod_comuna | NUMBER(5) | FK → COMUNA |
| cod_region | NUMBER(2) | FK → COMUNA |

### PERSONAL
| Atributo | Tipo | Clave |
|----------|------|-------|
| rut_persona | NUMBER(8) | PK |
| dv_persona | CHAR(1) — CHECK (0-9, K) | Obligatorio |
| primer_nombre | VARCHAR2(25) | Obligatorio |
| segundo_nombre | VARCHAR2(25) | Opcional |
| primer_apellido | VARCHAR2(25) | Obligatorio |
| segundo_apellido | VARCHAR2(25) | Obligatorio |
| fecha_contratacion | DATE | Obligatorio |
| fecha_nacimiento | DATE | Obligatorio |
| email | VARCHAR2(100) | Opcional (único) |
| calle | VARCHAR2(50) | Obligatorio |
| numeracion | NUMBER(5) | Obligatorio |
| sueldo | NUMBER(8) — CHECK (>= 450.000) | Obligatorio |
| cod_comuna | NUMBER(5) | FK → COMUNA |
| cod_region | NUMBER(2) | FK → COMUNA |
| cod_genero | VARCHAR2(3) | FK → GENERO |
| cod_estado_civil | VARCHAR2(2) | FK → ESTADO_CIVIL |
| cod_empresa | NUMBER(2) | FK → COMPANIA |
| encargado_rut | NUMBER(8) | FK → PERSONAL (recursiva, opcional) |

### TITULACION (entidad asociativa)
| Atributo | Tipo | Clave |
|----------|------|-------|
| cod_titulo | VARCHAR2(3) | PK / FK → TITULO |
| persona_rut | NUMBER(8) | PK / FK → PERSONAL |
| fecha_titulacion | DATE | Obligatorio |

### DOMINIO (entidad asociativa)
| Atributo | Tipo | Clave |
|----------|------|-------|
| id_idioma | NUMBER(3) | PK / FK → IDIOMA |
| persona_rut | NUMBER(8) | PK / FK → PERSONAL |
| nivel | VARCHAR2(25) | Obligatorio |

## Relaciones

- **REGION (1,1) — COMUNA (0,N):** una región puede tener cero o muchas comunas; toda comuna pertenece exactamente a una región. Es una relación **identificadora**, ya que `cod_region` forma parte de la PK de `COMUNA`.
- **COMUNA (1,1) — COMPANIA (0,N):** una comuna puede tener cero o muchas empresas; toda empresa está ubicada en exactamente una comuna (FK compuesta `cod_comuna`, `cod_region`).
- **COMUNA (1,1) — PERSONAL (0,N):** una comuna puede tener cero o muchos trabajadores domiciliados en ella; todo trabajador tiene registrada exactamente una comuna.
- **COMPANIA (1,1) — PERSONAL (0,N):** una empresa puede tener cero o muchos trabajadores; todo trabajador pertenece exactamente a una empresa.
- **GENERO (0,1) — PERSONAL (0,N):** un género puede estar asociado a cero o muchos trabajadores.
- **ESTADO_CIVIL (0,1) — PERSONAL (0,N):** un estado civil puede estar asociado a cero o muchos trabajadores.
- **PERSONAL (0,1) — PERSONAL (0,N):** relación **recursiva**; un trabajador puede ser encargado de cero o muchos trabajadores, y un trabajador puede tener cero o un encargado.
- **PERSONAL (0,N) — TITULO (0,N):** relación N:M resuelta mediante la entidad asociativa `TITULACION`, que además registra la fecha de titulación.
- **PERSONAL (0,N) — IDIOMA (0,N):** relación N:M resuelta mediante la entidad asociativa `DOMINIO`, que además registra el nivel de dominio del idioma.

## Caso 2: Reglas de negocio agregadas con ALTER TABLE

| Restricción | Tipo | Regla |
|-------------|------|-------|
| `PERSONAL_UN_EMAIL` | UNIQUE | El email es opcional, pero no se puede repetir. |
| `PERSONAL_CK_DV` | CHECK | El dígito verificador debe ser 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 o 'K'. |
| `PERSONAL_CK_SUELDO` | CHECK | El sueldo mínimo del personal es de $450.000. |

## Caso 3: Poblamiento

El script respeta el orden de dependencia (de las tablas fuertes a las más débiles):

1. **REGION** — `IDENTITY` genera los identificadores 7, 9 y 11.
2. **COMUNA** — `SEQ_COMUNA` genera 1101 (Arica), 1107 (Santiago) y 1113 (Temuco).
3. **IDIOMA** — `IDENTITY` genera 25, 28, 31, 34 y 37.
4. **COMPANIA** — `SEQ_COMPANIA` genera los identificadores del 10 al 55 (10 empresas).

## Caso 4: Informes

### Informe 1 — Simulación de Renta Promedio
Lista todas las empresas del holding con su nombre, dirección completa (`calle || ' ' || numeracion`), renta promedio y la simulación de renta aplicando su porcentaje de aumento: `ROUND(renta_promedio * (1 + pct_aumento))`. Se ordena por renta promedio descendente y, en caso de empate, por nombre de empresa ascendente.

### Informe 2 — Nueva simulación renta promedio
Suma un 15% adicional al porcentaje registrado (`pct_aumento + 0.15`) y calcula el monto del aumento: `ROUND(renta_promedio * (pct_aumento + 0.15))`. Se ordena por renta promedio ascendente y luego por nombre de empresa descendente. Las empresas sin porcentaje registrado (`NULL`) muestran `(null)` en ambas columnas calculadas.

## Consideraciones de diseño

- Todas las restricciones tienen un **nombre representativo** según la tabla y su tipo (`_PK`, `_FK_`, `_UN_`, `_CK_`).
- La columna `sueldo` se definió como `NUMBER(8)`: con la precisión `NUMBER(5)` que se aprecia en el diagrama no sería posible almacenar el sueldo mínimo de $450.000 exigido en el Caso 2.
- El script comienza con un bloque PL/SQL que elimina las tablas y secuencias si ya existen, lo que permite ejecutarlo varias veces sin errores.
