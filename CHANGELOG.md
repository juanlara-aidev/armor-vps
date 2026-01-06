# 📋 Changelog

Todos los cambios notables en este proyecto serán documentados en este archivo.

El formato está basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/),
y este proyecto adhiere a [Versionado Semántico](https://semver.org/lang/es/).

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

[1.0.0]: https://github.com/juanlara-aidev/armor-vps/releases/tag/v1.0.0
