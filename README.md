<div align="center">
<img loading="lazy" style="width:700px" src="./docs/banner.jpg">
<h1 align="center">Django Multiple File Upload Template</h1>
<h3 align="center">Sample Project to show you how to implement multiple file upload with api and django form</h3>
</div>
<p align="center">
<a href="https://www.python.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a>
<a href="https://www.djangoproject.com/" target="_blank"> <img src="https://user-images.githubusercontent.com/29748439/177030588-a1916efd-384b-439a-9b30-24dd24dd48b6.png" alt="django" width="60" height="40"/> </a> 
<a href="https://www.docker.com/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg" alt="docker" width="40" height="40"/> </a>
<a href="https://www.postgresql.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/postgresql/postgresql-original-wordmark.svg" alt="postgresql" width="40" height="40"/> </a>
<a href="https://git-scm.com/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" alt="git" width="40" height="40"/> </a>
</p>

# Guideline
- [Guideline](#guideline)
- [Goal](#goal)
- [Development usage](#development-usage)
  - [Clone the repo](#clone-the-repo)
  - [Enviroment Varibales](#enviroment-varibales)
  - [Build everything](#build-everything)
  - [Note](#note)
  - [Check it out in a browser](#check-it-out-in-a-browser)
- [Model schema](#model-schema)
- [Form Base](#form-base)
- [API Base](#api-base)
- [Test Upload](#test-upload)
- [License](#license)
- [Bugs](#bugs)

# Goal
This project main goal is to provide a sample to show you how to implement multiple file uploading with django form and api.

# Development usage
You'll need to have [Docker installed](https://docs.docker.com/get-docker/).
It's available on Windows, macOS and most distros of Linux. 

If you're using Windows, it will be expected that you're following along inside
of [WSL or WSL
2](https://nickjanetakis.com/blog/a-linux-dev-environment-on-windows-with-wsl-2-docker-desktop-and-more).

That's because we're going to be running shell commands. You can always modify
these commands for PowerShell if you want.


## Clone the repo
Clone this repo anywhere you want and move into the directory:
```bash
git clone https://github.com/AliBigdeli/Django-Multiple-File-Upload-Sample.git
```

## Enviroment Varibales
enviroment varibales are included in docker-compose.yml file for debugging mode and you are free to change commands inside:

```yaml
services:
  backend:
  command: sh -c "python manage.py check_database && \ 
                      yes | python manage.py makemigrations  && \
                      yes | python manage.py migrate  && \                    
                      python manage.py runserver 0.0.0.0:8000"
    environment:      
      - DEBUG=True
```


## Build everything

The first time you run this it's going to take 5-10 minutes depending on your
internet connection speed and computer's hardware specs. That's because it's
going to download a few Docker images such as minio and build the Python + requirements dependencies. and dont forget to create a .env file inside dev folder for django and postgres with the samples.

```bash
docker compose up --build
```

Now that everything is built and running we can treat it like any other Django
app.

## Note

If you receive an error about a port being in use? Chances are it's because
something on your machine is already running on port 8000. then you have to change the docker-compose.yml file according to your needs.


## Check it out in a browser

Visit <http://localhost:8000> in your favorite browser.


# Model schema
a simple model for testing purposes
```python
from django.db import models


# Create your models here.

class Photo(models.Model):
    file = models.ImageField(upload_to="gallery/")
```



# Form Base
this is flow of form base uploading

```forms.py```
```python
from django import forms
from .models import Photo

class PhotoForm(forms.ModelForm):
    file = forms.FileField(widget=forms.ClearableFileInput(attrs={'multiple': True}))
    class Meta:
        model = Photo
        fields = ('file', )
```

```views.py```
```python
from django.http import HttpResponseRedirect
from django.urls import reverse_lazy
from django.views.generic import ListView,FormView,CreateView
from .forms import PhotoForm
from .models import Photo
# Create your views here.


class UploadView(CreateView):
    template_name = 'website/index.html'
    form_class = PhotoForm
    success_url = '/'


    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["gallery"] = Photo.objects.all()
        return context
    
    def post(self, request, *args, **kwargs):
        form_class = self.get_form_class()
        form = self.get_form(form_class)
        files = request.FILES.getlist('file')
        
        if 'file' not in request.FILES or not form.is_valid():
            return HttpResponseRedirect(reverse_lazy("website:index"))
        
        if form.is_valid():
            for file in files:
                Photo.objects.create(file=file)
            return HttpResponseRedirect(self.request.path_info)
        else:
            return self.form_invalid(form)
    
    
```

# API Base
this is flow of api base uploading

```serializers.py```
```python
from rest_framework import serializers
from rest_framework.exceptions import ValidationError
from ..models import Photo


class PhotoSerializer(serializers.Serializer):
    file = serializers.FileField(max_length=None, allow_empty_file=False)

```


```views.py```
```python
from rest_framework.response import Response
from rest_framework import status, viewsets
from rest_framework.parsers import MultiPartParser


from django.views.decorators.csrf import csrf_exempt

from .serializers import *
from ..models import *


class PhotoModelViewSet(viewsets.ModelViewSet):

    serializer_class = PhotoSerializer
    parser_classes = [MultiPartParser]

    @csrf_exempt
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
                
        if 'file' not in request.FILES or not serializer.is_valid():
            return Response({"details":"there is an issue with uploaded files"},status=status.HTTP_400_BAD_REQUEST)
        
        for file in request.FILES.getlist('file'):
            Photo.objects.create(file=file)
        
        return Response({"details":"uploaded successfully"}, status=status.HTTP_201_CREATED)
```
# Test Upload
for testing the upload mechanism you just need to hit on either form base or api base file selection and then choose one or more files which you want to upload. then hit ok and upload it. done!

<div align="center" ><img loading="lazy" style="width:700px" src="./docs/demo.gif"></div>


# License
MIT.


# Bugs
Feel free to let me know if something needs to be fixed. or even any features seems to be needed in this repo.
