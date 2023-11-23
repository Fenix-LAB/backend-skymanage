En esta carpeta se realiza la conexion a la base de datos, con el ORM SQLAlchemy, y se crea la sesion de la base de datos.

La conexion la implementaria de forma asincrona

Y el objetivo es poder crear una sesion de la base de datos, para poder realizar las consultas a la base de datos.

```python
POSTGRES_SQL_CONNECTION_ASYNC = (
    f"postgresql+asyncpg://{user}:{password}@{host}:{port}/{db_name}?ssl=disable"
)

@asynccontextmanager
async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        async with session.begin():
            try:
                yield session
            finally:
                await session.close()
```

Y una funcion para crear tablas:

```python
async def create_tables(engine):
    print("Creating tables...")
    async with engine.begin() as conn:
        print(f"conn: {conn}")
        # await conn.run_sync(Base.metadata.drop_all)
        await conn.run_sync(Base.metadata.create_all)
        print("Base.metadata:")
    await engine.dispose()
```