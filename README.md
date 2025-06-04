GitHub full site map ex


Great! Here’s a complete project structure and files for a working Django sitemap example using a blog-style app (blog) with a Post model.

⸻

🗂 Project Structure

myproject/
├── blog/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── sitemaps.py
│   ├── urls.py
│   ├── views.py
│   ├── templates/
│   │   └── blog/
│   │       └── post_detail.html
│   └── migrations/
│       └── __init__.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── db.sqlite3
├── manage.py


⸻

📄 Key Files

1. blog/models.py

from django.db import models
from django.urls import reverse

class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('post_detail', kwargs={'slug': self.slug})


⸻

2. blog/sitemaps.py

from django.contrib.sitemaps import Sitemap
from .models import Post

class PostSitemap(Sitemap):
    changefreq = 'weekly'
    priority = 0.9

    def items(self):
        return Post.objects.all()

    def lastmod(self, obj):
        return obj.updated_at


⸻

3. blog/views.py

from django.shortcuts import render, get_object_or_404
from .models import Post

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'blog/post_detail.html', {'post': post})


⸻

4. blog/urls.py

from django.urls import path
from .views import post_detail

urlpatterns = [
    path('<slug:slug>/', post_detail, name='post_detail'),
]


⸻

5. myproject/urls.py

from django.contrib import admin
from django.urls import path, include
from django.contrib.sitemaps.views import sitemap
from blog.sitemaps import PostSitemap

sitemaps = {
    'posts': PostSitemap,
}

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    path('sitemap.xml', sitemap, {'sitemaps': sitemaps}, name='sitemap'),
]


⸻

6. blog/templates/blog/post_detail.html

<!DOCTYPE html>
<html>
<head>
    <title>{{ post.title }}</title>
</head>
<body>
    <h1>{{ post.title }}</h1>
    <p>{{ post.content }}</p>
</body>
</html>


⸻

7. myproject/settings.py (Only key additions)

INSTALLED_APPS = [
    ...,
    'django.contrib.sitemaps',
    'blog',
]

# If using templates
TEMPLATES[0]['DIRS'] = [BASE_DIR / 'blog/templates']


⸻

✅ Final Steps to Run
	1.	Create the project:

django-admin startproject myproject
cd myproject
python manage.py startapp blog

	2.	Add files as above
	3.	Apply migrations and create superuser:

python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser

	4.	Run the server:

python manage.py runserver

	5.	Visit:

	•	http://localhost:8000/admin/ → add Posts
	•	http://localhost:8000/sitemap.xml → view sitemap

⸻

Would you like me to zip up this project structure or give a GitHub-style README as well?
