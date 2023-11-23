# Base de datos PostgreSQL

La base de datos se construye con un Dockerfile y para correrlo se usa el comando:

```bash
docker build -t database/Dockerfile db_skymanage .
```

Para correr la base de datos se usa el comando:

```bash
docker run -d -p 5432:5432 --name db_skymanage db_skymanage
```