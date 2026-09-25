    ```shell title="$ Ejecución en laterminal!"
    poetry env use {{short_version}}
    ```

	En conjunto con esta instrucción, debemos también especificar a Poetry que usaremos exactamente la versión `{{short_version}}` en nuestro proyecto. Para eso, alteramos el archivo de configuración pyproject.toml en la raíz del proyecto:

	```toml title="pyproject.toml" linenums="9"
	[project]
	# ...
	requires-python = ">={{short_version}},<4.0" # (1)!
	```

	1. La expresión `">={{short_version}},<4.0"` significa que cualquier versión mayor o igual a `{{short_version}}` y menor que 4.0 será válida para el proyecto.
