# 🛡️ Armor VPS

**Blindaje de seguridad para VPS Ubuntu/Debian con un solo comando**

Automatiza la configuración de seguridad de tu servidor VPS en minutos: SSH endurecido, firewall UFW, y protección Fail2ban.

---

## ⚡ Instalación Rápida

Conéctate a tu VPS como **root** y ejecuta:

```bash
curl -fsSL https://raw.githubusercontent.com/TU_USUARIO/armor-vps/main/install.sh | bash
```

O si prefieres `wget`:

```bash
wget -qO- https://raw.githubusercontent.com/TU_USUARIO/armor-vps/main/install.sh | bash
```

**¡Eso es todo!** El script te pedirá:
1. Tu **llave pública SSH** (generada previamente en tu computadora)
2. El **puerto SSH** deseado (o presiona Enter para uno aleatorio)

---

## 📋 ¿Qué hace este script?

| Componente | Configuración |
|------------|---------------|
| **Usuario** | Se mantiene `root` como usuario principal |
| **Autenticación SSH** | Solo mediante llave pública (password desactivado) |
| **Puerto SSH** | Cambiado a un puerto alto (aleatorio o definido por ti) |
| **Firewall UFW** | Configuración mínima: puertos 80, 443 y el nuevo SSH |
| **Fail2ban** | Protección robusta con jails `sshd` + `recidive` |
| **Idempotencia** | Puedes ejecutarlo múltiples veces sin problemas |
| **Seguridad** | Rollback automático si algo falla |

---

## 🔐 Características de Seguridad

### SSH Endurecido
- ✅ Autenticación exclusiva por llave pública
- ✅ Puerto no estándar (dificulta escaneos automatizados)
- ✅ `PermitRootLogin prohibit-password` (root solo con llave)
- ✅ Desactivado: ChallengeResponse, X11Forwarding
- ✅ Máximo 3 intentos de autenticación

### Firewall UFW
- ✅ Política por defecto: denegar entrada, permitir salida
- ✅ Solo puertos esenciales: 80 (HTTP), 443 (HTTPS), SSH personalizado

### Fail2ban con Dos Jails
- ✅ **sshd**: 3 intentos fallidos = ban 24 horas
- ✅ **recidive**: reincidentes baneados 1 semana

---

## ⚠️ Requisitos Previos

### 1. Acceso root al servidor VPS
```bash
ssh root@TU_IP_DEL_SERVIDOR
```

### 2. Llave SSH pública generada

Si aún no tienes una llave SSH, créala en **tu computadora local**:

```bash
ssh-keygen -t ed25519 -C "tu_email@ejemplo.com"
```

Luego obtén tu llave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copia **todo** el contenido (comienza con `ssh-ed25519` o `ssh-rsa`).

### 3. Conexión activa

**¡IMPORTANTE!** No cierres tu sesión SSH actual hasta verificar que puedes conectarte con el nuevo puerto.

---

## 🚀 Instrucciones Detalladas

### Paso 1: Conecta a tu VPS como root

```bash
ssh root@TU_IP_DEL_SERVIDOR
```

### Paso 2: Ejecuta el comando de instalación

```bash
curl -fsSL https://raw.githubusercontent.com/TU_USUARIO/armor-vps/main/install.sh | bash
```

### Paso 3: Proporciona los datos solicitados

El script te pedirá:

1. **Tu llave pública SSH completa**
   Pégala tal como la copiaste (ejemplo: `ssh-ed25519 AAAAC3NzaC1... tu@email.com`)

2. **El puerto SSH deseado**
   - Ingresa un número entre 10000-65000
   - O presiona Enter para generar uno aleatorio

### Paso 4: Verifica la nueva conexión

**¡MUY IMPORTANTE!** Mantén abierta tu sesión actual y abre **una nueva terminal**.

En la nueva terminal, conéctate con el nuevo puerto:

```bash
ssh -p NUEVO_PUERTO root@TU_IP_DEL_SERVIDOR
```

Si la conexión funciona, ¡todo está listo! Puedes cerrar la sesión anterior.

---

## 🔧 Solución de Problemas

### No puedo conectarme después de ejecutar el script

Si aún tienes la sesión original abierta:

```bash
# Restaurar backup
cp /etc/ssh/sshd_config.backup.* /etc/ssh/sshd_config

# Reiniciar SSH
systemctl restart ssh
# o
systemctl restart sshd
```

### Ver el estado de los servicios

```bash
# Estado de SSH
systemctl status ssh

# Estado de UFW
ufw status verbose

# Estado de Fail2ban
fail2ban-client status
```

### Mi IP fue baneada por Fail2ban

Desde otra IP o consola de tu proveedor:

```bash
# Ver IPs baneadas
fail2ban-client status sshd

# Desbanear una IP específica
fail2ban-client set sshd unbanip TU_IP
```

### Necesito abrir puertos adicionales

```bash
# Ejemplo: abrir puerto 3000 para una aplicación
ufw allow 3000/tcp comment 'Mi App'

# Ver reglas actuales
ufw status numbered

# Eliminar una regla por número
ufw delete NUMERO
```

---

## 📚 Referencia Rápida

### Archivos de configuración importantes

| Archivo | Descripción |
|---------|-------------|
| `/etc/ssh/sshd_config` | Configuración principal de SSH |
| `/root/.ssh/authorized_keys` | Llaves públicas autorizadas |
| `/etc/fail2ban/jail.local` | Configuración de Fail2ban |
| `/var/log/auth.log` | Logs de autenticación |
| `/var/log/fail2ban.log` | Logs de Fail2ban |

### Comandos esenciales

```bash
# SSH
systemctl restart ssh           # Reiniciar SSH
sshd -t                         # Verificar sintaxis de config

# UFW
ufw status verbose              # Ver estado completo
ufw allow PUERTO/tcp            # Abrir puerto
ufw delete allow PUERTO/tcp     # Cerrar puerto
ufw reload                      # Recargar reglas

# Fail2ban
fail2ban-client status          # Ver jails activos
fail2ban-client status sshd     # Ver estado de jail sshd
fail2ban-client set sshd unbanip IP  # Desbanear IP
fail2ban-client reload          # Recargar configuración
```

---

## 🔌 Configuración de Cliente SSH (Opcional)

Para facilitar las conexiones futuras, configura tu archivo `~/.ssh/config` local:

```
Host mi-vps
    HostName TU_IP_DEL_SERVIDOR
    User root
    Port NUEVO_PUERTO
    IdentityFile ~/.ssh/id_ed25519
```

Y luego simplemente:

```bash
ssh mi-vps
```

---

## 📝 Compatibilidad

- ✅ Ubuntu 18.04+
- ✅ Ubuntu 20.04 LTS
- ✅ Ubuntu 22.04 LTS
- ✅ Ubuntu 24.04 LTS
- ✅ Debian 9+
- ✅ Debian 10 (Buster)
- ✅ Debian 11 (Bullseye)
- ✅ Debian 12 (Bookworm)

---

## ⚡ Características Técnicas

- **Idempotente**: Puedes ejecutarlo múltiples veces sin romper nada
- **Rollback automático**: Si algo falla, restaura la configuración original
- **Sin reinicio necesario**: El servidor no necesita reiniciarse
- **Backups automáticos**: Crea respaldo antes de cualquier cambio
- **Validaciones robustas**: Verifica cada paso antes de continuar

---

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add: nueva característica'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

---

## ⚠️ Descargo de Responsabilidad

Este script modifica configuraciones críticas de seguridad de tu servidor. Aunque incluye múltiples validaciones y rollback automático, **úsalo bajo tu propia responsabilidad**.

**Recomendaciones:**
- Prueba primero en un servidor de desarrollo
- Ten siempre una forma alternativa de acceder a tu servidor (consola del proveedor)
- Mantén la sesión SSH original abierta hasta verificar que todo funciona

---

## 💡 Tips de Seguridad Adicionales

Después de ejecutar Armor VPS, considera:

1. **Actualizar regularmente tu sistema:**
   ```bash
   apt update && apt upgrade -y
   ```

2. **Revisar logs periódicamente:**
   ```bash
   tail -f /var/log/auth.log
   tail -f /var/log/fail2ban.log
   ```

3. **Configurar alertas de Fail2ban** (opcional)

4. **Implementar 2FA** para SSH (avanzado)

5. **Usar fail2ban para otros servicios** (web, email, etc.)

---

## 📞 Soporte

Si tienes problemas o preguntas:

- 🐛 [Reportar un bug](https://github.com/TU_USUARIO/armor-vps/issues)
- 💬 [Iniciar una discusión](https://github.com/TU_USUARIO/armor-vps/discussions)
- ⭐ Si te fue útil, ¡deja una estrella!

---

<div align="center">

**Hecho con ❤️ para la comunidad**

[⬆ Volver arriba](#-armor-vps)

</div>
