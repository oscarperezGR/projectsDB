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



2. ```sql
'''CREATE DATABASE "InmobiliariaDB"
(
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
);

