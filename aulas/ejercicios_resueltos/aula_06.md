# Ejercicios de la sección 06 
# **Autenticación y Autorización con JWT**

## Ejercicio 01

Haz un test para cubrir el escenario que lanza la exception `credentials_exception` en la autenticación en caso de que el `email` no sea enviado vía JWT. Al mirar la cobertura de `security.py` notarás que este contexto no está cubierto.

### Solución

Para ejecutar el bloque de código debes hacer una llamada a cualquier endpoint que dependa del token (currentUser) y enviar un token que no contenga una dirección de e-mail (sub):

```python title="tests/test_app.py"
def test_get_current_user_not_found__exercicio(client):
    data = {'no-email': 'test'}
    token = create_access_token(data)

    response = client.delete(
        '/users/1',
        headers={'Authorization': f'Bearer {token}'},
    )

    assert response.status_code == HTTPStatus.UNAUTHORIZED
    assert response.json() == {'detail': 'Could not validate credentials'}
```

## Ejercicio 02

Haz un test para cubrir el escenario que lanza la exception `credentials_exception` en la autenticación en caso de que el email sea enviado, pero no exista un `User` correspondiente registrado en la base de datos. Al mirar la cobertura de `security.py` notarás que este contexto no está cubierto.

### Solución

Para ejecutar el bloque de código debes hacer una llamada a cualquier endpoint que dependa del token (currentUser) y enviar un token que contenga una dirección de email (sub) que no esté registrado en la base de datos:

```python title="tests/test_app.py"
def test_get_current_user_does_not_exists__exercicio(client):
    data = {'sub': 'test@test'}
    token = create_access_token(data)

    response = client.delete(
        '/users/1',
        headers={'Authorization': f'Bearer {token}'},
    )

    assert response.status_code == HTTPStatus.UNAUTHORIZED
    assert response.json() == {'detail': 'Could not validate credentials'}
```

## Ejercicio 03

Revisa los tests creados hasta la clase 5 y ve si todavía tienen sentido (tests involucrando `#!python 409`)

### Solución

Los tests para los endpoints de PUT y DELETE, que verifican usuarios no existentes en la base de datos no tienen más sentido. Ya que para modificar o eliminar un user, tiene que ser validado por el token. Estos tests pueden ser eliminados.