# Ejercicios de la sección 02

## Ejercicio 01

1. Crea un nuevo endpoint en `fast_zero/app.py` que retorne "hola mundo" usando HTML y escribe su test en `tests/test_app.py`.

> Consejo: para capturar la respuesta del HTML del cliente de tests, puedes usar `#!python response.text`


### Solución

Para la creación del endpoint retornando HTML debemos alterar la clase de respuesta por defecto de FastAPI a `HTMLResponse`:

```python title="Implementación del endpoint"
from fastapi.responses import HTMLResponse

# ...

@app.get('/exercicio-html', response_class=HTMLResponse)
def exercicio_aula_02():
    return """
    <html>
      <head>
        <title>¡Nuestro hola mundo!</title>
      </head>
      <body>
        <h1> Hola Mundo </h1>
      </body>
    </html>"""
```

El test que hace la validación del valor retornado por el endpoint no necesita ser muy robusto. La idea principal del ejercicio es solamente validar si estamos retornando el "Hola Mundo" en formato de HTML:

```python title="Implementación del test"
def test_exercicio_ola_mundo_em_html():
    client = TestClient(app)

    response = client.get('/exercicio-html')

    assert response.status_code == HTTPStatus.OK
    assert '<h1> Hola Mundo </h1>' in response.text
```

El `response.text` es un método del cliente de tests de FastAPI que convierte los bytes de respuesta en string.