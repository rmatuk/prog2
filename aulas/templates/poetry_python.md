	Para utilizar una versión específica de Python en nuestro ambiente, debemos solicitar a Poetry que instale esa versión:
	```bash title="$ Ejecución en terminal!"
	poetry python install {{short_version}}  #(1)!
	```

	1. Instala la última release da versión {{short_version}} de python
	
	Una respuesta similar a esta debe ser devuelta al ejecutar el comando:
	
	```bash title="Respuesta del comando `poetry python install`"
	Downloading and installing {{full_version}} (cpython) ... Done 
	Testing {{full_version}} (cpython) ... Done
	```
