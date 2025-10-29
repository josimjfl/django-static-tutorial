# django-static-tutorial
django static tutorial in production mode

**set up Django static files for production on Windows**, likely for **local deployment or desktop server use** (without Linux Nginx setup).

Here’s a **clean, working setup** specifically for **Windows production use** — for example, when using `runserver`, or packaging with **PyInstaller**, **NSSM**, or **XAMPP/Apache**.

---

## ⚙️ Step-by-Step Static File Setup (Windows)

### 🧩 1. Folder structure

Example project layout:

```
E:\Django\MyProject\
│
├── allfile\          # your development static files
│   ├── css\
│   ├── js\
│   └── images\
│
├── static\           # will be created automatically by collectstatic
│
├── media\            # for uploaded files
│
├── myapp\
├── myproject\
│   └── settings.py
└── manage.py
```

---

### ⚙️ 2. Configure `settings.py`

```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# STATIC FILES
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    BASE_DIR / "allfile",   # where you keep your dev static files
]
STATIC_ROOT = BASE_DIR / "static"

# MEDIA FILES (uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / "media"

# Production
DEBUG = False
ALLOWED_HOSTS = ['127.0.0.1', 'localhost']
```

---

### ⚙️ 3. Collect all static files

In the terminal (inside your project folder):

```bash
python manage.py collectstatic
```

✅ Django will copy everything from `allfile/` and apps’ `static/` folders into your `static/` directory.

You should see something like:

```
127 static files copied to 'E:\Django\MyProject\static'.
```

---

### ⚙️ 4. Serving Static Files in “Production” Mode on Windows

#### 🟢 Option A — Using `runserver` for internal deployment

Even though `runserver` isn’t meant for production, for internal/local apps you can still use it safely by manually serving static files:

In `urls.py` (usually your project-level one):

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # your urls
]

if settings.DEBUG:
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

👉 When `DEBUG = False`, `runserver` doesn’t serve static files.
But for **local Windows production**, you can just temporarily enable `DEBUG = True` if you’re not exposed to the internet.

---

#### 🟢 Option B — Using Whitenoise (best for Windows standalone use)

1. Install Whitenoise:

   ```bash
   pip install whitenoise
   ```

2. Add middleware in `settings.py`:

   ```python
   MIDDLEWARE = [
       'django.middleware.security.SecurityMiddleware',
       'whitenoise.middleware.WhiteNoiseMiddleware',
       ...
   ]
   ```

3. Set this for optimized serving:

   ```python
   STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
   ```

Now Whitenoise will automatically serve `/static/` and `/media/` when you run:

```bash
python manage.py runserver
```

Even with `DEBUG = False`, it will still work ✅

---

#### 🟢 Option C — Using Apache (via XAMPP / WAMP on Windows)

If you run Apache locally, add this inside your Apache config (e.g., `httpd-vhosts.conf`):

```apache
Alias /static/ "E:/Django/MyProject/static/"
<Directory "E:/Django/MyProject/static/">
    Require all granted
</Directory>

Alias /media/ "E:/Django/MyProject/media/"
<Directory "E:/Django/MyProject/media/">
    Require all granted
</Directory>
```

Then restart Apache from the XAMPP control panel.

---

### ⚙️ 5. Test Static and Media Access

Visit:

```
http://127.0.0.1:8000/static/css/yourfile.css
http://127.0.0.1:8000/media/uploads/test.png
```

If you see the file load — your setup is working perfectly ✅

---

### ⚙️ 6. Common Issues

| Problem                               | Solution                                                                   |
| ------------------------------------- | -------------------------------------------------------------------------- |
| Static not loading when `DEBUG=False` | Use Whitenoise or Apache alias                                             |
| `collectstatic` not copying           | Check `STATICFILES_DIRS` path correctness                                  |
| Browser cache not updating            | Clear browser cache or use Whitenoise compressed storage                   |
| Wrong slashes in paths                | Always use `BASE_DIR / "folder"` (Pathlib handles Windows paths correctly) |

---
