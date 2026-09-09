# projectsDB
QueriesSQL
Objetivos específicos de aprendizaje
Convertir el DER a modelo relacional implementable.
Crear tablas con PK, FK, restricciones y tipos de datos adecuados.
Insertar datos consistentes respetando reglas de integridad.
Resolver consultas SQL: simple, filtro, join, group by y subconsulta.
Aplicar operaciones de actualización y eliminación de forma segura.


El script de sus consultas SQL (todo debe hacerse con lenguaje SQL, no de manera manual)
Respuesta escrita a las preguntas de análisis.

1.Creacion de base de datos

```sql
CREATE DATABASE "InmobiliariaDB"
(
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False
);
```

2. Creacion tabla "USER"

```sql
CREATE TABLE "USER"
(
    id_user serial NOT NULL,
    num_docu_user character varying(30) NOT NULL,
    type_docu_user character varying(20) NOT NULL,
    name_user character varying(100) NOT NULL,
    lastname_user character varying(100) NOT NULL,
    birthyear_user integer NOT NULL,
    gender_user character varying(20) NOT NULL,
    mail_user character varying(150) NOT NULL,
    phone_user character varying(20),
    country_resi_user integer,
    status_user boolean NOT NULL,
    CONSTRAINT user_pkey PRIMARY KEY (id_user),
    CONSTRAINT user_mail_user_key UNIQUE (mail_user)
);
```


3. Creacion tabla "ROLE_USER"

```sql
CREATE TABLE "ROLE_USER"
(
    id_user_roleuser integer NOT NULL,
    id_role_roleuser integer NOT NULL,
    CONSTRAINT role_user_pkey PRIMARY KEY (id_user_roleuser,id_role_roleuser)
);
```

4. Creacion tabla "ROLE"

```sql
CREATE TABLE "ROLE"
(
    id_role serial NOT NULL,
    name_role character varying(20) NOT NULL,
    CONSTRAINT role_pkey PRIMARY KEY (id_role)
);
```

5. Creacion tabla "ROOM"
````sql

CREATE TABLE  "ROOM"
(
    id_room serial NOT NULL,
    id_owner_room integer NOT NULL,
    town_room integer NOT NULL,
    neibor_room integer NOT NULL,
    address_room character varying(50) NOT NULL,
    postal_room integer NOT NULL,
    floor_room integer NOT NULL,
    size_room integer NOT NULL,
    bed_qty_room integer NOT NULL,
    capacity_room integer NOT NULL,
    gender_allowed_room character varying(20) NOT NULL,
    closet_room integer NOT NULL,
    bath_room boolean NOT NULL,
    pivaty_bath_room boolean NOT NULL,
    balcony_room boolean NOT NULL,
    aircon_room boolean NOT NULL,
    wifi_room boolean NOT NULL,
    kitchen_allowed_room boolean NOT NULL,
    visit_allowed_room boolean NOT NULL,
    smoke_room boolean NOT NULL,
    pet_room boolean NOT NULL,
    utilities_incl_room boolean NOT NULL,
    status_room character varying(20) NOT NULL,
    PRIMARY KEY (id_room)
);
```