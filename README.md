# projectsDB

## QueriesSQL

### Objetivos específicos de aprendizaje

- Convertir el DER a modelo relacional implementable.
- Crear tablas con PK, FK, restricciones y tipos de datos adecuados.
- Insertar datos consistentes respetando reglas de integridad.
- Resolver consultas SQL: simple, filtro, join, group by y subconsulta.
- Aplicar operaciones de actualización y eliminación de forma segura.

### Entregables

- El script de las consultas SQL (todo hecho con lenguaje SQL, no de manera manual).
- Respuesta escrita a las preguntas de análisis.

### Adaptación del enunciado al modelo InmobiliariaDB

| Enunciado | Modelo InmobiliariaDB |
|---|---|
| Anfitriones | Usuarios (`"user"`) con rol `propietario` en `role_user` |
| Huéspedes | Usuarios (`"user"`) con rol `inquilino` en `role_user` |
| Habitaciones | `room` (se identifican por `address_room`) |
| Tipo de habitación | `capacity_room = 1` → Privada · `capacity_room > 1` → Compartida |
| Precio | `monthly_price_post` (precio mensual del anuncio en `post`) |
| Fecha de publicación | `datetime_post` |
| Reservas | `booking` |
| Pagos | `pay_method` + reservas con `pay_confirm_booking = TRUE` (importe = una mensualidad) |
| Servicios | Columnas booleanas de `room`, convertidas en filas con la vista `room_service` |
| Reseñas | `room_review` (ligadas a una reserva) |

---

## NIVEL I

### Parte 1. CREATE

#### 0. Crear las tablas del modelo

##### 0.1 Creación de base de datos

```sql
CREATE DATABASE "InmobiliariaDB"
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
```

##### 0.2 Limpieza previa (permite re-ejecutar el script)

```sql
DROP VIEW IF EXISTS room_service;
DROP TABLE IF EXISTS user_review, room_review, booking, pay_method, post,
                     enrollment, edu_center, room, neirborhood, town,
                     preference, role_user, role, "user", country CASCADE;
```

##### 0.3 Creación tabla "country"

```sql
CREATE TABLE country (
    id_contry    SERIAL PRIMARY KEY,
    name_country VARCHAR(100) NOT NULL UNIQUE
);
```

##### 0.4 Creación tabla "user"

```sql
CREATE TABLE "user" (
    id_user           SERIAL PRIMARY KEY,
    num_docu_user     VARCHAR(30)  NOT NULL UNIQUE,
    type_docu_user    VARCHAR(20)  NOT NULL CHECK (type_docu_user IN ('DNI', 'NIE', 'Pasaporte')),
    name_user         VARCHAR(100) NOT NULL,
    lastname_user     VARCHAR(100) NOT NULL,
    birthyear_user    INTEGER      NOT NULL CHECK (birthyear_user BETWEEN 1900 AND 2010),
    gender_user       VARCHAR(20)  NOT NULL,
    mail_user         VARCHAR(150) NOT NULL UNIQUE,
    phone_user        VARCHAR(20),
    country_resi_user INTEGER REFERENCES country (id_contry),
    status_user       VARCHAR(20)  NOT NULL DEFAULT 'activo'
                      CHECK (status_user IN ('activo', 'inactivo'))
);
```

##### 0.5 Creación tabla "role"

```sql
CREATE TABLE role (
    id_role   SERIAL PRIMARY KEY,
    name_role VARCHAR(50) NOT NULL UNIQUE
);
```

##### 0.6 Creación tabla "role_user"

```sql
CREATE TABLE role_user (
    id_roleuser      SERIAL PRIMARY KEY,
    id_user_roleuser INTEGER NOT NULL REFERENCES "user" (id_user) ON DELETE CASCADE,
    id_role_roleuser INTEGER NOT NULL REFERENCES role (id_role),
    UNIQUE (id_user_roleuser, id_role_roleuser)
);
```

##### 0.7 Creación tabla "preference"

```sql
CREATE TABLE preference (
    id_user_pref        INTEGER PRIMARY KEY REFERENCES "user" (id_user) ON DELETE CASCADE,
    pet_owner_pref      BOOLEAN,
    smoke_pref          BOOLEAN,
    aircon_pref         BOOLEAN,
    wifi_pref           BOOLEAN,
    bath_pref           BOOLEAN,
    closet_pref         BOOLEAN,
    kitchen_pref        BOOLEAN,
    balcony_pref        BOOLEAN,
    visit_allow_pref    BOOLEAN,
    utilities_incl_pref BOOLEAN
);
```

##### 0.8 Creación tabla "town"

```sql
CREATE TABLE town (
    id_town       SERIAL PRIMARY KEY,
    name_town     VARCHAR(100) NOT NULL,
    province_town VARCHAR(100) NOT NULL
);
```

##### 0.9 Creación tabla "neirborhood"

```sql
CREATE TABLE neirborhood (
    id_neibor      SERIAL PRIMARY KEY,
    id_town_neibor INTEGER NOT NULL REFERENCES town (id_town),
    name_neibor    VARCHAR(100) NOT NULL
);
```

##### 0.10 Creación tabla "room"

```sql
CREATE TABLE room (
    id_room             SERIAL PRIMARY KEY,
    id_owner_room       INTEGER NOT NULL REFERENCES "user" (id_user),
    address_room        VARCHAR(200) NOT NULL,
    postal_room         VARCHAR(20)  NOT NULL,
    floor_room          VARCHAR(10)  NOT NULL,
    town_room           INTEGER NOT NULL REFERENCES town (id_town),
    neibor_room         INTEGER NOT NULL REFERENCES neirborhood (id_neibor),
    size_room           NUMERIC(6, 2) NOT NULL CHECK (size_room > 0),
    bed_qty_room        INTEGER NOT NULL CHECK (bed_qty_room > 0),
    capacity_room       INTEGER NOT NULL CHECK (capacity_room > 0),
    gender_allow_room   VARCHAR(20) NOT NULL CHECK (gender_allow_room IN ('Mixto', 'Mujeres', 'Hombres')),
    closet_room         BOOLEAN NOT NULL,
    bath_room           BOOLEAN NOT NULL,
    balcony_room        BOOLEAN NOT NULL,
    aircon_room         BOOLEAN NOT NULL,
    wifi_room           BOOLEAN NOT NULL,
    kitchen_allow_room  BOOLEAN NOT NULL,
    visit_allow_room    BOOLEAN NOT NULL,
    smoke_room          BOOLEAN NOT NULL,
    pet_room            BOOLEAN NOT NULL,
    utilities_incl_room BOOLEAN NOT NULL,
    status_room         VARCHAR(20) NOT NULL CHECK (status_room IN ('disponible', 'ocupada'))
);
```

##### 0.11 Creación tabla "edu_center"

```sql
CREATE TABLE edu_center (
    id_edu      SERIAL PRIMARY KEY,
    town_edu    INTEGER NOT NULL REFERENCES town (id_town),
    name_edu    VARCHAR(150) NOT NULL,
    address_edu VARCHAR(200) NOT NULL,
    type_edu    VARCHAR(50)  NOT NULL
);
```

##### 0.12 Creación tabla "enrollment"

```sql
CREATE TABLE enrollment (
    id_enroll           SERIAL PRIMARY KEY,
    id_user_enroll      INTEGER NOT NULL REFERENCES "user" (id_user),
    id_educenter_enroll INTEGER NOT NULL REFERENCES edu_center (id_edu),
    attach_enroll       VARCHAR(255),
    exp_date_enroll     DATE
);
```

##### 0.13 Creación tabla "post"

```sql
CREATE TABLE post (
    id_post            SERIAL PRIMARY KEY,
    id_publisher_post  INTEGER NOT NULL REFERENCES "user" (id_user),
    id_room_post       INTEGER NOT NULL REFERENCES room (id_room),
    datetime_post      TIMESTAMP NOT NULL,
    min_period_post    INTEGER CHECK (min_period_post > 0),
    monthly_price_post NUMERIC(10, 2) NOT NULL CHECK (monthly_price_post > 0),
    deposit_price_post NUMERIC(10, 2) CHECK (deposit_price_post >= 0),
    status_post        VARCHAR(20) NOT NULL CHECK (status_post IN ('activa', 'pausada'))
);
```

##### 0.14 Creación tabla "pay_method"

```sql
CREATE TABLE pay_method (
    id_pay     SERIAL PRIMARY KEY,
    name_pay   VARCHAR(100) NOT NULL UNIQUE,
    type_pay   VARCHAR(50)  NOT NULL,
    status_pay VARCHAR(20)  NOT NULL CHECK (status_pay IN ('activo', 'inactivo'))
);
```

##### 0.15 Creación tabla "booking"

```sql
CREATE TABLE booking (
    id_booking          SERIAL PRIMARY KEY,
    id_user_booking     INTEGER NOT NULL REFERENCES "user" (id_user),
    id_post_booking     INTEGER NOT NULL REFERENCES post (id_post),
    datetime_booking    TIMESTAMP NOT NULL,
    pay_method_booking  INTEGER NOT NULL REFERENCES pay_method (id_pay),
    start_date_booking  DATE NOT NULL,
    end_date_booking    DATE NOT NULL,
    pay_confirm_booking BOOLEAN,
    status_booking      VARCHAR(20) NOT NULL
                        CHECK (status_booking IN ('pendiente', 'confirmada', 'cancelada')),
    CHECK (end_date_booking > start_date_booking)
);
```

##### 0.16 Creación tabla "room_review"

```sql
CREATE TABLE room_review (
    id_roomreview         SERIAL PRIMARY KEY,
    id_booking_roomreview INTEGER NOT NULL REFERENCES booking (id_booking),
    rate_room_roomreview  NUMERIC(2, 1) NOT NULL CHECK (rate_room_roomreview BETWEEN 1 AND 5),
    rate_owner_roomreview NUMERIC(2, 1) NOT NULL CHECK (rate_owner_roomreview BETWEEN 1 AND 5),
    desc_roomreview       TEXT
);
```

##### 0.17 Creación tabla "user_review"

```sql
CREATE TABLE user_review (
    id_userreview         SERIAL PRIMARY KEY,
    id_booking_userreview INTEGER NOT NULL REFERENCES booking (id_booking),
    rate_userreview       NUMERIC(2, 1) NOT NULL CHECK (rate_userreview BETWEEN 1 AND 5),
    desc_userreview       TEXT
);
```

##### 0.18 Creación vista "room_service"

Convierte las columnas booleanas de servicios de `room` en filas (una por habitación y servicio), para resolver las preguntas de servicios con `JOIN` / `LEFT JOIN`.

```sql
CREATE VIEW room_service AS
          SELECT id_room, 'Armario'            AS name_service FROM room WHERE closet_room
UNION ALL SELECT id_room, 'Baño privado'                       FROM room WHERE bath_room
UNION ALL SELECT id_room, 'Balcón'                             FROM room WHERE balcony_room
UNION ALL SELECT id_room, 'Aire acondicionado'                 FROM room WHERE aircon_room
UNION ALL SELECT id_room, 'WiFi'                               FROM room WHERE wifi_room;
```

#### 1. Insertar registros

##### Tabla "country"

```sql
INSERT INTO country (name_country) VALUES
('España'), ('Colombia'), ('Italia'), ('México');
```

##### Tabla "user"

```sql
INSERT INTO "user" (num_docu_user, type_docu_user, name_user, lastname_user, birthyear_user,
                    gender_user, mail_user, phone_user, country_resi_user, status_user) VALUES
('11111111A', 'DNI',       'Carlos', 'Ramírez',  1978, 'Hombre', 'carlos.ramirez@mail.com', '611111111', 1, 'activo'),
('22222222B', 'DNI',       'Laura',  'Gómez',    1985, 'Mujer',  'laura.gomez@mail.com',   '622222222', 1, 'activo'),
('33333333C', 'DNI',       'Luis',   'Martínez', 1990, 'Hombre', 'luis.martinez@mail.com', '633333333', 1, 'activo'),
('44444444D', 'DNI',       'Ana',    'Torres',   1982, 'Mujer',  'ana.torres@mail.com',    '644444444', 1, 'activo'),
('X1234567P', 'NIE',       'Paula',  'Ríos',     2003, 'Mujer',  'paula.rios@mail.com',    '655555555', 2, 'activo'),
('55555555E', 'DNI',       'María',  'López',    2004, 'Mujer',  'maria.lopez@mail.com',   '666666666', 1, 'activo'),
('YA9876543', 'Pasaporte', 'Mateo',  'Suárez',   2002, 'Hombre', 'mateo.suarez@mail.com',  '677777777', 4, 'activo'),
('Y7654321K', 'NIE',       'Andrés', 'Castro',   2001, 'Hombre', 'andres.castro@mail.com', '688888888', 2, 'activo'),
('66666666F', 'DNI',       'Sofía',  'Herrera',  2005, 'Mujer',  'sofia.herrera@mail.com', '699999999', 1, 'activo'),
('77777777G', 'DNI',       'Elena',  'Ruiz',     1988, 'Mujer',  'elena.ruiz@mail.com',    '600000000', 3, 'activo');
```

##### Tabla "role"

```sql
INSERT INTO role (name_role) VALUES
('admin'), ('propietario'), ('inquilino');
```

##### Tabla "role_user"

```sql
INSERT INTO role_user (id_user_roleuser, id_role_roleuser) VALUES
(1, 2), (2, 2), (3, 2), (4, 2),
(5, 3), (6, 3), (7, 3), (8, 3), (9, 3),
(10, 1), (10, 2);
```

##### Tabla "preference"

```sql
INSERT INTO preference (id_user_pref, pet_owner_pref, smoke_pref, aircon_pref, wifi_pref, bath_pref,
                        closet_pref, kitchen_pref, balcony_pref, visit_allow_pref, utilities_incl_pref) VALUES
(5, FALSE, FALSE, TRUE,  TRUE, TRUE,  TRUE, TRUE,  FALSE, TRUE,  TRUE),
(6, FALSE, FALSE, TRUE,  TRUE, FALSE, TRUE, TRUE,  TRUE,  FALSE, TRUE),
(7, TRUE,  FALSE, FALSE, TRUE, FALSE, TRUE, TRUE,  FALSE, TRUE,  FALSE),
(8, FALSE, TRUE,  TRUE,  TRUE, TRUE,  TRUE, FALSE, FALSE, TRUE,  TRUE),
(9, FALSE, FALSE, TRUE,  TRUE, TRUE,  TRUE, TRUE,  TRUE,  TRUE,  TRUE);
```

##### Tabla "town"

```sql
INSERT INTO town (name_town, province_town) VALUES
('Granada', 'Granada'), ('Madrid', 'Madrid'), ('Sevilla', 'Sevilla'),
('Valencia', 'Valencia'), ('León', 'León');
```

##### Tabla "neirborhood"

```sql
INSERT INTO neirborhood (id_town_neibor, name_neibor) VALUES
(1, 'Centro'), (1, 'Albaicín'), (2, 'Chamberí'), (2, 'Centro'),
(3, 'Triana'), (4, 'Ruzafa'), (5, 'Barrio Húmedo');
```

##### Tabla "room"

```sql
INSERT INTO room (id_owner_room, address_room, postal_room, floor_room, town_room, neibor_room,
                  size_room, bed_qty_room, capacity_room, gender_allow_room,
                  closet_room, bath_room, balcony_room, aircon_room, wifi_room,
                  kitchen_allow_room, visit_allow_room, smoke_room, pet_room, utilities_incl_room,
                  status_room) VALUES
(1, 'Calle Recogidas 12',  '18005', '2',    1, 1, 14.50, 1, 1, 'Mixto',   TRUE,  TRUE,  FALSE, TRUE,  TRUE,  TRUE,  TRUE,  FALSE, FALSE, TRUE,  'ocupada'),
(2, 'Calle Fuencarral 88', '28004', '3',    2, 3, 18.00, 1, 1, 'Mujeres', TRUE,  TRUE,  FALSE, TRUE,  TRUE,  TRUE,  FALSE, FALSE, FALSE, FALSE, 'ocupada'),
(1, 'Cuesta de Gomérez 5', '18009', '1',    1, 2, 12.00, 1, 1, 'Mixto',   TRUE,  FALSE, FALSE, FALSE, TRUE,  TRUE,  TRUE,  FALSE, TRUE,  TRUE,  'disponible'),
(3, 'Calle Betis 30',      '41010', 'Bajo', 3, 5, 20.00, 2, 2, 'Hombres', TRUE,  FALSE, FALSE, TRUE,  TRUE,  TRUE,  TRUE,  TRUE,  FALSE, FALSE, 'disponible'),
(2, 'Calle Sueca 45',      '46006', '4',    4, 6, 22.00, 2, 2, 'Mixto',   FALSE, FALSE, FALSE, FALSE, FALSE, TRUE,  TRUE,  FALSE, TRUE,  FALSE, 'ocupada'),
(3, 'Calle Ancha 7',       '24003', '2',    5, 7, 16.00, 2, 2, 'Mixto',   FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, 'disponible'),
(1, 'Calle Atocha 101',    '28012', '5',    2, 4, 25.00, 1, 1, 'Mixto',   TRUE,  TRUE,  FALSE, FALSE, TRUE,  TRUE,  TRUE,  FALSE, FALSE, TRUE,  'disponible');
```

##### Tabla "edu_center"

```sql
INSERT INTO edu_center (town_edu, name_edu, address_edu, type_edu) VALUES
(1, 'Universidad de Granada',   'Avenida del Hospicio s/n', 'Universidad'),
(3, 'Universidad de Sevilla',   'Calle San Fernando 4',     'Universidad'),
(4, 'Universitat de València',  'Avenida Blasco Ibáñez 13', 'Universidad'),
(2, 'Escuela de Idiomas Madrid','Calle Jesús Maestro 5',    'Escuela Oficial de Idiomas');
```

##### Tabla "enrollment"

```sql
INSERT INTO enrollment (id_user_enroll, id_educenter_enroll, attach_enroll, exp_date_enroll) VALUES
(5, 1, 'matricula_paula.pdf',  '2027-06-30'),
(6, 1, 'matricula_maria.pdf',  '2027-06-30'),
(7, 2, 'matricula_mateo.pdf',  '2027-06-30'),
(8, 3, 'matricula_andres.pdf', '2026-12-31'),
(9, 1, 'matricula_sofia.pdf',  '2027-06-30'),
(6, 4, 'matricula_idiomas_maria.pdf', '2027-05-31');
```

##### Tabla "post"

```sql
INSERT INTO post (id_publisher_post, id_room_post, datetime_post, min_period_post,
                  monthly_price_post, deposit_price_post, status_post) VALUES
(1, 1, '2023-06-01 10:00', 3, 380.00,  380.00, 'activa'),
(2, 2, '2024-02-15 09:30', 6, 550.00, 1100.00, 'activa'),
(1, 3, '2024-05-20 18:00', 1, 320.00,  320.00, 'activa'),
(3, 4, '2023-11-03 12:15', 3, 280.00,  280.00, 'activa'),
(2, 5, '2024-08-10 11:45', 6, 450.00,  450.00, 'activa'),
(1, 7, '2025-04-05 16:20', 3, 600.00,  600.00, 'pausada');
```

##### Tabla "pay_method"

```sql
INSERT INTO pay_method (name_pay, type_pay, status_pay) VALUES
('Tarjeta de crédito',     'tarjeta',       'activo'),
('Transferencia bancaria', 'transferencia', 'activo'),
('Bizum',                  'móvil',         'activo'),
('Apple Pay',              'móvil',         'activo'),
('Google Pay',             'móvil',         'inactivo');
```

##### Tabla "booking"

```sql
INSERT INTO booking (id_user_booking, id_post_booking, datetime_booking, pay_method_booking,
                     start_date_booking, end_date_booking, pay_confirm_booking, status_booking) VALUES
(5, 1, '2025-08-20 12:00', 1, '2025-09-01', '2026-06-30', TRUE,  'confirmada'),
(5, 2, '2026-09-15 17:30', 2, '2026-10-01', '2027-03-31', FALSE, 'pendiente'),
(6, 1, '2024-08-10 10:00', 3, '2024-09-01', '2025-06-30', TRUE,  'confirmada'),
(7, 3, '2026-09-20 20:10', 3, '2026-11-01', '2027-01-31', FALSE, 'pendiente'),
(8, 5, '2025-12-10 09:00', 1, '2026-01-01', '2026-12-31', TRUE,  'confirmada'),
(6, 2, '2025-08-01 13:40', 4, '2025-09-01', '2026-06-30', TRUE,  'confirmada'),
(7, 4, '2026-09-25 19:00', 2, '2026-10-15', '2027-01-15', FALSE, 'pendiente');
```

##### Tabla "room_review"

```sql
INSERT INTO room_review (id_booking_roomreview, rate_room_roomreview, rate_owner_roomreview, desc_roomreview) VALUES
(1, 5.0, 4.5, 'Muy luminosa y bien comunicada.'),
(3, 4.0, 4.0, 'Buena habitación, algo de ruido de la calle.'),
(6, 4.5, 5.0, 'La propietaria es muy atenta.'),
(5, 5.0, 4.5, 'Barrio con mucho ambiente, la recomiendo.');
```

##### Tabla "user_review"

```sql
INSERT INTO user_review (id_booking_userreview, rate_userreview, desc_userreview) VALUES
(1, 5.0, 'Inquilina responsable y puntual con los pagos.'),
(6, 4.5, 'Muy ordenada.');
```

### Parte 2. READ

#### 2. Todos los registros de anfitriones (usuarios con rol 'propietario')

```sql
SELECT u.*
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario';
```

#### 3. Todos los registros de huéspedes (usuarios con rol 'inquilino')

```sql
SELECT u.*
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'inquilino';
```

#### 4. Todos los registros de habitaciones

```sql
SELECT * FROM room;
```

#### 5. Todos los registros de reservas

```sql
SELECT * FROM booking;
```

#### 6. Todos los registros de pagos (métodos de pago)

```sql
SELECT * FROM pay_method;
```

#### 7. Nombre y email de los anfitriones

```sql
SELECT u.name_user, u.lastname_user, u.mail_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario';
```

#### 8. Dirección (título), tipo y ciudad de las habitaciones

```sql
SELECT ro.address_room,
       CASE WHEN ro.capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       t.name_town AS ciudad
FROM room ro
JOIN town t ON t.id_town = ro.town_room;
```

#### 9. Nombre y teléfono de los huéspedes

```sql
SELECT u.name_user, u.lastname_user, u.phone_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'inquilino';
```

### Parte 3. UPDATE (siempre con WHERE para no modificar toda la tabla)

#### 10. Actualizar el teléfono de un anfitrión

```sql
UPDATE "user"
SET phone_user = '622000111'
WHERE id_user = 2;
```

#### 11. Cambiar el estado de una reserva de 'pendiente' a 'confirmada'

```sql
UPDATE booking
SET status_booking = 'confirmada'
WHERE id_booking = 4 AND status_booking = 'pendiente';
```

#### 12. Actualizar el precio mensual de una habitación (en su anuncio)

```sql
UPDATE post
SET monthly_price_post = 340.00
WHERE id_post = 3;
```

#### 13. Cambiar el coste de un servicio de una habitación

En este modelo los servicios no tienen coste propio; lo que cambia el coste es si los suministros (luz, agua, internet) están incluidos en el precio.

```sql
UPDATE room
SET utilities_incl_room = TRUE
WHERE id_room = 2;
```

#### 14. Actualizar el monto de un pago (la fianza de un anuncio)

```sql
UPDATE post
SET deposit_price_post = 500.00
WHERE id_post = 5;
```

### Parte 4. DELETE (siempre con WHERE para no borrar toda la tabla)

#### 15. Eliminar un servicio asignado a una habitación

Los servicios son columnas de room, así que "quitar" un servicio es poner su columna a FALSE (la vista room_service deja de mostrarlo).

```sql
UPDATE room
SET aircon_room = FALSE
WHERE id_room = 4;
```

#### 16. Eliminar una reserva específica (una sin reseñas asociadas)

```sql
DELETE FROM booking
WHERE id_booking = 7;
```

#### 17. Intentar eliminar una habitación que tiene reservas y reseñas

El bloque DO captura el error para que el resto del script siga ejecutándose.

```sql
DO $$
BEGIN
    DELETE FROM room WHERE id_room = 1;
EXCEPTION WHEN foreign_key_violation THEN
    RAISE NOTICE 'No se puede eliminar la habitación: %', SQLERRM;
END $$;
```

> **Explicación:** PostgreSQL rechaza el borrado con un error de violación de clave foránea. La habitación 1 está referenciada por la tabla post (su anuncio), y ese anuncio a su vez por booking (reservas) y estas por room_review (reseñas). Si se borrara la habitación, esos registros quedarían apuntando a algo que no existe (registros huérfanos). Como las FK no tienen ON DELETE CASCADE, el comportamiento por defecto (NO ACTION) impide el borrado y protege la integridad referencial.

#### 18. Intentar eliminar un anfitrión que tiene habitaciones registradas

```sql
DO $$
BEGIN
    DELETE FROM "user" WHERE id_user = 1;
EXCEPTION WHEN foreign_key_violation THEN
    RAISE NOTICE 'No se puede eliminar el anfitrión: %', SQLERRM;
END $$;
```

> **Explicación:** ocurre lo mismo. Carlos Ramírez (id 1) es id_owner_room de varias habitaciones y publicador de anuncios. La FK room.id_owner_room impide borrarlo para que no queden habitaciones sin propietario. Habría que borrar o reasignar antes sus habitaciones (y todo lo que depende de ellas).

---

## NIVEL II

### Parte 5. Filtros con WHERE

#### 19. Habitaciones de tipo 'Privada' (capacidad para 1 persona)

```sql
SELECT * FROM room WHERE capacity_room = 1;
```

#### 20. Habitaciones de tipo 'Compartida' (capacidad para más de 1 persona)

```sql
SELECT * FROM room WHERE capacity_room > 1;
```

#### 21. Reservas en estado 'pendiente'

```sql
SELECT * FROM booking WHERE status_booking = 'pendiente';
```

#### 22. Reservas en estado 'confirmada'

```sql
SELECT * FROM booking WHERE status_booking = 'confirmada';
```

#### 23. Habitaciones con precio mensual mayor a 400 €

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
WHERE p.monthly_price_post > 400;
```

#### 24. Habitaciones publicadas después del 1 de enero de 2024

```sql
SELECT ro.address_room, p.datetime_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
WHERE p.datetime_post > '2024-01-01';
```

#### 25. Anfitriones cuyo nombre sea 'Carlos'

```sql
SELECT u.*
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario' AND u.name_user = 'Carlos';
```

### Parte 6. Búsquedas con LIKE

#### 26. Anfitriones cuyo nombre empiece por 'L'

```sql
SELECT u.name_user, u.lastname_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario' AND u.name_user LIKE 'L%';
```

#### 27. Huéspedes cuyo nombre empiece por 'M'

```sql
SELECT u.name_user, u.lastname_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'inquilino' AND u.name_user LIKE 'M%';
```

#### 28. Habitaciones cuyo barrio contenga 'Centro'

```sql
SELECT ro.address_room, n.name_neibor
FROM room ro
JOIN neirborhood n ON n.id_neibor = ro.neibor_room
WHERE n.name_neibor LIKE '%Centro%';
```

#### 29. Servicios cuyo nombre termine en 'Fi'

```sql
SELECT DISTINCT name_service FROM room_service WHERE name_service LIKE '%Fi';
```

#### 30. Habitaciones cuya ciudad contenga la letra 'a'

```sql
SELECT ro.address_room, t.name_town
FROM room ro
JOIN town t ON t.id_town = ro.town_room
WHERE t.name_town LIKE '%a%';
```

### Parte 7. Ordenamiento

#### 30 (bis). Habitaciones ordenadas alfabéticamente por dirección

```sql
SELECT * FROM room ORDER BY address_room ASC;
```

#### 31. Habitaciones de mayor a menor precio mensual

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
ORDER BY p.monthly_price_post DESC;
```

#### 32. Reservas por fecha de entrada, de la más próxima a la más lejana

```sql
SELECT * FROM booking ORDER BY start_date_booking ASC;
```

#### 33. Anfitriones ordenados por nombre descendente

```sql
SELECT u.name_user, u.lastname_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario'
ORDER BY u.name_user DESC;
```

### Parte 8. Rangos y listas

#### 34. Habitaciones con precio mensual entre 300 € y 500 €

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
WHERE p.monthly_price_post BETWEEN 300 AND 500;
```

#### 35. Habitaciones de tipo 'Privada' o 'Compartida' usando IN

```sql
SELECT *
FROM (
    SELECT address_room,
           CASE WHEN capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo
    FROM room
) AS habitaciones_con_tipo
WHERE tipo IN ('Privada', 'Compartida');
```

#### 36. Reservas en estado 'pendiente' o 'confirmada'

```sql
SELECT * FROM booking WHERE status_booking IN ('pendiente', 'confirmada');
```

#### 37. Habitaciones ubicadas en Madrid o Granada

```sql
SELECT ro.address_room, t.name_town
FROM room ro
JOIN town t ON t.id_town = ro.town_room
WHERE t.name_town IN ('Madrid', 'Granada');
```

---

## NIVEL III

### Parte 9. Relaciones con JOIN

#### 38. Cada habitación con el nombre de su anfitrión

```sql
SELECT ro.address_room, u.name_user || ' ' || u.lastname_user AS anfitrion
FROM room ro
JOIN "user" u ON u.id_user = ro.id_owner_room;
```

#### 39. Habitación, tipo y nombre del anfitrión

```sql
SELECT ro.address_room,
       CASE WHEN ro.capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       u.name_user || ' ' || u.lastname_user AS anfitrion
FROM room ro
JOIN "user" u ON u.id_user = ro.id_owner_room;
```

#### 40. Reservas con el nombre del huésped

```sql
SELECT b.id_booking, u.name_user || ' ' || u.lastname_user AS huesped,
       b.start_date_booking, b.end_date_booking, b.status_booking
FROM booking b
JOIN "user" u ON u.id_user = b.id_user_booking;
```

#### 41. Reservas con la habitación (reserva -> anuncio -> habitación)

```sql
SELECT b.id_booking, ro.address_room,
       b.start_date_booking, b.end_date_booking, b.status_booking
FROM booking b
JOIN post p  ON p.id_post  = b.id_post_booking
JOIN room ro ON ro.id_room = p.id_room_post;
```

#### 42. Reservas con id, huésped, habitación, entrada, salida y estado

```sql
SELECT b.id_booking,
       u.name_user || ' ' || u.lastname_user AS huesped,
       ro.address_room AS habitacion,
       b.start_date_booking, b.end_date_booking, b.status_booking
FROM booking b
JOIN "user" u ON u.id_user  = b.id_user_booking
JOIN post p   ON p.id_post  = b.id_post_booking
JOIN room ro  ON ro.id_room = p.id_room_post;
```

#### 43. Pagos realizados con el nombre del huésped y el método de pago

```sql
SELECT b.id_booking, u.name_user || ' ' || u.lastname_user AS huesped,
       pm.name_pay AS metodo_pago, p.monthly_price_post AS monto
FROM booking b
JOIN "user" u      ON u.id_user  = b.id_user_booking
JOIN pay_method pm ON pm.id_pay  = b.pay_method_booking
JOIN post p        ON p.id_post  = b.id_post_booking
WHERE b.pay_confirm_booking = TRUE;
```

#### 44. Servicios de cada habitación

```sql
SELECT ro.address_room, rs.name_service
FROM room ro
JOIN room_service rs ON rs.id_room = ro.id_room
ORDER BY ro.address_room;
```

#### 45. Habitación, servicio, suministros incluidos y disponibilidad

```sql
SELECT ro.address_room, rs.name_service,
       ro.utilities_incl_room AS suministros_incluidos,
       ro.status_room         AS disponibilidad
FROM room ro
JOIN room_service rs ON rs.id_room = ro.id_room
ORDER BY ro.address_room;
```

#### 46. Todas las habitaciones con su anfitrión y sus reseñas

(LEFT JOIN para que salgan también las habitaciones sin reseñas)

```sql
SELECT ro.address_room, u.name_user || ' ' || u.lastname_user AS anfitrion,
       rr.rate_room_roomreview, rr.desc_roomreview
FROM room ro
JOIN "user" u           ON u.id_user = ro.id_owner_room
LEFT JOIN post p        ON p.id_room_post = ro.id_room
LEFT JOIN booking b     ON b.id_post_booking = p.id_post
LEFT JOIN room_review rr ON rr.id_booking_roomreview = b.id_booking
ORDER BY ro.address_room;
```

#### 47. Reservas con huésped, anfitrión y habitación

```sql
SELECT b.id_booking,
       hu.name_user || ' ' || hu.lastname_user AS huesped,
       an.name_user || ' ' || an.lastname_user AS anfitrion,
       ro.address_room AS habitacion
FROM booking b
JOIN "user" hu ON hu.id_user = b.id_user_booking
JOIN post p    ON p.id_post  = b.id_post_booking
JOIN room ro   ON ro.id_room = p.id_room_post
JOIN "user" an ON an.id_user = ro.id_owner_room;
```

### Parte 10. Consultas de negocio con JOIN

#### 48. Habitaciones del anfitrión Carlos Ramírez

```sql
SELECT ro.address_room, t.name_town
FROM room ro
JOIN "user" u ON u.id_user = ro.id_owner_room
JOIN town t   ON t.id_town = ro.town_room
WHERE u.name_user = 'Carlos' AND u.lastname_user = 'Ramírez';
```

#### 49. Reservas del huésped Paula Ríos

```sql
SELECT b.id_booking, ro.address_room,
       b.start_date_booking, b.end_date_booking, b.status_booking
FROM booking b
JOIN "user" u ON u.id_user  = b.id_user_booking
JOIN post p   ON p.id_post  = b.id_post_booking
JOIN room ro  ON ro.id_room = p.id_room_post
WHERE u.name_user = 'Paula' AND u.lastname_user = 'Ríos';
```

#### 50. Servicios de la habitación de la Calle Recogidas 12

```sql
SELECT rs.name_service
FROM room ro
JOIN room_service rs ON rs.id_room = ro.id_room
WHERE ro.address_room = 'Calle Recogidas 12';
```

#### 51. Huésped(es) que reservaron la habitación de la Calle Fuencarral 88

```sql
SELECT u.name_user || ' ' || u.lastname_user AS huesped,
       b.start_date_booking, b.end_date_booking, b.status_booking
FROM booking b
JOIN "user" u ON u.id_user  = b.id_user_booking
JOIN post p   ON p.id_post  = b.id_post_booking
JOIN room ro  ON ro.id_room = p.id_room_post
WHERE ro.address_room = 'Calle Fuencarral 88';
```

#### 52. Habitaciones con al menos una reseña

```sql
SELECT DISTINCT ro.address_room
FROM room ro
JOIN post p         ON p.id_room_post = ro.id_room
JOIN booking b      ON b.id_post_booking = p.id_post
JOIN room_review rr ON rr.id_booking_roomreview = b.id_booking;
```

#### 53. Habitaciones sin reseñas (LEFT JOIN)

```sql
SELECT ro.address_room
FROM room ro
LEFT JOIN post p         ON p.id_room_post = ro.id_room
LEFT JOIN booking b      ON b.id_post_booking = p.id_post
LEFT JOIN room_review rr ON rr.id_booking_roomreview = b.id_booking
GROUP BY ro.id_room, ro.address_room
HAVING COUNT(rr.id_roomreview) = 0;
```

#### 54. Habitaciones con servicios

```sql
SELECT DISTINCT ro.address_room
FROM room ro
JOIN room_service rs ON rs.id_room = ro.id_room;
```

#### 55. Habitaciones sin servicios

```sql
SELECT ro.address_room
FROM room ro
LEFT JOIN room_service rs ON rs.id_room = ro.id_room
WHERE rs.id_room IS NULL;
```

### Parte 11. Funciones de agregación

#### 56. Número de anfitriones

```sql
SELECT COUNT(*) AS total_anfitriones
FROM role_user ru
JOIN role r ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'propietario';
```

#### 57. Número de huéspedes

```sql
SELECT COUNT(*) AS total_huespedes
FROM role_user ru
JOIN role r ON r.id_role = ru.id_role_roleuser
WHERE r.name_role = 'inquilino';
```

#### 58. Número de habitaciones publicadas (con anuncio activo)

```sql
SELECT COUNT(DISTINCT id_room_post) AS habitaciones_publicadas
FROM post
WHERE status_post = 'activa';
```

#### 59. Número de reservas

```sql
SELECT COUNT(*) AS total_reservas FROM booking;
```

#### 60. Precio mensual promedio

```sql
SELECT ROUND(AVG(monthly_price_post), 2) AS precio_promedio FROM post;
```

#### 61. Habitación más cara

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
ORDER BY p.monthly_price_post DESC
LIMIT 1;
```

#### 62. Habitación más económica

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
ORDER BY p.monthly_price_post ASC
LIMIT 1;
```

#### 63. Suma total de ingresos (reservas con pago confirmado)

```sql
SELECT SUM(p.monthly_price_post) AS ingresos_totales
FROM booking b
JOIN post p ON p.id_post = b.id_post_booking
WHERE b.pay_confirm_booking = TRUE;
```

### Parte 12. GROUP BY

#### 64. Habitaciones por tipo

```sql
SELECT CASE WHEN capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       COUNT(*) AS total_habitaciones
FROM room
GROUP BY tipo;
```

#### 65. Habitaciones por anfitrión (LEFT JOIN para incluir a quien tenga 0)

```sql
SELECT u.name_user || ' ' || u.lastname_user AS anfitrion,
       COUNT(ro.id_room) AS total_habitaciones
FROM "user" u
JOIN role_user ru  ON ru.id_user_roleuser = u.id_user
JOIN role r        ON r.id_role = ru.id_role_roleuser
LEFT JOIN room ro  ON ro.id_owner_room = u.id_user
WHERE r.name_role = 'propietario'
GROUP BY u.id_user, u.name_user, u.lastname_user
ORDER BY total_habitaciones DESC;
```

#### 66. Reservas por huésped

```sql
SELECT u.name_user || ' ' || u.lastname_user AS huesped,
       COUNT(b.id_booking) AS total_reservas
FROM "user" u
JOIN role_user ru    ON ru.id_user_roleuser = u.id_user
JOIN role r          ON r.id_role = ru.id_role_roleuser
LEFT JOIN booking b  ON b.id_user_booking = u.id_user
WHERE r.name_role = 'inquilino'
GROUP BY u.id_user, u.name_user, u.lastname_user;
```

#### 67. Reservas por habitación

```sql
SELECT ro.address_room, COUNT(b.id_booking) AS total_reservas
FROM room ro
LEFT JOIN post p    ON p.id_room_post = ro.id_room
LEFT JOIN booking b ON b.id_post_booking = p.id_post
GROUP BY ro.id_room, ro.address_room;
```

#### 68. Servicios por habitación

```sql
SELECT ro.address_room, COUNT(rs.name_service) AS total_servicios
FROM room ro
LEFT JOIN room_service rs ON rs.id_room = ro.id_room
GROUP BY ro.id_room, ro.address_room;
```

#### 69. Precio mensual promedio por ciudad

```sql
SELECT t.name_town AS ciudad, ROUND(AVG(p.monthly_price_post), 2) AS precio_promedio
FROM post p
JOIN room ro ON ro.id_room = p.id_room_post
JOIN town t  ON t.id_town  = ro.town_room
GROUP BY t.name_town;
```

### Parte 13. GROUP BY + HAVING

#### 70. Anfitriones con más de una habitación

```sql
SELECT u.name_user || ' ' || u.lastname_user AS anfitrion, COUNT(*) AS total_habitaciones
FROM "user" u
JOIN room ro ON ro.id_owner_room = u.id_user
GROUP BY u.id_user, u.name_user, u.lastname_user
HAVING COUNT(*) > 1;
```

#### 71. Tipos de habitación con más de una habitación

```sql
SELECT CASE WHEN capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       COUNT(*) AS total_habitaciones
FROM room
GROUP BY tipo
HAVING COUNT(*) > 1;
```

#### 72. Huéspedes con más de una reserva

```sql
SELECT u.name_user || ' ' || u.lastname_user AS huesped, COUNT(*) AS total_reservas
FROM "user" u
JOIN booking b ON b.id_user_booking = u.id_user
GROUP BY u.id_user, u.name_user, u.lastname_user
HAVING COUNT(*) > 1;
```

#### 73. Habitaciones con más de un servicio

```sql
SELECT ro.address_room, COUNT(*) AS total_servicios
FROM room ro
JOIN room_service rs ON rs.id_room = ro.id_room
GROUP BY ro.id_room, ro.address_room
HAVING COUNT(*) > 1;
```

---

## NIVEL IV (NO OBLIGATORIO)

### Parte 14. Subconsultas

#### 74. Habitaciones con precio mayor al promedio

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
WHERE p.monthly_price_post > (SELECT AVG(monthly_price_post) FROM post);
```

#### 75. Pagos con monto mayor al monto promedio pagado

```sql
SELECT b.id_booking, p.monthly_price_post AS monto
FROM booking b
JOIN post p ON p.id_post = b.id_post_booking
WHERE b.pay_confirm_booking = TRUE
  AND p.monthly_price_post > (
      SELECT AVG(p2.monthly_price_post)
      FROM booking b2
      JOIN post p2 ON p2.id_post = b2.id_post_booking
      WHERE b2.pay_confirm_booking = TRUE
  );
```

#### 76. Habitaciones con reservas

```sql
SELECT address_room
FROM room
WHERE id_room IN (
    SELECT p.id_room_post
    FROM post p
    JOIN booking b ON b.id_post_booking = p.id_post
);
```

#### 77. Habitaciones sin reservas

```sql
SELECT address_room
FROM room
WHERE id_room NOT IN (
    SELECT p.id_room_post
    FROM post p
    JOIN booking b ON b.id_post_booking = p.id_post
);
```

#### 78. Anfitriones con al menos una habitación

```sql
SELECT name_user, lastname_user
FROM "user"
WHERE id_user IN (SELECT id_owner_room FROM room);
```

#### 79. Huéspedes con reservas confirmadas

```sql
SELECT name_user, lastname_user
FROM "user"
WHERE id_user IN (SELECT id_user_booking FROM booking WHERE status_booking = 'confirmada');
```

#### 80. Habitación o habitaciones con más reservas (contempla empates)

```sql
SELECT ro.address_room, COUNT(*) AS total_reservas
FROM room ro
JOIN post p    ON p.id_room_post = ro.id_room
JOIN booking b ON b.id_post_booking = p.id_post
GROUP BY ro.id_room, ro.address_room
HAVING COUNT(*) = (
    SELECT MAX(total)
    FROM (
        SELECT COUNT(*) AS total
        FROM booking b2
        JOIN post p2 ON p2.id_post = b2.id_post_booking
        GROUP BY p2.id_room_post
    ) AS conteos
);
```

#### 81. Habitación más cara usando subconsulta

```sql
SELECT ro.address_room, p.monthly_price_post
FROM room ro
JOIN post p ON p.id_room_post = ro.id_room
WHERE p.monthly_price_post = (SELECT MAX(monthly_price_post) FROM post);
```

### Parte 15. LEFT JOIN y análisis de datos faltantes

#### 82. Anfitriones sin habitaciones

```sql
SELECT u.name_user, u.lastname_user
FROM "user" u
JOIN role_user ru ON ru.id_user_roleuser = u.id_user
JOIN role r       ON r.id_role = ru.id_role_roleuser
LEFT JOIN room ro ON ro.id_owner_room = u.id_user
WHERE r.name_role = 'propietario' AND ro.id_room IS NULL;
```

#### 83. Habitaciones sin reservas

```sql
SELECT ro.address_room
FROM room ro
LEFT JOIN post p    ON p.id_room_post = ro.id_room
LEFT JOIN booking b ON b.id_post_booking = p.id_post
WHERE b.id_booking IS NULL;
```

#### 84. Habitaciones sin servicios

```sql
SELECT ro.address_room
FROM room ro
LEFT JOIN room_service rs ON rs.id_room = ro.id_room
WHERE rs.id_room IS NULL;
```

#### 85. Huéspedes sin reservas

```sql
SELECT u.name_user, u.lastname_user
FROM "user" u
JOIN role_user ru   ON ru.id_user_roleuser = u.id_user
JOIN role r         ON r.id_role = ru.id_role_roleuser
LEFT JOIN booking b ON b.id_user_booking = u.id_user
WHERE r.name_role = 'inquilino' AND b.id_booking IS NULL;
```

#### 86. Habitaciones sin reseñas

```sql
SELECT ro.address_room
FROM room ro
LEFT JOIN post p         ON p.id_room_post = ro.id_room
LEFT JOIN booking b      ON b.id_post_booking = p.id_post
LEFT JOIN room_review rr ON rr.id_booking_roomreview = b.id_booking
GROUP BY ro.id_room, ro.address_room
HAVING COUNT(rr.id_roomreview) = 0;
```

### Parte 16. Consultas de reto

#### 87. Listado completo: habitación, tipo, anfitrión, huésped y servicio

(una habitación con 2 reservas y 4 servicios genera 2 x 4 = 8 filas)

```sql
SELECT ro.address_room AS habitacion,
       CASE WHEN ro.capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       an.name_user || ' ' || an.lastname_user AS anfitrion,
       hu.name_user || ' ' || hu.lastname_user AS huesped,
       rs.name_service AS servicio
FROM room ro
JOIN "user" an            ON an.id_user = ro.id_owner_room
LEFT JOIN post p          ON p.id_room_post = ro.id_room
LEFT JOIN booking b       ON b.id_post_booking = p.id_post
LEFT JOIN "user" hu       ON hu.id_user = b.id_user_booking
LEFT JOIN room_service rs ON rs.id_room = ro.id_room
ORDER BY ro.address_room;
```

#### 88. Habitaciones por tipo, solo tipos con 2 o más

```sql
SELECT CASE WHEN capacity_room = 1 THEN 'Privada' ELSE 'Compartida' END AS tipo,
       COUNT(*) AS total_habitaciones
FROM room
GROUP BY tipo
HAVING COUNT(*) >= 2;
```

#### 89. Anfitrión con más habitaciones (contempla empates)

```sql
SELECT u.name_user || ' ' || u.lastname_user AS anfitrion, COUNT(*) AS total_habitaciones
FROM "user" u
JOIN room ro ON ro.id_owner_room = u.id_user
GROUP BY u.id_user, u.name_user, u.lastname_user
HAVING COUNT(*) = (
    SELECT MAX(total)
    FROM (SELECT COUNT(*) AS total FROM room GROUP BY id_owner_room) AS conteos
);
```

#### 90. Habitación con más reservas

(FETCH FIRST ... WITH TIES devuelve también las empatadas)

```sql
SELECT ro.address_room, COUNT(*) AS total_reservas
FROM room ro
JOIN post p    ON p.id_room_post = ro.id_room
JOIN booking b ON b.id_post_booking = p.id_post
GROUP BY ro.id_room, ro.address_room
ORDER BY total_reservas DESC
FETCH FIRST 1 ROWS WITH TIES;
```

#### 91. Huéspedes ordenados por cantidad de reservas (mayor a menor)

```sql
SELECT u.name_user || ' ' || u.lastname_user AS huesped,
       COUNT(b.id_booking) AS total_reservas
FROM "user" u
JOIN role_user ru   ON ru.id_user_roleuser = u.id_user
JOIN role r         ON r.id_role = ru.id_role_roleuser
LEFT JOIN booking b ON b.id_user_booking = u.id_user
WHERE r.name_role = 'inquilino'
GROUP BY u.id_user, u.name_user, u.lastname_user
ORDER BY total_reservas DESC;
```

#### 92. Habitaciones con reservas y con servicios

```sql
SELECT address_room
FROM room
WHERE id_room IN (SELECT p.id_room_post FROM post p JOIN booking b ON b.id_post_booking = p.id_post)
  AND id_room IN (SELECT id_room FROM room_service);
```

#### 93. Habitaciones con reservas pero sin servicios

```sql
SELECT address_room
FROM room
WHERE id_room IN (SELECT p.id_room_post FROM post p JOIN booking b ON b.id_post_booking = p.id_post)
  AND id_room NOT IN (SELECT id_room FROM room_service);
```

#### 94. Servicios que ninguna habitación tiene

```sql
SELECT s.name_service
FROM (VALUES ('Armario'), ('Baño privado'), ('Balcón'), ('Aire acondicionado'), ('WiFi'))
     AS s (name_service)
LEFT JOIN room_service rs ON rs.name_service = s.name_service
WHERE rs.id_room IS NULL;
```

#### 95. Ingreso total por habitación (pagos confirmados de sus reservas)

```sql
SELECT ro.address_room,
       COALESCE(SUM(p.monthly_price_post) FILTER (WHERE b.pay_confirm_booking), 0) AS ingreso_total
FROM room ro
LEFT JOIN post p    ON p.id_room_post = ro.id_room
LEFT JOIN booking b ON b.id_post_booking = p.id_post
GROUP BY ro.id_room, ro.address_room
ORDER BY ingreso_total DESC;
```

#### 96. Habitación con la reserva de mayor valor pagado

```sql
SELECT ro.address_room, b.id_booking, p.monthly_price_post AS monto
FROM booking b
JOIN post p  ON p.id_post  = b.id_post_booking
JOIN room ro ON ro.id_room = p.id_room_post
WHERE b.pay_confirm_booking = TRUE
  AND p.monthly_price_post = (
      SELECT MAX(p2.monthly_price_post)
      FROM booking b2
      JOIN post p2 ON p2.id_post = b2.id_post_booking
      WHERE b2.pay_confirm_booking = TRUE
  );
```

