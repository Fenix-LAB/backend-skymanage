# Tablas
Para mi la tabla de rutas es la mas importante, ya que es la que contiene la informacion de los vuelos, y es la que se va a consultar mas frecuentemente.

La tabla de rutas se ve asi:

```bash
CREATE TABLE routes (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    origin VARCHAR(3) NOT NULL,
    datetime_origin TIMESTAMP NOT NULL,
    destination VARCHAR(3) NOT NULL,
    datetime_destination TIMESTAMP NOT NULL,
    duration_travel INTEGER NOT NULL,
    duration INTEGER NOT NULL
    airplane_id INTEGER REFERENCES airplanes(id)
    imprevistos VARCHAR(100)
    language_destination VARCHAR(3) NOT NULL
    is_active BOOLEAN NOT NULL # Para saber si el vuelo esta activo o no depender de imprevistos y demas factores
    delay BOOLEAN NOT NULL # Para saber si el vuelo esta demorado o no
);
```

La tabla de aviones se ve asi:

```bash
CREATE TABLE airplanes (
    id SERIAL PRIMARY KEY,
    route_id INTEGER REFERENCES routes(id),
    name VARCHAR(100) NOT NULL,
    model VARCHAR(100) NOT NULL,
    capacity_passengers INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL # Para saber si el avion esta disponible para volar
    capacity_combustible INTEGER NOT NULL
    # pilot_id INTEGER REFERENCES pilots(id)
);
```

La tabla de pilotos se ve asi:

```bash
CREATE TABLE pilots (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    lastname VARCHAR(100) NOT NULL,
    age INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL # Para saber si el piloto esta disponible para volar
    languages VARCHAR(3) NOT NULL
    airplane_id INTEGER REFERENCES airplanes(id)
);
```

La tabla de usuarios se ve asi:

```bash
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    password VARCHAR(100) NOT NULL,
    is_active BOOLEAN NOT NULL # Para saber si el usuario esta activo o no
    incidents VARCHAR(100) NOT NULL # Para notificarle al usuario si hay algun incidente en su vuelo
    routes_id INTEGER REFERENCES routes(id)
    available_to_travel BOOLEAN NOT NULL # Para saber si el usuario esta disponible para viajar
    delay BOOLEAN NOT NULL # Para saber si el usuario esta demorado o no
);
```