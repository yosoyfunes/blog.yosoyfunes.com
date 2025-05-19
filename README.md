# blog.yosoyfunes.com

Este es el repositorio del blog personal de Matias, construido con [Hugo](https://gohugo.io/) y el tema [PaperMod](https://adityatelange.github.io/hugo-PaperMod/).

## Requisitos

- [Hugo](https://gohugo.io/getting-started/installing/) (versión extendida recomendada)
- Git

## Instalación

1. Clona el repositorio:
   ```sh
   git clone https://github.com/yosoyfunes/blog.yosoyfunes.com.git
   cd blog.yosoyfunes.com
   ```

2. Instala el tema PaperMod como submódulo:
   ```sh
   git submodule update --init --recursive
   ```

## Uso local

Para correr el sitio en modo desarrollo:

```sh
hugo server
```

Abre tu navegador en [http://localhost:1313](http://localhost:1313).

## Publicación en GitHub Pages

El sitio se publica en la carpeta `docs/` para ser compatible con GitHub Pages.

1. Genera el sitio:
   ```sh
   hugo
   ```
   Esto actualizará el contenido en `docs/`.

2. Haz commit y push de los cambios:
   ```sh
   git add .
   git commit -m "Actualizar sitio"
   git push origin main
   ```

3. En la configuración del repositorio en GitHub, ve a **Settings > Pages** y selecciona la rama `main` y la carpeta `/docs` como fuente.

## Crear un nuevo post

```sh
hugo new posts/nombre-del-post.md
```
Edita el archivo generado en `content/posts/` y cambia `draft: true` a `draft: false` para publicarlo.

## Configuración de redes sociales

Las redes sociales se configuran en el archivo `hugo.toml` bajo la sección `[params.social]`.

## Recursos

- [Documentación de Hugo](https://gohugo.io/documentation/)
- [PaperMod Theme](https://adityatelange.github.io/hugo-PaperMod/)

---