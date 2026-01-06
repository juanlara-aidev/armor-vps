# 🤝 Guía de Contribución

¡Gracias por tu interés en contribuir a Armor VPS! Este documento te guiará en el proceso.

## 🚀 Formas de Contribuir

- 🐛 Reportar bugs
- 💡 Sugerir nuevas características
- 📝 Mejorar la documentación
- 🔧 Enviar pull requests con correcciones o mejoras
- ⭐ Dar una estrella al proyecto

## 📋 Proceso de Contribución

### 1. Fork el Repositorio

Haz clic en el botón "Fork" en la parte superior derecha de la página del repositorio.

### 2. Clona tu Fork

```bash
git clone https://github.com/juanlara-aidev/armor-vps.git
cd armor-vps
```

### 3. Crea una Rama

```bash
git checkout -b feature/mi-nueva-caracteristica
```

Usa prefijos descriptivos:
- `feature/` - Para nuevas características
- `fix/` - Para correcciones de bugs
- `docs/` - Para cambios en documentación
- `refactor/` - Para refactorizaciones de código

### 4. Realiza tus Cambios

- Escribe código claro y comentado
- Sigue el estilo existente del proyecto
- Prueba tus cambios en un entorno de desarrollo

### 5. Commit tus Cambios

```bash
git add .
git commit -m "Add: descripción clara de tu cambio"
```

Usa prefijos en los commits:
- `Add:` - Para nuevas características
- `Fix:` - Para correcciones
- `Update:` - Para actualizaciones
- `Remove:` - Para eliminaciones
- `Docs:` - Para documentación

### 6. Push a tu Fork

```bash
git push origin feature/mi-nueva-caracteristica
```

### 7. Abre un Pull Request

Ve a tu fork en GitHub y haz clic en "Pull Request".

## 🧪 Pruebas

Antes de enviar un PR:

1. **Prueba el script** en un servidor VPS de desarrollo
2. **Verifica** que funcione en Ubuntu Y Debian
3. **Asegúrate** de que la idempotencia se mantenga (puede ejecutarse múltiples veces)
4. **Valida** que el rollback funcione correctamente en caso de error

## 📝 Estándares de Código

### Shell Script (Bash)

- Usa `#!/bin/bash` al inicio
- Comenta las secciones complejas
- Valida todas las entradas del usuario
- Usa variables descriptivas en español
- Maneja errores apropiadamente

### Documentación

- Usa **español** para toda la documentación
- Sé claro y conciso
- Incluye ejemplos cuando sea posible
- Usa emojis para mejorar la legibilidad

## 🐛 Reportar Bugs

Si encuentras un bug, [abre un issue](https://github.com/juanlara-aidev/armor-vps/issues/new) incluyendo:

- **Descripción clara** del problema
- **Pasos para reproducir** el bug
- **Comportamiento esperado** vs **comportamiento actual**
- **Sistema operativo** y versión (Ubuntu 22.04, Debian 11, etc.)
- **Logs relevantes** si están disponibles

## 💡 Sugerir Características

Para sugerir nuevas características:

1. Verifica que no exista una sugerencia similar
2. Abre un issue con el título "Feature: Tu sugerencia"
3. Describe el problema que resuelve
4. Propón una solución (opcional)

## 🔒 Consideraciones de Seguridad

Este proyecto trata con configuraciones de seguridad críticas:

- **Nunca** incluyas contraseñas o llaves privadas en el código
- **Valida** todas las entradas del usuario
- **Documenta** cualquier cambio relacionado con seguridad
- **Prueba** exhaustivamente antes de enviar PRs

## ✅ Checklist para PRs

Antes de enviar tu Pull Request, verifica:

- [ ] Mi código sigue el estilo del proyecto
- [ ] He probado mis cambios en Ubuntu y/o Debian
- [ ] He actualizado la documentación si es necesario
- [ ] Mi commit tiene un mensaje descriptivo
- [ ] He verificado que no rompo funcionalidad existente
- [ ] El script sigue siendo idempotente

## 📞 ¿Necesitas Ayuda?

Si tienes preguntas:

- Revisa la [documentación](README.md)
- Busca en [issues existentes](https://github.com/juanlara-aidev/armor-vps/issues)
- Abre un [nuevo issue](https://github.com/juanlara-aidev/armor-vps/issues/new)

---

**¡Gracias por contribuir a Armor VPS!** 🎉
