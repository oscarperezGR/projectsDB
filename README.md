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
```sql

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

5. Creacion tabla "TOWN"
```sql
CREATE TABLE public."TOWN"
(
    id_town serial NOT NULL,
    name_town character varying(30) NOT NULL,
    province_town character varying(30) NOT NULL,
    PRIMARY KEY (id_town)
);
```
6. Creacion tabla "NEIGHBORHOOD"
```sql
CREATE TABLE "NEIGHBORHOOD"
(
    id_neighbor serial NOT NULL,
    id_town_neighbor integer NOT NULL,
    name_neighbor character varying(100) NOT NULL,
    CONSTRAINT neighborhood_pkey PRIMARY KEY (id_neighbor)
);
```
7. Creacion tabla "EDU_CENTER"
```sql
CREATE TABLE "EDU_CENTER"
(
    id_edu serial NOT NULL,
    town_edu integer NOT NULL,
    name_edu character varying(150) NOT NULL,
    address_edu character varying(200) NOT NULL,
    type_edu character varying(50) NOT NULL,
    CONSTRAINT edu_center_pkey PRIMARY KEY (id_edu)
);
```
8. Creacion tabla "ENROLLMENT"
```sql
CREATE TABLE "ENROLLMENT"
(
    id_enroll serial NOT NULL,
    id_user_enroll integer NOT NULL,
    id_educenter_enroll integer NOT NULL,
    attach_enroll character varying(255),
    exp_date_enroll date,
    CONSTRAINT enrollment_pkey PRIMARY KEY (id_enroll)
);
```
9. Creacion tabla "POST"
```sql
CREATE TABLE "POST"
(
    id_post serial NOT NULL,
    id_publisher_post integer NOT NULL,
    id_room_post integer NOT NULL,
    datetime_post timestamp NOT NULL,
    min_period_post integer,
    monthly_price_post numeric(10,2) NOT NULL,
    deposit_price_post numeric(10,2),
    status_post character varying(20) NOT NULL,
    CONSTRAINT post_pkey PRIMARY KEY (id_post)
);
```

10. Creacion tabla "BOOKING"
```sql
CREATE TABLE "BOOKING"
(
    id_booking serial NOT NULL,
    id_user_booking integer NOT NULL,
    id_post_booking integer NOT NULL,
    datetime_booking timestamp NOT NULL,
    pay_method_booking integer NOT NULL,
    start_date_booking date NOT NULL,
    end_date_booking date NOT NULL,
    pay_confirm_booking boolean,
    status_booking character varying(20) NOT NULL,
    CONSTRAINT booking_pkey PRIMARY KEY (id_booking)
);
```
11. Creacion tabla "COUNTRY"
```sql
CREATE TABLE "COUNTRY"
(
    id_country serial NOT NULL,
    name_country character varying(100) NOT NULL,
    CONSTRAINT country_pkey PRIMARY KEY (id_contry)
);
```

12. Creacion tabla "PAY_METHOD"
```sql
CREATE TABLE "PAY_METHOD"
(
    id_pay serial NOT NULL,
    name_pay character varying(50) NOT NULL,
    type_pay character varying(50) NOT NULL,
    status_pay boolean NOT NULL,
    CONSTRAINT pay_method_pkey PRIMARY KEY (id_pay)
);
```
13. Creacion tabla "ROOM_REVIEW"
```sql
CREATE TABLE "ROOM_REVIEW"
(
    id_roomreview serial NOT NULL,
    id_booking_roomreview integer NOT NULL,
    rate_room_roomreview integer NOT NULL,
    rate_owner_roomreview integer NOT NULL,
    desc_roomreview character varying(500),
    CONSTRAINT room_review_pkey PRIMARY KEY (id_roomreview)
);
```
14. Creacion tabla "USER_REVIEW"
```sql
CREATE TABLE "USER_REVIEW"
(
    id_userreview serial NOT NULL,
    id_booking_userreview integer NOT NULL,
    rate_userreview integer NOT NULL,
    desc_userreview character varying(500),
    CONSTRAINT user_review_pkey PRIMARY KEY (id_userreview)
);
```


