# 📋 Changelog

Todos los cambios notables en este proyecto serán documentados en este archivo.

El formato está basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/),
y este proyecto adhiere a [Versionado Semántico](https://semver.org/lang/es/).

## [1.0.3] - 2026-01-07

### 🐛 Corregido

- **CRÍTICO**: Script ahora tolera errores de repositorios opcionales (backports en Debian 11)
  - El repositorio `bullseye-backports` retorna 404 Not Found en algunos servidores
  - Backports es OPCIONAL y no afecta la instalación de UFW/Fail2ban
  - Usa `--allow-releaseinfo-change` para ser más permisivo
  - Verifica disponibilidad de paquetes necesarios antes de fallar
  - Continúa la instalación si los paquetes principales están disponibles
  - Solo falla si realmente no se pueden instalar UFW o Fail2ban
  - Resuelve definitivamente el problema de instalación en Debian 11

## [1.0.2] - 2026-01-07

### 🐛 Corregido

- **CRÍTICO**: Corregido fallo en instalación de paquetes en Debian 11
  - El script fallaba silenciosamente en `apt-get update` sin mostrar el error
  - Ahora detecta y espera si hay procesos apt/dpkg bloqueados
  - Captura y muestra errores reales de apt en lugar de ocultarlos
  - Intenta limpiar caché de apt y reintentar automáticamente si falla
  - Muestra detalles útiles de errores al usuario para mejor diagnóstico
  - Mejora significativa en la robustez de instalación en Debian/Ubuntu

## [1.0.1] - 2026-01-06

### 🐛 Corregido

- **CRÍTICO**: Corregido problema donde el script se detenía al ejecutarse mediante `curl | bash`
  - Ahora `read` lee desde `/dev/tty` en lugar de `stdin`
  - Permite que el usuario pueda ingresar su llave SSH y puerto correctamente
  - El script ahora funciona perfectamente con `curl -fsSL URL | bash`

## [1.0.0] - 2026-01-06

### ✨ Añadido

- Script de instalación automatizado con un solo comando
- Configuración de SSH endurecido:
  - Autenticación exclusiva por llave pública
  - Puerto personalizado (aleatorio o definido por usuario)
  - Desactivación de autenticación por password
  - Configuraciones de seguridad robustas
- Firewall UFW con configuración mínima:
  - Puertos 80 (HTTP) y 443 (HTTPS) abiertos
  - Puerto SSH personalizado configurado
  - Política de deny por defecto para entrada
- Fail2ban con dos jails:
  - `sshd`: Protección contra ataques de fuerza bruta (3 intentos = ban 24h)
  - `recidive`: Penalización para reincidentes (3 bans = ban 1 semana)
- Sistema de rollback automático en caso de error
- Validaciones robustas de entrada de usuario
- Backups automáticos antes de aplicar cambios
- Idempotencia: puede ejecutarse múltiples veces sin problemas
- Compatibilidad con Ubuntu 18.04+ y Debian 9+
- README completo con instrucciones detalladas
- Licencia MIT
- Documentación de contribución (CONTRIBUTING.md)
- Sistema de colores en la salida del script para mejor UX

### 🔒 Seguridad

- Todas las operaciones críticas son verificadas antes de aplicarse
- Backup automático de configuración SSH
- Validación de sintaxis SSH antes de reiniciar el servicio
- Verificación de que el puerto SSH está escuchando después del reinicio
- Protección contra duplicación de reglas UFW
- Configuración de Fail2ban con filtros actualizados

---

## Tipos de cambios

- `✨ Añadido` - Para nuevas características
- `🔄 Cambiado` - Para cambios en funcionalidad existente
- `⚠️ Deprecado` - Para características que serán removidas en próximas versiones
- `🗑️ Removido` - Para características removidas
- `🐛 Corregido` - Para corrección de bugs
- `🔒 Seguridad` - Para cambios relacionados con vulnerabilidades

---

[1.0.3]: https://github.com/juanlara-aidev/armor-vps/compare/v1.0.2...v1.0.3
[1.0.2]: https://github.com/juanlara-aidev/armor-vps/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/juanlara-aidev/armor-vps/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/juanlara-aidev/armor-vps/releases/tag/v1.0.0
