# Arquitectura del backend

#### El Negocio:
"SkyManage" es una aerolínea que opera en rutas internacionales y enfrenta desafíos
únicos en la gestión y asignación de sus recursos. Con una flota variada y operaciones
en múltiples zonas horarias y climáticas, SkyManage busca soluciones tecnológicas para
mejorar su eficiencia operativa.

#### Problemática Central:
SkyManage necesita un sistema que ayude a manejar de manera eficiente las
asignaciones de tripulación y aviones, especialmente en situaciones de cambios
imprevistos como mal tiempo, problemas técnicos, o cambios en la disponibilidad de la
tripulación

La arquitectura de la solución y organización de carpetas

## Arquitectura
Se creara una API Rest, una base de datos relacional.

### 1 Base de datos.
Yo unsaria docker para crear la base de datos, y la base de datos que usaria seria postgresql.

Cadena de conexión:
```bash
postgresql://postgres:postgres@localhost:5432/skymanage

host = "aws-us-east-1-portal.6.dblayer.com"
port = "5432"
user = "user_name"
password = "user_password"
db_name = "db_name"

```
Las credenciales de la base se implementan a traves de variables de entorno.

### API Rest

La API Rest se creara con FastAPI, y se usara el ORM SQLAlchemy para la conexión con la base de datos.

#### Carpetas

```bash
API REST/
│
├── app/
│ ├── config/
│ │ ├── db_config.py
│ │ ├── session.py
│ │ └── ...
│ │
│ ├── models/
│ │ ├── models.py
│ │ ├── responses_models.py
│ │ └── ...
│ │
│ ├── routes/
│ │ ├── router
│ │ │ ├── router1.py
│ │ │ ├── router2.py
│ │ │ └── ...
│ │ ├── api.py
│ │ └── ...
│ │
│ ├── schemas/
│ │ ├── schemas.py
│ │ ├── responses_schemas.py
│ │ └── ...
│ │
│ ├── utils/
│ │ ├── security.py
│ │ ├── validators.py
│ │ └── ...
│ │
│ ├── services/
│ │ ├── service1.py
│ │ ├── service2.py
│ │ └── ...
│ │
│ └── server.py
│
├── database/
│ ├── Dockerfile
│ ├── init_db.sh
│ └── ...
│
├── README.md
└── ...
```

### 2 Conexion con la base de datos

La conexion se realiza de forma asincrona usando el ORM SQLAlchemy, y se crea la sesion de la base de datos.

### 3 Crear tablas

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

### 4 Crear Routing de la API

Tenemos el conjunto de las urls de los endpoints de la API


#### CRUDS de los servicios 

##### CRUD de los usuarios
POST: /users
GET: /users
GET: /users/{id}
PUT: /users/{id}
DELETE: /users/{id}

##### CRUD de los aviones
POST: /airplanes
GET: /airplanes
GET: /airplanes/{id}
PUT: /airplanes/{id}
DELETE: /airplanes/{id}

##### CRUD de los pilotos
POST: /pilots
GET: /pilots
GET: /pilots/{id}
PUT: /pilots/{id}
DELETE: /pilots/{id}

##### CRUD de las rutas
POST: /routes
GET: /routes
GET: /routes/{id}
PUT: /routes/{id}
DELETE: /routes/{id}

#### Servicios

##### Notificar si tiene retraso

##### Notificar si tiene un inconveniente

##### Notificar si esta disponible para el vuelo

##### Notificar si el vuelo esta activo

##### Notificar si el vuelo esta demorado

##### Notificar si el avion esta disponible

##### Notificar si el piloto esta disponible

### 5 Crear los modelos de Request y Response

#### Request

```bash
class UserCreate(BaseModel):
    username: str
    password: str
    is_active: bool
    incidents: str
    routes_id: int
    available_to_travel: bool
    delay: bool
```

#### Response

```bash

status_code: int
message: str
data: UserResponse

class UserResponse(BaseModel):
    id: int
    username: str
    is_active: bool
    incidents: str
    routes_id: int
    available_to_travel: bool
    delay: bool
```

### 6 Servicios

Son los que se encargan de realizar codigo mas complejo, como por ejemplo notificar si el vuelo esta activo o no.

Estan relacionados con los endpoints de la API, un endpoint puede hacer uso de uno o mas servicios.

Ejemplo de un servicio:

```python
sql_static_data = (
                select(
                    routes_model.Routes.id.label("route_id"),
                    routes_model.Routes.name.label("route_name"),
                    operators_model.Operators.name.label("operator_name"),
                    destinations_model.Destinations.name.label("destination_name"),
                    destinations_model.Destinations.time.label("destination_time"),
                    destinations_model.Destinations.tolerance.label("destination_tolerance"),
                    departures_model.Departures.name.label("departure_name"),
                    departures_model.Departures.time.label("departure_time"),
                    departures_model.Departures.tolerance.label("departure_tolerance"),
                    geozone_op.id.label("operator_geozone_id"),
                    geozone_op.name.label("operator_geozone_name"),
                    geozone_op.latitude.label("operator_latitude"),
                    geozone_op.longitude.label("operator_longitude"),
                    geozone_op.radius.label("operator_radius"),
                    geozone_des.id.label("destination_geozone_id"),
                    geozone_des.name.label("destination_geozone_name"),
                    geozone_des.latitude.label("destination_latitude"),
                    geozone_des.longitude.label("destination_longitude"),
                    geozone_des.radius.label("destination_radius"),
                )
                .select_from(
                    join(
                        routes_model.Routes,
                        operators_model.Operators,
                        routes_model.Routes.operator_id == operators_model.Operators.id,
                    )
                    .outerjoin(
                        operators_departures_model.OperatorsDeparturesRelationship,
                        operators_model.Operators.id
                        == operators_departures_model.OperatorsDeparturesRelationship.operator_id,
                    )
                    .outerjoin(
                        departures_model.Departures,
                        operators_departures_model.OperatorsDeparturesRelationship.departure_id
                        == departures_model.Departures.id,
                    )
                    .outerjoin(
                        destinations_model.Destinations,
                        routes_model.Routes.destination_id == destinations_model.Destinations.id,
                    )
                    .outerjoin(
                        geozone_op,
                        departures_model.Departures.geozone_id == geozone_op.id,
                    )
                    .outerjoin(
                        geozone_des,
                        destinations_model.Destinations.geozone_id == geozone_des.id,
                    )
                )
                .where(operators_model.Operators.object_id == object.id)
            )

            # Se agrega el objeto a la lista de objetos
            objects_list.append(
                Object(
                    object_id=object.id,
                    name=object.name,
                    group_name=object.group_name,
                    imei=object.imei,
                    is_broken=object.is_broken,
                    active=(lambda x: True if x == "true" else False)(object.active),
                    is_moving=(lambda x: True if x > 0 else False)(int(event_dict["speed"])),
                    protocol=object.protocol,
                    is_online=is_online,
                    offline_time=time,
                    is_ign_on=ignition,
                    route_name=(lambda x: x[0][1] if x != [] else "Not asigned")(db_static_data),
                    operator_name=(lambda x: x[0][2] if x != [] else "Not asigned")(db_static_data),
                    departure_name=(
                        lambda x: list(map(lambda x: x[6], db_static_data))
                        if x != []
                        else ["Not asigned"]
                    )(db_static_data),
                    destination_name=(lambda x: x[0][3] if x != [] else "Not asigned")(
                        db_static_data
                    ),
                    status=status,
                    events=EventGps(**event_dict),
                )
            )

```
