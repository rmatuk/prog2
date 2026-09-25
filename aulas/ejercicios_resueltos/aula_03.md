# Ejercicios de la sección 03

## Ejercicio 01

Escribe un test para el error de `#!python 404` (NOT FOUND) para el endpoint de PUT.

### Solución

La idea de un test de `#!python 404` para el PUT es **intentar** hacer la alteración de un usuario que no existe en la base de datos.

```python title="Test de 404"
def test_update_user_should_return_not_found__exercicio(client):
    response = client.put(
        '/users/666', #(1)!
        json={
            'username': 'bob',
            'email': 'bob@example.com',
            'password': 'mynewpassword',
        },
    )
    assert response.status_code == HTTPStatus.NOT_FOUND #(2)!
    assert response.json() == {'detail': 'User not found'} #(3)!
```

1. El user con id `#!python 666` no existe en nuestro sistema.
2. Como el user no existe, el status code retornado por la función será `#!python 404`
3. Por entrar en el bloque de validación del `if` el `HTTPException` fue rellenado con `detail='User not found'`

## Ejercicio 02

Escribe un test para el error de `#!python 404` (NOT FOUND) para el endpoint de DELETE

### Solución

La idea de un test de 404 para el DELETE es **intentar** hacer la alteración de un usuario que no existe en la base de datos.

```python title="Test de 404"
def test_delete_user_should_return_not_found__exercicio(client):
    response = client.delete('/users/666') #(1)!

	assert response.status_code == HTTPStatus.NOT_FOUND #(2)!
    assert response.json() == {'detail': 'User not found'} #(3)!
```

1. El user con id `#!python 666` no existe en nuestro sistema.
2. Como el user no existe, el status code retornado por la función será `#!python 404`
3. Por entrar en el bloque de validación del `if` el `HTTPException` fue rellenado con `detail='User not found'`


## Ejercicio 03

Crea un endpoint de GET para obtener un único recurso como `users/{id}` y haz sus tests para `#!python 200` y `#!python 404`.

### Solución

La implementación del endpoint es bastante parecida a las que hicimos hasta ahora. Necesitamos validar si existe un `id` compatible en nuestra base de datos falsa, basándonos en la posición del elemento en la lista.

```python
@app.get('/users/{user_id}', response_model=UserPublic)
def read_user__exercicio(user_id: int):
    if user_id > len(database) or user_id < 1:
        raise HTTPException(
            status_code=HTTPStatus.NOT_FOUND, detail='User not found'
        )

    return database[user_id - 1]
```

Uno de los tests es sobre el retorno `#!python 404`, que se retorna un user que no existe en la base de datos y otro es el comportamiento por defecto para cuando el user es retornado con éxito:

```python
def test_get_user_should_return_not_found__exercicio(client):
    response = client.get('/users/666')

    assert response.status_code == HTTPStatus.NOT_FOUND
    assert response.json() == {'detail': 'User not found'}


def test_get_user___exercicio(client):
    response = client.get('/users/1')

    assert response.status_code == HTTPStatus.OK
    assert response.json() == {
        'username': 'bob',
        'email': 'bob@example.com',
        'id': 1,
    }
```