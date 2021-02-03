creando una app con rails
----------------------------------------------------------------
rails new myapp
rails new myapp -d postgresql
rails new myapp -d mysql

configurando la base de datos en postgresql
----------------------------------------------------------------
-Creamos el archivo lacal_env.yml, en la ruta config/ almacenara las credenciales para el manejo de la base de datos en ambiente de dasorrollo o ambiente local

-Creamos el archivo lacal_env.yml, en la ruta config/, adicionar las credenciales de acceso a la base de datos local

	BD_LOCAL_USER: xxxxx
	BD_LOCAL_PASSWORD: xxxx

-En el archivo aplication.rb, en la ruta config/, colocamos el siguiente codigo, que permitira acceder al archivo local_env

    config.before_configuration do
      env_file = File.join(Rails.root, 'config', 'local_env.yml')
      YAML.load(File.open(env_file)).each do |key, value|
        ENV[key.to_s] = value.to_s
      end if File.exists?(env_file)
    end

-El archivo Database.yml, que se encuentra en la ruta config/ debe quedar de la siguiente manera

	  default: &default
	  adapter: postgresql
	  encoding: unicode
	  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
	  username: <%= ENV['BD_LOCAL_USER'] %>
	  password: <%= ENV['BD_LOCAL_PASSWORD'] %>

	development:
	  <<: *default
	  database: seguimiento_development
	  port: 5432
	  host: localhost

	test:
	  <<: *default
	  database: seguimiento_test
	  port: 5432
	  host: localhost

	production:

	  url: <%= ENV['DATABASE_URL'] %>

-Agregar el archivo lacal_env al git ignore, la razon, al ser un configuracion local solo le importa al desarrollador en su pc, no al resto del equipo, por lo
 tanto este archivo no debe ser inculido en el contro de versiones

-Para crear las bases de datos development y test ejecutar rake db:create

Ya tenemos nuestra nueva aplicacion creada.

covencion de nombre rails
----------------------------------------------------------------
controlador: plural y capitalzado(mayuscula la primera letra)
modelo: singular y no capitalizado(minusculas)

http://rubyni.com/2017/08/07/convenciones-de-nombres-de-rails/

Buscar ruta de un controlador en la consola
----------------------------------------------------------------
rails routes (muesta todas las rutas disponibles)
rails routes -c "nombredelcontrolado_sincomillas" (muestra todos los controladores asociados a la palabra colocada)

para sistemas basados en unix una alternativa de filtro seria:
rails routes | grep "aqui el nomnre del controlador" (usar las comillas mas el nombre del controlador que buscamos)

* script para creacion de una especialidad x consola
----------------------------------------------------------------
Especialidad.create(nombre:"SISTEMAS")

e=Especialidad.first{instancia la primera especialidad creada}

p=e.programas.create(nombre:"ADSI"){asigna el programa que se va a crear a la especialidad instanciada e instancia el programa}

p.fichas.create(numero:1693770,numero_aprendices:15,fecha_fin_at:"2020-10-20")

*asignar un user a la ficha con el formulario

f=Ficha.first

crear el user con el formulario de la aplicacion

u=User.first

u.add_role :admin

agregar una columna a una tabla en rails por consola
-------------------------------------------------------------------
rails g migration add_created_by_to_anotacion created_by:integer
rails g migration add_updated_by_to_anotacion created_by:integer

rails g migration add_created_by_to_comentario created_by:integer
rails g migration add_updated_by_to_comentario created_by:integer


* Script para obtener usuario y notificar nuevo comentario asociado a su anotacion
-------------------------------------------------------------------
Notificar al instructor de que un nuevo comentario ha sido asociado a una anotacion


* Script para obtener usuario y notificar nuevo comentario asociado a su anotacion
-------------------------------------------------------------------
Notificar al aprendiz de que un comentario ha sido generado sobre la nueva anotacion
c=Comentario.find(1)
userid=c.anotacion.anotable_id
user=User.find(userid)

Crear registro desde consola
-------------------------------------------------------------------
rails c
Obejto.create(campo1: "", campo2:
depende del tipo de formato del campo se necesitan las comiiillas para el string y sin comillas para campos numericos
Pet.create(name:"Muñeco", breed:"sinzhud")

Eliminar registro desde la cosola
-------------------------------------------------------------------
Entidad.find(1).destroy

Eliminar todos los registros de una tabla
-------------------------------------------------------------------
Post.delete_all

iniciar servicio de postgres nativo wsl2
-------------------------------------------------------------------
sudo service postgresql start

Restablecer ids de una tabla de la base de datos
-------------------------------------------------------------------
Curso.reset_pk_sequence
ActiveRecord::Base.connection.reset_pk_sequence!('table_name')

concetar con postgres
-------------------------------------------------------------------
sudo -u postgres psql

sino se conecta ejecutar
-------------------------------------------------------------------
su - postgres
psql

eliminar componente en rails
--------------------------------------------------------------------
rails d controller NombreControlador
rails d model 	   NombreModelo
rails d scaffold   NombreScaffold
rails d mailer     NombreMailer

Blanquear el schema tras eliminar todos los objetos del proyecto
--------------------------------------------------------------------

-Crear una migracion en blanco con nombre resetSchema
	rails g migrations resetSchema

-Ejecutar la migracion
	rais db:migration

esto crea un schema en blanco, a aprtir del cual se puede inicar a
trabajar nuevamente

Eliminar una base de datos en postgresql:
--------------------------------------------------------------------
DROP DATABASE myapp_development;

acceder a una credencial en ruby
--------------------------------------------------------------------
Rails.application.credentials.nombredelacredencial

editar registro de a una credencial en ruby
--------------------------------------------------------------------
EDITOR=NANO rails credentials:edit

Referencias con scaffolds:
--------------------------------------------------------------------
primero el modelo sin referencia
$ rails g scaffold Category name:string

segundo modelo que referencia
$ rails g scaffold Tarea description:text category:references

# adicionar una referencia con llave primaria personalizada
"add_foreign_key(from_table, to_table, **options)"
--------------------------------------------------------------------
add_foreign_key :articles, :users, column: :author_id, primary_key: "lng_id"

ref: https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_foreign_key

# establecer referencia con llave primaria customs
--------------------------------------------------------------------

add_foreign_key :group_fields, :poll_headers, column: :code, primary_key: 'code'poll_header_id
add_foreign_key :poll_bodies, :poll_headers, column: :code, primary_key: 'code'

la columna code debe encontrarse en los modelos que referencian a poll_headers
esta se agrega como unica y del mismo tipo que el que se encuentre en poll_headers

paso a paso:

Eliminar el id automatico generado por ruby:
------------------------------------------
Se genera una migracion en blanco y se agrega lo siguiente

    create_table :poll_headers, id: false do |t|
      t.string :code, null: false
      t.integer :age
      t.date :date
      t.references :gender, null: false, foreign_key: true
      t.references :institution, null: false, foreign_key: true

      t.timestamps
    end
    add_index :poll_headers, :code, unique: true

se crean los modelos sin estar relacionados
se agrega la columna en los modelos relacionados
se crea una nueva migracion en blanco, con rails g migration AddPollHeaderToGroupFields
se establece la llave foranea en los modelos creados adicionando el siguiente codigo
  def change
    add_foreign_key :group_fields, :poll_headers, column: :code, primary_key: 'code'
  end


sin alterar el nombre de la columna
--------------------------------------------------------------------

# Generacion de las relaciones entre los distintos modelos
--------------------------------------------------------------------

# Uno a muchos Tipodocumento user
$ rails g migration add_tipodocumento_to_users tipodocumento:references

En vscode.
class Tipodocumento < ApplicationRecord
	has_many :users
end

class User < ApplicationRecord
  belongs_to :tipodocumento
end

$ rails g migration add_tipodonovedad_to_novedades tiponovedad:references

#uno muchos Tipodonovedad novedades
class Tipodonovedad < ApplicationRecord
	has_many :novedades
end

class Novedades < ApplicationRecord
  belongs_to :Tiponovedad
end

# Muchos muchos grupo user

rails g migration create_join_table_grupos_books author:references book:references


class Grupo < ApplicationRecord
   has_and_belongs_to_many :users
end

class User < ApplicationRecord
   has_and_belongs_to_many :grupos
end

--------------------------------------------------------------------------------------------------------------------
BUSQUEDAS RAILS

Busqueda de user pertenecientes a una ficha con el user y rol definido en consola consola
--------------------------------------------------------------------------------------------------------------------
User.includes(:ficha).with_role(:aprendiz).where(fichas:{numero:1693770}).references(:ficha)

Busqueda de user pertenecientes a una ficha con el user y rol definido en consola consola
mas busqueda de user por parametros propios del user
---------------------------------------------------------------------------------------------------------------------
User.includes(:ficha).with_role(:aprendiz).where(fichas:{numero:''}).references(:ficha).or(User.includes(:ficha).
with_role(:aprendiz).where(users:{nombres:"Aprendiz"}).references(:ficha))

Rails
Listado de usuarios filtrados por, ficha, nombres, apellidos, ndocumento

Codigo controlador user, metodo index
@users = User.includes(:ficha).with_role(:aprendiz).where('cast(numero as text) ilike :q', q: "%#{params[:q]}%").references(:ficha).or(User.includes(
        :ficha).with_role(:aprendiz).where('ndocumento ilike :q or nombres ilike :q or apellidos ilike :q',
		q: "%#{params[:q]}%").references(:ficha)).page params[:page]

Vista index users caja de busuqeda
<form action="<%= users_path %>" method="GET" class="d-flex justify-content-between align-items-center">
	<div class="input-group">
		<input type="text" class="form-control" placeholder="Buscar.." aria-label="Recipient's username"
		aria-describedby="button-addon2" name="q" value="<%= @q %>">
		<div class="input-group-append">
			<button class="btn btn-primary" type="submit" id="button-addon2"><i
			class="fa fa-search"></i></button>
		</div>
	</div>
</form>

Filtros por un campo o multiples campos
---------------------------------------------------------------------------------------------------------------------
consola
Busca listado de anotacion por id o fecha de creacion.
User.find_by(ndocumento: 22699311).anotaciones.where('cast(id as text) ilike :q or cast(created_at as text) ilike :q' ,q: "%#{params[:q]}%")

Busca listado de anotacion por el nombre o apellido de quien se cree las creo.
User.find_by(ndocumento: 22699311).anotaciones.where(created_by:(User.where(nombres:"instructor2", apellidos:"instructor2")))

rails - controler - metodo index
Busqueda con parcial del nombre del creador
@anotaciones = @user.anotaciones.where('cast(created_by as text) ilike any (array[?])', User.where('nombres ilike :q or apellidos ilike :q', q:"%#{params[:q]}%").ids.map {|val| val.to_s}).order(id: :asc).order(id: :asc).page params[:page]

busqueda por id
@anotaciones = @user.anotaciones.where('cast(id as text) ilike :q or cast(created_at as text) ilike :q', q: "%#{params[:q]}%").order(id: :asc).page params[:page]

2 Busquedas para 4 campos
@anotaciones = @user.anotaciones.where('cast(id as text) ilike :q or cast(created_at as text) ilike :q', q: "%#{params[:q]}%").order(id: :asc).page params[:page].or(@user.anotaciones.where('cast(created_by as text) ilike any (array[?])', User.where('nombres ilike :q or apellidos ilike :q', q:"%#{params[:q]}%").ids.map {|val| val.to_s})).order(id: :asc).page params[:page]

busqueda con un caracter comodin en este caso :(adiciona un boton de informacion)
if params[:q].include? ':'
@anotaciones = @user.anotaciones.where('cast(id as text) ilike :q', q: "%#{params[:q].gsub(":","").to_i}%").order(id: :asc).page params[:page]
end

Buscar todos los usuarios con un rol determinado
@users = Role.find_by_name("%#{params[:q]}%").users

## Buscar por numero de dumento de manera unica
----------------------------------------------------------------------------------------------------------------------
@users = User.where('cast(ndocumento as text) ilike :q', q: "%#{params[:q].gsub(":","").to_i}%").order(id: :asc).page params[:page]

## Estilizar boton subir archivo simple_form

    <label class="btn btn-primary">
        Add a Avatar!
        <span style="display:none;">
            <%= f.file_field :avatar %>
        </span>
    </label>



