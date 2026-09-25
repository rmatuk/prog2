# Ejercicios de la sección 05

## Ejercicio 01
Escribir un test para el endpoint de POST (create_user) que contemple el escenario donde el username ya fue registrado. Validando el error `#!python 409`.

### Solución

Para testear este escenario, necesitamos un username que ya esté registrado en la base de datos. Para eso, podemos usar la fixture de `user` que creamos. Ella es una garantía de que el valor ya está insertado en la base de datos:

```py title="/tests/test_app.py" hl_lines="1 5"
def test_create_user_should_return_409_username_exists__exercicio(client, user):
    response = client.post(
        '/users/',
        json={
            'username': user.username,
            'email': 'alice@example.com',
            'password': 'secret',
        },
    )
    assert response.status_code == HTTPStatus.CONFLICT
    assert response.json() == {'detail': 'Username already exists'}
```

## Ejercicio 02
Escribir un test para el endpoint de POST (create_user) que contemple el escenario donde el e-mail ya fue registrado. Validando el error `#!python 409`.

### Solución

Para testear este escenario, necesitamos un e-mail que ya esté registrado en la base de datos. Para eso, podemos usar la fixture de `user` que creamos. Ella es una garantía de que el valor ya está insertado en la base de datos:

```py title="/tests/test_app.py" hl_lines="1 5"
def test_create_user_should_return_409_email_exists__exercicio(client, user):
    response = client.post(
        '/users/',
        json={
            'username': 'alice',
            'email': user.email,
            'password': 'secret',
        },
    )
    assert response.status_code == HTTPStatus.CONFLICT
    assert response.json() == {'detail': 'Email already exists'}
```

## Ejercicio 03

Actualizar los tests creados en los ejercicios 1 y 2 de la [clase 03](../03.md/#ejercicios){:target="_blank"} para soportar la base de datos.

### Solución

El objetivo de este ejercicio no necesariamente es una actualización de los tests, sino el caso de una ejecución para validar si los tests, como fueron hechos, todavía funcionarían en esta nueva estructura.

Mis tests de la clase 03:

```python
def test_delete_user_should_return_not_found__exercicio(client):
    response = client.delete('/users/666')

    assert response.status_code == HTTPStatus.NOT_FOUND
    assert response.json() == {'detail': 'User not found'}


def test_update_user_should_return_not_found__exercicio(client):
    response = client.put(
        '/users/666',
        json={
            'username': 'bob',
            'email': 'bob@example.com',
            'password': 'mynewpassword',
        },
    )
    assert response.status_code == HTTPStatus.NOT_FOUND
    assert response.json() == {'detail': 'User not found'}
```

Al ejecutarlos continúan pasando.

## Ejercicio 04

Implementar la base de datos para el endpoint de listado por id, creado en el ejercicio 3 de la [clase 03](../03.md/#ejercicios){:target="_blank"}.


### Solución

Este ejercicio básicamente consiste en dos partes. La primera es alterar el endpoint para usar la base de datos. Esto puede ser hecho de manera simple inyectando la dependencia de la `session`:

```python
@app.get('/users/{user_id}', response_model=UserPublic)
def read_user__exercicio(
    user_id: int, session: Session = Depends(get_session)
):
    db_user = session.scalar(select(User).where(User.id == user_id))

    if not db_user:
        raise HTTPException(
            status_code=HTTPStatus.NOT_FOUND, detail='User not found'
        )

    return db_user
```

La segunda parte es entender qué necesita ser hecho en los tests para que puedan cubrir los dos casos previstos. El de éxito y el de falla.

El test de falla continúa pasando, pues de hecho no depende de ninguna interacción con la base de datos:
```python
def test_get_user_should_return_not_found__exercicio(client):
    response = client.get('/users/666')

    assert response.status_code == HTTPStatus.NOT_FOUND
    assert response.json() == {'detail': 'User not found'}
```

Ya el test de éxito, depende de que exista un usuario en la base de datos. Con eso podemos usar la fixture de `user` tanto en la llamada, como en la validación de los datos:

```python
def test_get_user___exercicio(client, user):
    response = client.get(f'/users/{user.id}')

    assert response.status_code == HTTPStatus.OK
    assert response.json() == {
        'username': user.username,
        'email': user.email,
        'id': user.id,
    }
```