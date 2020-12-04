# Generacion de llave primaria aleatoria y unica

## Contexto

    Se requiere almacenar un registro en la base de datos, pero de
    que la llave primaria(PK), no sea autogenerada por rails, sino
    generada de manera aleatoria, cumpliendo con las cararcteristicas
    de las PK(primary key)
        unica.
        no nula.

## Ejecutando la creacion por consola

    Teniendo en cuenta que tenemos un modelo en la base de datos de nombre
    poll_header, que no tiene un id autgenerado, utilizaremos el metodo de
    ruby SecureRandom.hex, el cual nos garantiza un codigo unico el parametro
    que requiere es la cantidad de digitos a usar, usamos dos para un codigo
    de 4 digitos.

    ### prueba del metodo SecureRandom.hex

       - Ingresamos a la consola con el comando: rails c
       - Digitamos: SecureRandom.hex 2
       - susalida sera una expresion como la siguiente: "2b7c"

    ### Prueba para ingresar el dato en el campo id desde consola

    - Ingresamos a la consola con el comando: rails c

    - Digitamos: PollHeader.create(id: SecureRandom.hex(2), age: 28, gender_id: 2, institution_id: 1)

    - Al terminar la sentencia tenemos una salida como la siguiente:
        #<PollHeader id: "ef05", age: 28, date: nil, gender_id: 2, institution_id: 1,
        created_at: "2020-12-04 19:09:51", updated_at: "2020-12-04 19:09:51">

    - Podemos ver que en el campo id tenemo un valor aleatorio de 4 digitos.

## Ahora en rails.

    Para aplicar la solucion necesitamos hacer lo siguiente:

        - Recibir un request con los datos enviados desde el formulario.
        - Generar el id para el campo id antes de la creacion del registro
        - Insertar el dato en el request
        - Enviar los parametros

    Como establecimos en la lista anterior.

    - Recibir un request con los datos enviados desde el formulario.

        En la vista del elemento poll_header  tenemos un formulario new,
        el cual enviara el request al controlador.

    - Generar el id para el campo id antes de la creacion del registro e Insertar el
      dato en el request.

        params[:poll_header][:id] = SecureRandom.hex 2

      se introduce un dato en la llave id, la cual se encuentra al interior del request
      por lo tanto es llamar a los params y asignar valor a una llave de nombre poll_header
      en su llave id, por medio del generador SecureRandom

        # parametros obligatorios para el manejo en base de datos
        params.require(:poll_header).permit(:id, :age, :date, :gender_id, :institution_id)

    - Enviar los parametros

        el metodo create solicitara los datos del strong_params que hemos preparado