# Auxiliar5-Forms

En esta clase crearemos formularios utilizando las clases y métodos pre-hechos que nos entrega Django. Esta forma de hacerlo es útil para crear formularios sencillos que estén directamente asociados con el modelo de datos. Cabe destacar que, como en casi todo lo que hemos visto, esta no es la única manera de hacer esto y es posible que para distintas situaciones sea igual o más adecuado usar los métodos manuales que usamos anteriormente.

Como siempre, clona este repositorio en tu computador y crea tu ambiente virtual con Django para empezar a trabajar. Si tienes dudas sobre esto consulta la [Auxiliar 1](https://github.com/Aux-Ing-1/Auxiliar1-GIT) o durante la clase.

## Django Forms
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
     return render(request, "todoapp/index.html", {"tareas": mis_tareas, "form_tarea":form_tarea})
```

Por otro lado, debemos re-definir lo que haremos si recibimos una request POST, es decir, cuando recibamos un formulario con data.
Reemplaza la respuesta al método POST por lo siguiente:
```python
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
    return redirect("/tarea") # recargar la página
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
