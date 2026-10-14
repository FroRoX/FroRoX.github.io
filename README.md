# Blog de FroRoX

Blog personal de seguridad ofensiva / Red Team, hecho con [Jekyll](https://jekyllrb.com) y el tema [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy), en español.

## Antes de publicar: qué debes cambiar

Busca `TU_USUARIO` en el proyecto y reemplázalo por tu usuario de GitHub. Aparece en:

- `_config.yml` (`url`, `github.username`, `social.links`)
- `_tabs/about.md`
- `_data/contact.yml` (solo si activas LinkedIn)

También conviene completar:

- `_tabs/about.md`: tu presentación y tus certificaciones.
- `avatar:` en `_config.yml`: guarda tu imagen en `assets/img/` y apunta a ella (por ejemplo `/assets/img/avatar.png`).
- `social.email`, `twitter.username` y los contactos extra, si los quieres (ver `_data/contact.yml`).

## Probarlo en tu equipo

Requiere Ruby (3.x) y Bundler.

```bash
bundle install
bundle exec jekyll serve
```

Luego abre <http://127.0.0.1:4000>.

## Escribir un post

1. Copia `_posts/2026-10-10-plantilla-writeup.md`.
2. Renómbralo como `AAAA-MM-DD-titulo.md`.
3. Edita el encabezado (`title`, `date`, `categories`, `tags`) y escribe el contenido.

Las categorías y tags se generan solas en las pestañas **Categorías** y **Tags**.

## Publicar en GitHub Pages

1. Crea un repositorio llamado `TU_USUARIO.github.io` y sube este proyecto.
2. En el repositorio ve a **Settings → Pages** y en **Source** elige **GitHub Actions**.
3. El flujo incluido en `.github/workflows/` construye y publica el sitio en cada push a la rama principal.

## Estructura

| Ruta | Para qué sirve |
| --- | --- |
| `_config.yml` | Configuración general del sitio |
| `_posts/` | Tus publicaciones |
| `_tabs/` | Páginas del menú (About, Categorías, Tags, Archivo) |
| `_data/contact.yml` | Iconos de contacto en la barra lateral |
| `assets/img/` | Imágenes (avatar, capturas) |
