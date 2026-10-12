---
marp: true
---

# Configurando la base de datos y gestionando migraciones con Alembic

> https://fastapidozero.dunossauro.com/4.0/04/

---

En las fuentes provistas, los **conceptos fundamentales** sobre bases de datos y ORMs se presentan como la base estructural para construir aplicaciones mantenibles, seguras y escalables con **SQLAlchemy** y **Alembic**.

---

### 1. Mapeo Objeto-Relacional (ORM) y sus Beneficios
Un **ORM (Object-Relational Mapping)** es una técnica de programación que vincula u objetos de un programa Python con registros de una base de datos relacional. Permite manipular la base de datos mediante métodos y atributos nativos sin necesidad de escribir consultas SQL crudas. Sus ventajas principales incluyen:
* **Abstracción de la base de datos:** Facilita el cambio de motor relacional (como MySQL, PostgreSQL o SQLite) con cambios mínimos en el código.
* **Seguridad:** Maneja de forma implícita los escapes de consultas, previniendo vulnerabilidades de inyección SQL.
* **Eficiencia en el desarrollo:** Automatiza la generación de esquemas y las migraciones de datos.

---

### 2. Componentes Clave de SQLAlchemy
SQLAlchemy se divide estructuralmente en dos niveles:
* **Core:** Proporciona una interfaz abstracta de bajo nivel para construir y ejecutar instrucciones SQL de manera segura a través de múltiples motores.
* **ORM:** Proporciona la interfaz de alto nivel orientada a objetos.

Dentro de esta arquitectura, destacan dos componentes esenciales:
* **Engine:** Es el punto de contacto central con la base de datos. Se instancia mediante `create_engine()`, gestiona las credenciales, la URI de conexión y administra el pool de conexiones.
* **Session:** Es la interfaz de persistencia e interacción del ORM. Actúa como intermediario gestionando las transacciones y aplicando patrones como la *Unidad de Trabajo* (*Unit of Work*).

---

### 3. Definición de Modelos y Configuración de Columnas
Los modelos definen la estructura de las tablas mediante clases Python:
* **Registro de tablas (`registry`):** Permite transformar las clases en *dataclasses* (usando el decorador `@mapped_as_dataclass`) y vincular los metadatos globales (`table_registry.metadata`).
* **Nombre de tabla:** Se especifica explícitamente mediante la propiedad `__tablename__`.
* **Anotaciones de tipo `Mapped[...]`:** Mapean atributos Python a tipos de columnas en la base de datos (por ejemplo, `Mapped[str]` o `Mapped[int]`).
* **Función `mapped_column()`:** Permite añadir restricciones y comportamientos específicos a las columnas:
  * `primary_key=True`: Define la clave primaria de la tabla.
  * `unique=True`: Garantiza que el valor no se repita en la tabla.
  * `server_default`: Asigna valores por defecto generados en el servidor SQL (como `server_default=func.now()`).
  * `init=False`: Indica que el campo no debe pasarse como argumento en el constructor al instanciar el objeto en Python, dejando que la base de datos o un evento lo rellene automáticamente.

---

### 4. Configuración del Entorno y la Metodología de los 12 Factores
Siguiendo el **tercer factor (*Config*)** de la metodología de los 12 factores, se recomienda desacoplar completamente la configuración del código fuente:
* **Pydantic Settings:** Se utiliza `pydantic-settings` (mediante clases que heredan de `BaseSettings` y `SettingsConfigDict`) para cargar de forma estructurada variables de entorno como `DATABASE_URL` desde archivos `.env`.
* **Control de versiones:** El archivo físico de la base de datos local (por ejemplo, `database.db`) debe agregarse al archivo `.gitignore` para evitar publicar credenciales o datos binarios en el repositorio.

---

### 5. Gestión de Migraciones Evolutivas con Alembic
A medida que la aplicación evoluciona, **Alembic** actúa como un sistema de control de versiones (similar a Git) para el esquema de la base de datos:
* **Estructura y archivos principales:**
  * `alembic.ini`: Contiene las configuraciones globales de Alembic.
  * `migrations/env.py`: Archivo responsable de importar las configuraciones de la aplicación y asociar `target_metadata = table_registry.metadata` para permitir la autogeneración de migraciones.
* **Proceso de migración:**
  1. **Generación automática:** El comando `alembic revision --autogenerate` compara las clases del modelo con la base de datos existente y crea un archivo de revisión.
  2. **Funciones `upgrade()` y `downgrade()`:** La revisión contiene código del Core de SQLAlchemy con las instrucciones necesarias para aplicar los cambios (`upgrade`) o revertirlos (`downgrade`).
  3. **Aplicación:** El comando `alembic upgrade head` ejecuta las migraciones pendientes hasta la versión más reciente (`head`).
  4. **Tabla `alembic_version`:** Alembic crea y mantiene esta tabla especial en la base de datos para almacenar el hash identificador (`version_num`) de la migración actual.

---

### 6. Pruebas Unitarias y Sistema de Eventos
* **Pruebas en memoria:** Para testear la creación y manipulación de tablas sin persistir datos reales, se emplea SQLite en memoria (`sqlite:///:memory:`) dentro de *fixtures* de Pytest, garantizando tests rápidos e independientes.
* **Eventos del ORM:** SQLAlchemy posee un sistema de eventos (`event.listen`) que permite ejecutar funciones *hook* en momentos específicos del ciclo de vida del objeto (por ejemplo, `before_insert`). Esto resulta útil para manipular campos como fechas de creación en entornos de prueba utilizando gestores de contexto deterministas (`@contextmanager`).

💡 ¿Te gustaría explorar algún aspecto en detalle, como la implementación práctica del sistema de eventos en tests o los comandos CLI de Alembic?

---

# Objetivos de esta clase:

-  Introducción a SQLAlchemy y Alembic
-  Instalando SQLAlchemy y Alembic
-  Configurando y creando la base de datos
-  Creando y localizando tablas utilizando SQLAlchemy
-  Testeando la creación de tablas
-  Eventos de SQLAlchemy
-  Gestionando migraciones de la base de datos con Alembic

---

# Una introducción a SQLAlchemy

---

## SQLAlchemy

SQLAlchemy es un ORM. Permite que trabajes con bases de datos SQL de manera más natural para los programadores Python. En vez de escribir consultas SQL crudas, puedes usar métodos y atributos Python para manipular tus registros de base de datos.

ORM significa Mapeo Objeto-Relacional. Es una técnica de programación que vincula (o mapea) objetos a registros de base de datos. En otras palabras, un ORM permite que interactúes con tu base de datos, como si estuvieras trabajando con objetos Python.


---

## ¿Pero por qué usaríamos un ORM?

- Abstracción de base de datos: Los ORMs permiten que cambies de un tipo de base de datos a otro con pocas alteraciones en el código.

- Seguridad: Los ORMs generalmente lidian con el escape de consultas y previenen inyecciones SQL, un tipo común de vulnerabilidad de seguridad.

- Eficiencia en el desarrollo: Los ORMs pueden generar automáticamente esquemas, realizar migraciones y otras tareas que serían demoradas de hacer manualmente.

---

## Instalación de SQLAlchemy

```bash
poetry add sqlalchemy
```

---

### Definiendo nuestro modelo de "user" con SQLAlchemy

en el archivo `fast_zero/models.py` vamos a crear

```python
from datetime import datetime
from sqlalchemy.orm import Mapped, mapped_as_dataclass, registry

table_registry = registry()


@mapped_as_dataclass(table_registry)
class User:
    __tablename__ = 'users'

    id: Mapped[int]
    username: Mapped[str]
    password: Mapped[str]
    email: Mapped[str]
    created_at: Mapped[datetime]
```

---

# Restricciones en columnas

```python
@mapped_as_dataclass(table_registry)
class User:
    __tablename__ = 'users'

    id: Mapped[int] = mapped_column(init=False, primary_key=True)
    username: Mapped[str] = mapped_column(unique=True)
    password: Mapped[str]
    email: Mapped[str] = mapped_column(unique=True)
    created_at: Mapped[datetime] = mapped_column(
        init=False, server_default=func.now()
    )
```

---

### Creando un test para este modelo

Vamos a crear un archivo nuevo para tests de base de datos: `tests/test_db.py`

```python
from fast_zero.models import User

def test_create_user():
    user = User(username='test', email='test@test.com', password='secret')

    assert user.password == 'secret'
```

> ¡Aquí tenemos una bomba!

---

### ¿Qué testea este test?

Aparentemente testea si una clase puede ser instanciada **o sea, NADA**.

Necesitamos garantizar algunas cosas:

1. Si es posible crear esta tabla
    - `Metadata`!
2. Si es posible buscar un `User` usando ella como base
    - `Session`!

Solo que para eso necesitamos conocer algunos otros componentes importantes.

---

## Otros componentes importantes

### Engine

La 'Engine' de SQLAlchemy es el punto de contacto con la base de datos, estableciendo y gestionando las conexiones. Se instancia a través de la función `create_engine()`, que recibe las credenciales de la base de datos, la dirección de conexión (URI) y configura el pool de conexiones.

### Session

En cuanto a la persistencia de datos y consultas a la base de datos utilizando el ORM, la Session es la principal interfaz. Actúa como un intermediario entre la aplicación Python y la base de datos, mediada por la Engine. La Session se encarga de todas las transacciones, proporcionando una API para conducirlas.

---

<div class="mermaid" style="text-align: center;">
graph
  B[SQLAlchemy ORM] -- proporciona --> D[Session]
  D -- eventos --> D
  D -- interactúa con --> C[Modelos]
  C -- eventos --> C
  C -- mapeados para --> G[Tablas en la Base de Datos]
  D -- depende de --> E[Engine]
  E -- se conecta con --> F[Base de Datos]
  C -- se asocia a --> H[Metadata]
  H -- mantiene informaciones de --> G[Tablas en la Base de Datos]
</div>

---

## Escribiendo tests para este modelo


---

La primera cosa que tenemos que montar es una fixture de la sesión de la base en `tests/conftest.py`

```python
import pytest
from sqlalchemy import create_engine, select
from sqlalchemy.orm import sessionmaker

from fast_zero.models import table_registry


@pytest.fixture
def session():
    engine = create_engine('sqlite:///:memory:')
    table_registry.metadata.create_all(engine)

    with Session(engine) as session:
        yield session

    table_registry.metadata.drop_all(engine)
```
---

### Lo sé, este código es un poco complejo de más [0]

1. `create_engine('sqlite:///:memory:')`: crea un mecanismo de base de datos SQLite en memoria usando SQLAlchemy. Este mecanismo será usado para crear una sesión de base de datos para nuestros tests.

2. `table_registry.metadata.create_all(engine)`: crea todas las tablas en la base de datos de test antes de cada test que usa la fixture `session`.

3. `with Session(engine) as session`: crea una sesión `Session` para que los tests puedan comunicarse con la base de datos vía `engine`.

---

### Lo sé, este código es un poco complejo de más [1]

4. `yield session`: proporciona una instancia de Session que será inyectada en cada test que solicita la fixture `session`. Esta sesión será usada para interactuar con la base de datos de test.

5. `table_registry.metadata.drop_all(engine)`: después de cada test que usa la fixture `session`, todas las tablas de la base de datos de test son eliminadas, garantizando que cada test sea ejecutado contra una base de datos limpia.

---

Ahora nuestro test

```python
from sqlalchemy import select
from fast_zero.models import User


def test_create_user(session):
    new_user = User(username='alice', password='secret', email='teste@test')
    session.add(new_user)
    session.commit()

    user = session.scalar(select(User).where(User.username == 'alice'))

    assert user.username == 'alice'
```

> task test

---

## Un éxito, pero no tanto

Tenemos un problema en este test que puede tornar complicado de validar. Validamos solamente el nombre `alice`, pero no validamos el objeto completo. Esto es un tanto complicado. Pues para validar el objeto entero, necesitamos saber a qué hora fue creado, por cuenta del campo `init=False`.

Inviabiliza el envío de un dato determinístico al objeto.

---

## Eventos del ORM

Para actuar en escenarios así, podemos "hacer trampa en los tests" usando eventos de SQLAlchemy.

<div class="columns">
<div class="mermaid" style="text-align: center;">
flowchart TD
   subgraph Operación
     direction LR
     A[Hook] --> B[Operación]
     B --> C[Hook]
   end
</div>

<div class="mermaid" style="text-align: center;">
flowchart TD
   commit --> Z["Insertar registro en la base (operación)"]
   subgraph Z["Insertar registro en la base (operación)"]
     direction LR
     A[Hook - before_insert] --> B[insert]
   end
</div>
</div>

La idea detrás de los eventos es hacer alguna operación antes o después de alguna operación.

---

### Ejemplo de evento

Llamamos al objeto `event` de SQLAlchemy para "escuchar" una operación:

```python
from sqlalchemy import event


def hook(mapper, connection, target):
   ...


event.listen(User, 'before_insert', hook)
```

En este caso, estamos escuchando `before_insert`. Lo que significa que ejecutará la función `hook` antes de insertar en la base de hecho.

---

### Nuestro evento para manipular el campo de fecha

```python
from contextlib import contextmanager
from datetime import datetime
from sqlalchemy import create_engine, event

# ...

@contextmanager
def _mock_db_time(*, model, time=datetime(2024, 1, 1)):

    def fake_time_hook(mapper, connection, target):
        if hasattr(target, 'created_at'):
            target.created_at = time

    event.listen(model, 'before_insert', fake_time_hook)

    yield time

    event.remove(model, 'before_insert', fake_time_hook)
```

---

### ¿Qué de hecho va a hacer este evento?

```python
@contextmanager
def _mock_db_time(*, model, time=datetime(2024, 1, 1)):

    def fake_time_hook(mapper, connection, target):
        if hasattr(target, 'created_at'):
            target.created_at = time

    event.listen(model, 'before_insert', fake_time_hook)

    yield time

    event.remove(model, 'before_insert', fake_time_hook)
```

Antes de ejecutar el insert la función `fake_time_hook` va a alterar el `created_at` al valor default del parámetro `time`. Haciendo que no use el valor por defecto del datetime de la db.

El `contextmanager` hace que la función pueda ser usada con el bloque `with`.

---

### Transformando en una fixture

Ahora que tenemos la función gestora de contexto, para evitar el sistema de importación durante los tests, podemos crear una fixture para él.

De forma bien simple, solamente retornando la función `_mock_db_time`:

```python title="tests/conftest.py"
@pytest.fixture
def mock_db_time():
    return _mock_db_time
```

De esta forma podemos hacer la llamada directa en el test.

---

### Haciendo el test del objeto completo

```python
from dataclasses import asdict
from sqlalchemy import select
from fast_zero.models import User

def test_create_user(session, mock_db_time):
    with mock_db_time(model=User) as time:
        new_user = User(
            username='alice', password='secret', email='teste@test'
        )
        session.add(new_user)
        session.commit()

	user = session.scalar(select(User).where(User.username == 'alice'))

    assert asdict(user) == {
        'id': 1,
        'username': 'alice',
        'password': 'secret',
        'email': 'teste@test',
        'created_at': time,
    }
```

---

### La validación completa

De esta forma todos los campos, hasta los que son manipulados directamente por el ORM pueden ser testeados.

```python
    assert asdict(user) ==
        'id': 1,
        'username': 'alice',
        'password': 'secret',
        'email': 'teste@test',
        'created_at': time,
    }
```

---

## Configuraciones de entorno y los 12 factores

Una buena práctica en el desarrollo de aplicaciones es separar las configuraciones del código.

Configuraciones, como credenciales de base de datos, son propensas a cambios entre entornos diferentes (como desarrollo, test y producción). 

Mezclarlas con el código puede tornar el proceso de cambio entre estos entornos complicado y propenso a errores.

```bash
poetry add pydantic-settings
```

---

## Configuración del entorno de la base de datos


```python
#fast_zero/settings.py
from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file='.env', env_file_encoding='utf-8'
    )

    DATABASE_URL: str = Field(init=False)
```

---

# `.env`

Esta configuración permite que usemos archivos `.env` para no insertar datos de la base en el código fuente

```env
DATABASE_URL="sqlite:///database.db"
```

No podemos olvidar de agregar esta base de datos al `.gitignore`

```
echo 'database.db' >> .gitignore
```

---

# Migraciones

Antes de avanzar, es importante entender qué son las migraciones de base de datos y por qué son útiles.

- Base de datos evolutiva
- La base acompaña las alteraciones del código
- Revertir alteraciones en el schema de la base

---

## Instalación y configuración de Alembic

```bash
poetry add alembic

alembic init migrations
```

---

## Esto creará una estructura de carpetas nueva

```
.
├── .env
├── alembic.ini     <-
├── fast_zero
│  ├── __init__.py
│  ├── app.py
│  ├── models.py
│  └── schemas.py
├── migrations      <-
│  ├── env.py
│  ├── README
│  ├── script.py.mako
│  └── versions
├── poetry.lock
├── pyproject.toml
├── README.md
└── tests
   ├── __init__.py
   ├── conftest.py
   ├── test_app.py
   └── test_db.py
```

---

## Configurando la migración automática

Vamos a hacer algunas alteraciones en el archivo `migrations/env.py` para que nuestras configuraciones de base de datos sean pasadas a Alembic:

1. Importar las `Settings` de nuestro archivo `settings.py` y el `table_registry` de nuestros modelos.
2. Configurar la URL de SQLAlchemy para ser la misma que definimos en `Settings`.
3. Verificar la existencia del archivo de configuración de Alembic y, si está presente, leerlo.
4. Definir los metadatos de destino como `table_registry.metadata`, que es lo que Alembic utilizará para generar automáticamente las migraciones.

---

```python
from alembic import context
from fast_zero.settings import Settings
from fast_zero.models import table_registry

config = context.config
config.set_main_option('sqlalchemy.url', Settings().DATABASE_URL)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = table_registry.metadata
```

---

# Generando la migración

```bash
alembic revision --autogenerate -m "create users table"
```
---

## Aplicando la migración

```bash
alembic upgrade head
```

---

## Dando una mirada a la base de datos

Para eso podríamos usar una herramienta gráfica o usando la CLI de sqlite:

```bash
python -m sqlite3 database.db

# Abrirá el shell de sqlite3
sqlite>
```

```sql
select * from alembic_version;
select * from users;
```