# Blog DevOps & Cloud

Blog personal sobre DevOps, Cloud Computing y tecnologías modernas, construido con [Hugo](https://gohugo.io/) y el tema [Book](https://github.com/alex-shpak/hugo-book).

## 🚀 Desarrollo local

```bash
# Clonar repositorio
git clone https://github.com/yosoyfunes/blog.yosoyfunes.com.git
cd blog.yosoyfunes.com

# Instalar tema
git submodule update --init --recursive

# Ejecutar servidor local
hugo server
```

Visita [http://localhost:1313](http://localhost:1313)

## 📝 Crear contenido

```bash
# Nuevo post
hugo new docs/categoria/nuevo-post.md

# Generar sitio
hugo
```

## 🌐 Despliegue

El sitio se publica automáticamente en GitHub Pages desde la carpeta `docs/`.

---

**Tecnologías:** Hugo, GitHub Pages, Markdown