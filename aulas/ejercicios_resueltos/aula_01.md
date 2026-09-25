# Ejercicios de la sección 01

## Ejercicio 01

Agrega el `typos` para analizar gramática en inglés para ser ejecutado antes de la etapa de lint, transformándola en una secuencia que corra el `typos` y el `ruff check` en la tabla `[tool.poe.tasks]` de tu archivo `pyproject.toml`.

### Solución

El objetivo aquí es automatizar la verificación ortográfica para que corra siempre de forma automática antes del `ruff check`. Para hacer eso, vamos a transformar nuestro comando `lint` en una secuencia en Poe.

Abre tu archivo `pyproject.toml` y navega hasta la tabla `[tool.poe.tasks]`.

Como el `lint` antes era solo una string simple (`lint = { cmd = "ruff check" }`), necesitamos alterarlo para usar la propiedad `.sequence`, pasando una lista con el `typos` primero y el `ruff check` justo a continuación.

 Tu bloque de tareas debe quedar exactamente así:

```toml title="pyproject.toml" hl_lines="2-5"
[tool.poe.tasks]
lint.sequence = [
  { cmd = "typos" }, # (1)!
  { cmd = "ruff check" },
]
format.sequence = [
  { cmd = "ruff check --fix" },
  { cmd = "ruff format" },
]
serve = "fastapi dev fast_zero/app.py"
test.sequence = [
  "lint",
  { cmd = "pytest -s -x -vv $POE_EXTRA_ARGS" },
  { cmd = "coverage html --show-contexts" },
]
```

1. El comando `typos` fue agregado como el primer paso de la secuencia. Ahora, siempre que corras el lint, verificará la ortografía antes de pasar a las reglas del ruff.

Para garantizar que está funcionando, ejecuta el linter en tu terminal:

```shell title="$ Ejecución en la terminal!"
poetry lint
```