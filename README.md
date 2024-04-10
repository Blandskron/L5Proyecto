# L5Proyecto

# Entorno virtual
python -m venv venv
cd venv
Scripts\activate
cd ..

# Instalar Django
pip install django

# Proyecto Django
django-admin startproject users
cd users

# Correr migraciones
python manage.py makemigrations
python manage.py migrate

# Crear una app
python manage.py startapp perfiles

# Agregar la app al Settings.py
INSTALLED_APPS = [
    ...
    'perfiles',
]

# Crear el models.py
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Perfil(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.CharField(max_length=255, blank=True)
    web = models.URLField(blank=True)

    def __str__(self):
        return self.user.username

@receiver(post_save, sender=User)
def crear_o_actualizar_usuario_perfil(sender, instance, created, **kwargs):
    if created:
        Perfil.objects.create(user=instance)
    else:
        instance.perfil.save()

# Crear el forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class SignUpForm(UserCreationForm):
    first_name = forms.CharField(max_length=140, required=True)
    last_name = forms.CharField(max_length=140, required=False)
    email = forms.EmailField(max_length=254, help_text='Required. Inform a valid email address.', required=True)

    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name', 'password1', 'password2')

# Crear el urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('signup/', views.signup, name='signup'),
    path('accounts/profile/', views.index, name='profile'),
]

# Modificar el views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib.auth import logout
from .forms import SignUpForm


@login_required
def index(request):
    return render(request, 'accounts/index.html')

def logout_view(request):
    logout(request)
    return redirect('signup')


def signup(request):
    if request.method == 'POST':
        form = SignUpForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('login')
    else:
        form = SignUpForm()
    return render(request, 'registration/signup.html', {'form': form})

# Modificamos el urls.py de users
from django.contrib import admin
from django.urls import path, include
from perfiles.views import logout_view 

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('perfiles.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
    path('logout/', logout_view, name='logout'),
]

# Creamos la carpeta "templates" y dentro la carpeta "accounts" y "registration"
# En "accounts" index.html
# En "registration" login.html y signup.html
# Buscar estos archivos en el repositorio

# Daremos estilos a nuestros html para eso agregamos la ruta en setting.py
STATICFILES_DIRS = [
    BASE_DIR / "static",
]

# Correr migraciones
python manage.py makemigrations perfiles
python manage.py migrate
