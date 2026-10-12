	```docker title="Dockerfile"
	FROM python:{{short_version}}-slim
	WORKDIR app/
	COPY . .

	RUN pip install .

	EXPOSE 8000
	CMD uvicorn --host 0.0.0.0 fast_zero.app:app
	```

	Aquí está lo que hace cada línea:

	1. `FROM python:{{short_version}}-slim`: define la imagen base para nuestro contenedor. Estamos usando la versión slim de la imagen de Python {{short_version}}, que tiene todo lo que necesitamos para ejecutar nuestra aplicación.
	2. `WORKDIR app/`: define el directorio en el que ejecutaremos los comandos a continuación.
	3. `COPY . .`: copia todos los archivos del directorio actual al contenedor.
	4. `RUN pip install .`: instala las dependencias de nuestro proyecto. Solo las de `project.dependencies`, sin los grupos de opcionales.
	5. `EXPOSE 8000`: informa a Docker que el contenedor escuchará en el puerto 8000.
	6. `CMD uvicorn --host 0.0.0.0 fast_zero.app:app`: define el comando que será ejecutado cuando el contenedor sea iniciado.
