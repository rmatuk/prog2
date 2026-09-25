??? warning "sqlalchemy.exc.OperationalError: (sqlite3.OperationalError)"
	En caso de que recibas este error al aplicar la migración, esto es un problema causado por la forma en que Python se comunica con SQLite, haciendo que cada alteración en la base sea aplicada línea por línea.
	Para hacer todas las modificaciones de una vez, usamos las `operaciones en batch`.
	La idea es abrir una única conexión con la base de datos y ejecutar determinadas operaciones para todas las líneas antes de que la conexión sea cerrada.

	Para eso será preciso alterar el archivo de migraciones manualmente.
	El archivo debe parecerse a este:
	
    ```python title="/migrations/versions/bb77f9679811_exercicio_02_aula_04.py" linenums="20" hl_lines="3-4 15-16"
    # ...
    def upgrade():
        with op.batch_alter_table('users', schema=None) as batch_op:  #(1)!
            batch_op.add_column(   #(2)!
                sa.Column(
                    'updated_at',
                    sa.DateTime(),
                    server_default=sa.text('(CURRENT_TIMESTAMP)'),
                    nullable=False,
                )
            )
    
    
    def downgrade():
        with op.batch_alter_table('users', schema=None) as batch_op:  #(1)!
            batch_op.drop_column('updated_at')  #(3)!
    ```

    1. Entrando en el contexto de las operaciones en lote
    2. Agrega la columna `updated_at` en la tabla `users` con el `batch_op`
    3. Remueve la columna `updated_at` de la tabla `users` con el `batch_op`