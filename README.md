# Auxiliar5-Forms

En esta clase crearemos formularios utilizando las clases y métodos pre-hechos que nos entrega Django. Esta forma de hacerlo es útil para crear formularios sencillos que estén directamente asociados con el modelo de datos. Cabe destacar que, como en casi todo lo que hemos visto, esta no es la única manera de hacer esto y es posible que para distintas situaciones sea igual o más adecuado usar los métodos manuales que usamos anteriormente.

Como siempre, clona este repositorio en tu computador y crea tu ambiente virtual con Django para empezar a trabajar. Si tienes dudas sobre esto consulta la [Auxiliar 1](https://github.com/Aux-Ing-1/Auxiliar1-GIT) o durante la clase.

## Django Form
Primero crearemos un formulario personalizado con los campos específicos que indiquemos nosotros/as mismos/as. Este formulario NO estará asociado directamente a nuestro modelo de datos.
### todoapp/forms.py
Debemos crear el archivo `forms.py`, en la misma carpeta que `todoapp/views.py`. El archivo debe contener lo siguiente:
```python
from django import forms
from categorias.models import Categoria

class NuevaTareaForm(forms.Form):
   titulo = forms.CharField(label="titulo de la tarea")
   contenido = forms.CharField(widget=forms.Textarea()) # <textarea> en vez de <input>
   categoria = forms.ModelChoiceField(queryset=Categoria.objects.all())
```
Nota que:
- Indicamos que los fields `titulo` y `contenido` son de tipo `CharField`, es decir texto.
- Cada field tiene asociado un tipo de widget, es decir el elemento HTML que se renderizará (`<input>`, `<select>`, `<textarea>`, etc). El tipo `CharField` por defecto tiene asociado `<input type='text'>`, pero para el field de `contenido` especificamos que queremos un `<textarea>`, que renderiza un cuadro para textos largos.
- Para `categoria` usamos un `ModelChoiceField` que trae todos los elementos `Categoria` de la base de datos.

### todoapp/views.py
Luego de crear el formulario, tenemos que usarlo y entregarlo al template. Como sabemos, esta lógica se controla en las vistas.
En el archivo `todoapp/views.py` debemos importar el formulario que creamos y renderizarlo cuando se haga una request GET a la URL '/tareas' (osea, cuando se solicita cargar la página). Reemplaza el contenido de la respuesta GET por el siguiente:
```python
from todoapp.forms import NuevaTareaForm
...
def tareas(request):
	...
  if request.method == "GET":
     form_tarea = NuevaTareaForm()
     return render(request, "todoapp/index.html", {"tareas": mis_tareas, "form_tarea": form_tarea})
```

Por otro lado, debemos re-definir lo que haremos si recibimos una request POST, es decir, cuando recibamos un formulario con data.
Reemplaza la respuesta al método POST por lo siguiente:
```python
from todoapp.forms import NuevaTareaForm
...
def tareas(request):
	...
  if request.method == "POST":
    form_tarea = NuevaTareaForm(request.POST)
    if form_tarea.is_valid():
       cleaned_data = form_tarea.cleaned_data
       if request.user.is_authenticated:
           Tarea.objects.create(**cleaned_data,owner=request.user)
       else:
           Tarea.objects.create(**cleaned_data)
    return render(request, "todoapp/index.html", {"tareas": mis_tareas, "form_tarea": form_tarea})
```
Notemos que esta vez al crear `form_tarea` le estamos entregando la request POST como parámetro. De esta forma creamos un `NuevaTareaForm` con la data que recibimos. Luego validamos la información (más adelante hablaremos de esto), y si es correcta, creamos una nueva Tarea con la data recibida, asociándola a un User si corresponde.

### todoapp/templates/todoapp/index.html
Finalmente, debemos actualizar nuestro template para que reciba y muestre el formulario enviado por la view. Para eso quitamos todo el formulario que construíamos a mano y lo reemplazamos por lo siguiente:
```html
<form action="" method="post">
    ...
        <div class="card-body">
            {{ form_tarea.as_p }}
            <button class="btn btn-primary" name="taskAdd" type="submit">Agregar tarea</button>
        </div>
    ...
</form>
```
El atributo `as_p` indica que cada field estará envuelto en un elemento `<p>`. También existe `as_ul` y `as_table`, o puede simplemente usarse `{{ form_tarea }}` para no usar ningún wrapper.

Si corremos nuestro servidor e ingresamos a `/tareas`, deberíamos ver algo así:
![image](https://user-images.githubusercontent.com/22943973/137008214-ab4a8d5e-1234-47bc-9815-178ba91d8bed.png)

Deberías poder agregar tareas utilizando esta interfaz. (Tendrás que agregar Categorias a mano desde el "/admin", como mencionamos al final de la [Auxiliar 2](https://github.com/Aux-Ing-1/Auxiliar2-Django#extra-acceder-al-admin-de-django)).

## Django ModelForm

Acabamos de crear un formulario con campos específicos indicados explícitamente. Ahora veremos cómo crear un formulario automáticamente utilizando los fields de un modelo ya definido en la base de datos.

Para esto usaremos el modelo `Tarea` que ya está creado en `models.py` y que se define así:
```python
class Tarea(models.Model):  # Todolist able name that inherits models.Model
    titulo = models.CharField(max_length=250)  # un varchar
    contenido = models.TextField(blank=True)  # un text
    fecha_creación = models.DateField(default=timezone.now().strftime("%Y-%m-%d"))  # un date
    categoria = models.ForeignKey(Categoria, default="general", on_delete=models.CASCADE)  # la llave foránea
    owner = models.ForeignKey(User, blank=True, null=True, on_delete=models.CASCADE)
```

### todoapp/forms.py
En la definición del formulario, esta vez usaremos `forms.ModelForm` en vez de `forms.Form`, de la siguiente manera:
```python
from django import forms
from .models import Tarea
...
class NuevaTareaModelForm(forms.ModelForm):
	titulo = forms.CharField(label="Título de la tarea")
	class Meta:
	       model = Tarea
	       fields = ['titulo', 'contenido', 'categoria']
```
Aquí en `class Meta` le estamos indicando cuál `model` queremos usar (`Tarea`) y cuáles campos queremos traer. En este caso solo queremos esos tres, pues los otros se crean automáticamente y no dependen de lo que ingrese el usuario.
Notemos que, a modo de ejemplo, estamos haciendo override del field `titulo`, para darle un label personalizado. Esto podría hacerse con cualquiera de los campos.

### todoapp/views.py
En la vista, para la request GET solo basta con crear el nuevo ModelForm que definimos y enviarlo al template.
```python
from todoapp.forms import NuevaTareaModelForm
...
def tareas(request):
	...
	if request.method == "GET":
	   form_tarea = NuevaTareaModelForm()
	   return render(request, "todoapp/index.html", {"tareas": mis_tareas, "form_tarea":form_tarea})
```

La respuesta para POST es un poco distinta. Reemplázala por lo siguiente:
```python
if request.method == "POST":
	form_tarea= NuevaTareaModelForm(request.POST)
	if form_tarea.is_valid():
		nueva_tarea = form_tarea.save() # save() de ModelForm
		if request.user.is_authenticated:
			nueva_tarea.owner = request.user
			nueva_tarea.save()  # save() de Model
	return render(request, "todoapp/index.html", {"tareas": mis_tareas, "form_tarea": form_tarea})
```
Notemos que, dado que `form_tarea` está directamente ligado a un model (en este caso, `Tarea`), podemos usar su método `.save()` directamente.

### todoapp/templates/todoapp/index.html
Para el template no es necesario hacer ninguna modificación sobre lo que hicimos en el Form normal.

## Validaciones
Al usar ModelForm, el template renderiza los campos con las limitaciones adecuadas para no permitir datos incorrectos (por ejemplo, si el model tenía `max_length=250`, el campo no permitirá más de 250 caracteres). Sin embargo, esta restricción estará solo del lado del cliente, y siempre es buena práctica hacer validaciones tanto en el cliente (frontend) como en el servidor (backend). Para validar campos desde el lado del servidor debemos modificar nuestra definición de los formularios. Para ejemplificar:
```python
class NuevaTareaModelForm(forms.ModelForm):
    class Meta:
        model = Tarea
        fields = ['titulo', 'contenido', 'categoria']

    def clean_titulo(self):	
        field = self.cleaned_data.get("titulo")
        if not "Tarea" in field:
            raise forms.ValidationError("Debe incluir el texto 'Tarea'")
        return field
```
Agregamos el método `clean_titulo`. Este método se correrá automáticamente al validar la data de field `titulo`, y en este ejemplo en particular, levantará un error de validación si el título no contiene el string "Tarea".

La forma genérica de validar sería agregar métodos a la clase del form, similares al ejemplo, de la siguiente forma:
```python
def clean_<nombre_del_field>(self):	
	field = self.cleaned_data.get("<nombre_del_field>")
	if <condicion_sobre_el_field>:
	    raise forms.ValidationError("<string de error>")
	elif <otra_condicion_sobre_el_field>:
	    raise forms.ValidationError("<otro_string de error>")
	return field
```
Siguiendo con el ejemplo, si intentamos agregar un valor incorrecto, se enviará la request POST, pero la respuesta será el mismo formulario con un error asociado.

![image](https://user-images.githubusercontent.com/22943973/137013211-5da6e051-c0e4-45b5-abc2-613e2dea2eae.png)

## Referencias y ejemplos
https://www.youtube.com/watch?v=wVnQkKf-gHo
https://docs.djangoproject.com/en/3.2/ref/forms/validation/

