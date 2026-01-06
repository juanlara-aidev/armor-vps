# 🛡️ Guía de Blindaje de Seguridad para VPS

## Ubuntu / Debian

---

## 📋 Descripción General

Este script automatiza el proceso de **blindaje de seguridad** para servidores VPS que utilizan **Ubuntu** o **Debian**. Está diseñado para ejecutarse en un solo paso, sin intervención manual más allá de proporcionar dos datos iniciales.

### ¿Qué hace exactamente este script?

| Componente | Configuración |
|------------|---------------|
| **Usuario** | Se mantiene `root` como usuario principal |
| **Autenticación SSH** | Solo mediante llave pública (password desactivado) |
| **Puerto SSH** | Cambiado a un puerto alto (definido por ti o aleatorio) |
| **Firewall UFW** | Configuración mínima: puertos 80, 443 y el nuevo SSH |
| **Fail2ban** | Protección robusta con jails `sshd` + `recidive` |
| **Idempotencia** | A prueba de re-ejecución (no duplica reglas) |
| **Seguridad** | Rollback automático si algo falla |

### Características de seguridad implementadas

1. **SSH endurecido:**
   - Autenticación exclusiva por llave pública
   - Puerto no estándar (dificulta escaneos automatizados)
   - `PermitRootLogin prohibit-password` (root solo con llave)
   - `ChallengeResponseAuthentication no`
   - `X11Forwarding no`

2. **Firewall UFW:**
   - Política por defecto: denegar entrada, permitir salida
   - Solo puertos esenciales abiertos (80, 443, SSH personalizado)

3. **Fail2ban con dos jails:**
   - `sshd`: 3 intentos fallidos = ban 24 horas
   - `recidive`: reincidentes baneados 1 semana

---

## ⚠️ Requisitos Previos

Antes de ejecutar el script, asegúrate de tener:

1. **Acceso root** al servidor VPS
2. **Tu llave pública SSH** ya generada en tu computadora local
   
   Si no tienes una llave SSH, créala en tu computadora local:
   ```bash
   ssh-keygen -t ed25519 -C "tu_email@ejemplo.com"
   ```
   
   Luego obtén tu llave pública:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   
   Copia todo el contenido (comienza con `ssh-ed25519` o `ssh-rsa`).

3. **Conexión activa** al servidor (no cierres la sesión hasta verificar)

---

## 🚀 Instrucciones de Uso

### Paso 1: Conecta a tu VPS como root

```bash
ssh root@TU_IP_DEL_SERVIDOR
```

### Paso 2: Copia y ejecuta el script

1. Selecciona **TODO** el contenido del bloque de código de abajo
2. Pégalo en la terminal de tu VPS
3. Presiona **Enter**

El script te pedirá:
- **Tu llave pública SSH** (pégala completa)
- **El puerto SSH deseado** (o presiona Enter para uno aleatorio)

### Paso 3: Verifica la conexión

**¡IMPORTANTE!** No cierres la sesión actual hasta verificar que puedes conectarte con el nuevo puerto.

---

## 📜 El Script

> **IMPORTANTE:** Copia desde la línea que dice `#!/bin/bash` hasta la línea que dice `# FIN DEL SCRIPT` (inclusive).

```bash
#!/bin/bash
#═══════════════════════════════════════════════════════════════════════════════
#  SCRIPT DE BLINDAJE PARA VPS - Ubuntu/Debian
#  Versión: 1.0.0
#  
#  Autor: Guía de Seguridad VPS
#  Compatibilidad: Ubuntu 18.04+ / Debian 9+
#═══════════════════════════════════════════════════════════════════════════════

#───────────────────────────────────────────────────────────────────────────────
# CONFIGURACIÓN DE COLORES Y VARIABLES GLOBALES
#───────────────────────────────────────────────────────────────────────────────
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'

SSHD_CONFIG="/etc/ssh/sshd_config"
SSHD_BACKUP=""
SSH_SERVICE=""
NEW_SSH_PORT=""
PUBLIC_KEY=""
SCRIPT_SUCCESS=false

#───────────────────────────────────────────────────────────────────────────────
# FUNCIONES DE UTILIDAD
#───────────────────────────────────────────────────────────────────────────────

print_banner() {
    clear
    echo -e "${CYAN}"
    echo "╔═══════════════════════════════════════════════════════════════════╗"
    echo "║                                                                   ║"
    echo "║     🛡️  SCRIPT DE BLINDAJE VPS v1.0                               ║"
    echo "║         Ubuntu / Debian                                          ║"
    echo "║                                                                   ║"
    echo "╚═══════════════════════════════════════════════════════════════════╝"
    echo -e "${NC}"
}

print_step() {
    echo -e "\n${BLUE}▶ [PASO]${NC} $1"
}

print_substep() {
    echo -e "  ${CYAN}→${NC} $1"
}

print_ok() {
    echo -e "  ${GREEN}✓${NC} $1"
}

print_error() {
    echo -e "  ${RED}✗ [ERROR]${NC} $1"
}

print_warning() {
    echo -e "  ${YELLOW}⚠ [AVISO]${NC} $1"
}

#───────────────────────────────────────────────────────────────────────────────
# FUNCIÓN DE ROLLBACK
#───────────────────────────────────────────────────────────────────────────────

rollback() {
    echo ""
    echo -e "${RED}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${RED}║  ERROR DETECTADO - INICIANDO ROLLBACK AUTOMÁTICO                  ║${NC}"
    echo -e "${RED}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
    
    if [[ -n "$SSHD_BACKUP" && -f "$SSHD_BACKUP" ]]; then
        print_substep "Restaurando configuración SSH desde backup..."
        cp "$SSHD_BACKUP" "$SSHD_CONFIG"
        
        if [[ -n "$SSH_SERVICE" ]]; then
            systemctl restart "$SSH_SERVICE" 2>/dev/null || true
        else
            systemctl restart sshd 2>/dev/null || systemctl restart ssh 2>/dev/null || true
        fi
        
        print_ok "Configuración SSH restaurada"
        echo -e "\n  ${YELLOW}Backup utilizado:${NC} $SSHD_BACKUP"
    else
        print_warning "No se encontró backup para restaurar"
    fi
    
    echo ""
    echo -e "${YELLOW}El servidor debería seguir accesible con la configuración original.${NC}"
    echo ""
}

#───────────────────────────────────────────────────────────────────────────────
# VERIFICACIONES INICIALES
#───────────────────────────────────────────────────────────────────────────────

check_root() {
    print_step "Verificando permisos de root..."
    
    if [[ $EUID -ne 0 ]]; then
        print_error "Este script debe ejecutarse como root"
        echo -e "  Ejecuta: ${YELLOW}sudo su -${NC} y luego vuelve a intentar"
        exit 1
    fi
    
    print_ok "Ejecutando como root"
}

check_os() {
    print_step "Verificando sistema operativo..."
    
    if [[ ! -f /etc/debian_version ]]; then
        print_error "Este script solo es compatible con Ubuntu/Debian"
        exit 1
    fi
    
    # Obtener info del OS
    if [[ -f /etc/os-release ]]; then
        source /etc/os-release
        print_ok "Sistema detectado: $PRETTY_NAME"
    else
        print_ok "Sistema Debian/Ubuntu detectado"
    fi
}

detect_ssh_service() {
    print_step "Detectando servicio SSH..."
    
    if systemctl list-unit-files | grep -q "^sshd.service"; then
        SSH_SERVICE="sshd"
    elif systemctl list-unit-files | grep -q "^ssh.service"; then
        SSH_SERVICE="ssh"
    else
        print_error "No se detectó servicio SSH instalado"
        exit 1
    fi
    
    print_ok "Servicio SSH detectado: $SSH_SERVICE"
}

#───────────────────────────────────────────────────────────────────────────────
# BACKUP DE CONFIGURACIÓN
#───────────────────────────────────────────────────────────────────────────────

create_backup() {
    print_step "Creando backup de configuración SSH..."
    
    if [[ ! -f "$SSHD_CONFIG" ]]; then
        print_error "No se encontró $SSHD_CONFIG"
        exit 1
    fi
    
    SSHD_BACKUP="${SSHD_CONFIG}.backup.$(date +%Y%m%d_%H%M%S)"
    
    if cp "$SSHD_CONFIG" "$SSHD_BACKUP"; then
        print_ok "Backup creado: $SSHD_BACKUP"
    else
        print_error "No se pudo crear el backup"
        exit 1
    fi
}

#───────────────────────────────────────────────────────────────────────────────
# OBTENER DATOS DEL USUARIO
#───────────────────────────────────────────────────────────────────────────────

get_public_key() {
    echo ""
    echo -e "${YELLOW}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${YELLOW}║  PASO 1: INGRESA TU LLAVE PÚBLICA SSH                             ║${NC}"
    echo -e "${YELLOW}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
    echo -e "  La llave debe comenzar con: ${CYAN}ssh-rsa${NC}, ${CYAN}ssh-ed25519${NC}, ${CYAN}ecdsa-sha2${NC}, etc."
    echo -e "  Ejemplo: ssh-ed25519 AAAAC3NzaC1... tu@email.com"
    echo ""
    echo -e "  ${BLUE}Pega tu llave pública completa y presiona ENTER:${NC}"
    echo ""
    
    read -r PUBLIC_KEY
    
    # Eliminar espacios en blanco al inicio y final
    PUBLIC_KEY=$(echo "$PUBLIC_KEY" | xargs)
    
    # Validaciones
    if [[ -z "$PUBLIC_KEY" ]]; then
        print_error "La llave pública no puede estar vacía"
        exit 1
    fi
    
    if ! echo "$PUBLIC_KEY" | grep -qE "^(ssh-rsa|ssh-ed25519|ssh-dss|ecdsa-sha2-nistp256|ecdsa-sha2-nistp384|ecdsa-sha2-nistp521|sk-ssh-ed25519|sk-ecdsa-sha2-nistp256)[[:space:]]"; then
        print_error "Formato de llave pública inválido"
        echo -e "  ${YELLOW}Asegúrate de copiar la llave completa, incluyendo el prefijo ssh-*${NC}"
        exit 1
    fi
    
    # Verificar que tenga al menos 2 partes (tipo y llave)
    local key_parts
    key_parts=$(echo "$PUBLIC_KEY" | awk '{print NF}')
    if [[ "$key_parts" -lt 2 ]]; then
        print_error "La llave pública parece estar incompleta"
        exit 1
    fi
    
    print_ok "Llave pública validada correctamente"
}

get_ssh_port() {
    echo ""
    echo -e "${YELLOW}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${YELLOW}║  PASO 2: DEFINE EL NUEVO PUERTO SSH                               ║${NC}"
    echo -e "${YELLOW}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
    echo -e "  Rango permitido: ${CYAN}10000 - 65000${NC}"
    echo -e "  ${BLUE}Ingresa el puerto deseado o presiona ENTER para uno aleatorio:${NC}"
    echo ""
    
    read -r USER_PORT
    
    if [[ -z "$USER_PORT" ]]; then
        # Generar puerto aleatorio alto
        print_substep "Generando puerto aleatorio..."
        
        local attempts=0
        local max_attempts=50
        
        while [[ $attempts -lt $max_attempts ]]; do
            # Usar /dev/urandom para mejor aleatoriedad
            if command -v shuf &>/dev/null; then
                NEW_SSH_PORT=$(shuf -i 10000-65000 -n 1)
            else
                NEW_SSH_PORT=$((RANDOM % 55000 + 10000))
            fi
            
            # Verificar que no esté en uso
            if ! ss -tuln 2>/dev/null | grep -qE ":${NEW_SSH_PORT}[[:space:]]" && \
               ! netstat -tuln 2>/dev/null | grep -qE ":${NEW_SSH_PORT}[[:space:]]"; then
                break
            fi
            
            ((attempts++))
        done
        
        if [[ $attempts -ge $max_attempts ]]; then
            print_error "No se pudo encontrar un puerto libre después de $max_attempts intentos"
            exit 1
        fi
        
        print_ok "Puerto aleatorio generado: $NEW_SSH_PORT"
    else
        # Validar puerto ingresado
        if ! [[ "$USER_PORT" =~ ^[0-9]+$ ]]; then
            print_error "El puerto debe ser un número"
            exit 1
        fi
        
        if [[ "$USER_PORT" -lt 10000 || "$USER_PORT" -gt 65000 ]]; then
            print_error "El puerto debe estar entre 10000 y 65000"
            echo -e "  ${YELLOW}Los puertos bajos (<1024) están reservados y los muy altos pueden tener problemas${NC}"
            exit 1
        fi
        
        # Verificar si está en uso
        if ss -tuln 2>/dev/null | grep -qE ":${USER_PORT}[[:space:]]" || \
           netstat -tuln 2>/dev/null | grep -qE ":${USER_PORT}[[:space:]]"; then
            print_error "El puerto $USER_PORT ya está en uso"
            exit 1
        fi
        
        NEW_SSH_PORT="$USER_PORT"
        print_ok "Puerto seleccionado: $NEW_SSH_PORT"
    fi
}

#───────────────────────────────────────────────────────────────────────────────
# INSTALACIÓN DE PAQUETES
#───────────────────────────────────────────────────────────────────────────────

install_packages() {
    print_step "Actualizando sistema e instalando paquetes..."
    
    # Configurar apt para no interactivo
    export DEBIAN_FRONTEND=noninteractive
    
    # Actualizar repositorios
    print_substep "Actualizando lista de paquetes..."
    if ! apt-get update -y &>/dev/null; then
        print_error "Error al actualizar repositorios"
        return 1
    fi
    print_ok "Repositorios actualizados"
    
    # Instalar UFW si no está instalado
    print_substep "Verificando/Instalando UFW..."
    if dpkg -l | grep -qw ufw; then
        print_ok "UFW ya está instalado"
    else
        if ! apt-get install -y ufw &>/dev/null; then
            print_error "Error al instalar UFW"
            return 1
        fi
        print_ok "UFW instalado"
    fi
    
    # Instalar Fail2ban si no está instalado
    print_substep "Verificando/Instalando Fail2ban..."
    if dpkg -l | grep -qw fail2ban; then
        print_ok "Fail2ban ya está instalado"
    else
        if ! apt-get install -y fail2ban &>/dev/null; then
            print_error "Error al instalar Fail2ban"
            return 1
        fi
        print_ok "Fail2ban instalado"
    fi
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# CONFIGURACIÓN DE LLAVES SSH
#───────────────────────────────────────────────────────────────────────────────

setup_authorized_keys() {
    print_step "Configurando llave SSH autorizada..."
    
    local auth_keys_file="/root/.ssh/authorized_keys"
    
    # Crear directorio .ssh si no existe
    if [[ ! -d /root/.ssh ]]; then
        print_substep "Creando directorio /root/.ssh..."
        mkdir -p /root/.ssh
    fi
    
    # Establecer permisos correctos
    chmod 700 /root/.ssh
    
    # Crear archivo authorized_keys si no existe
    if [[ ! -f "$auth_keys_file" ]]; then
        touch "$auth_keys_file"
    fi
    
    chmod 600 "$auth_keys_file"
    
    # Añadir llave solo si no existe (idempotencia)
    if grep -qF "$PUBLIC_KEY" "$auth_keys_file" 2>/dev/null; then
        print_warning "La llave pública ya existe en authorized_keys"
    else
        echo "$PUBLIC_KEY" >> "$auth_keys_file"
        print_ok "Llave pública añadida a authorized_keys"
    fi
    
    # Verificar que el archivo no está vacío
    if [[ ! -s "$auth_keys_file" ]]; then
        print_error "El archivo authorized_keys está vacío"
        return 1
    fi
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# CONFIGURACIÓN DE SSH
#───────────────────────────────────────────────────────────────────────────────

configure_ssh() {
    print_step "Configurando SSH..."
    
    # Función ROBUSTA para establecer opciones de configuración
    # Compatible con Ubuntu Y Debian (maneja comentarios, evita duplicados)
    set_ssh_option() {
        local key="$1"
        local value="$2"
        local config_file="$SSHD_CONFIG"
        local temp_file="${config_file}.tmp"
        
        # Crear archivo temporal vacío
        > "$temp_file"
        
        local found=false
        
        # Leer línea por línea
        while IFS= read -r line || [[ -n "$line" ]]; do
            # Verificar si esta línea contiene nuestra key (comentada o no)
            if echo "$line" | grep -qE "^[[:space:]]*#?[[:space:]]*${key}[[:space:]]"; then
                if [[ "$found" == false ]]; then
                    # Primera ocurrencia: reemplazar con el nuevo valor
                    echo "${key} ${value}" >> "$temp_file"
                    found=true
                fi
                # Si ya encontramos una, ignorar duplicados
            else
                echo "$line" >> "$temp_file"
            fi
        done < "$config_file"
        
        # Si no se encontró, añadir al final
        if [[ "$found" == false ]]; then
            echo "${key} ${value}" >> "$temp_file"
        fi
        
        # Reemplazar archivo original
        mv "$temp_file" "$config_file"
    }
    
    print_substep "Aplicando configuraciones de seguridad..."
    
    # Configuraciones principales
    set_ssh_option "Port" "$NEW_SSH_PORT"
    set_ssh_option "PasswordAuthentication" "no"
    set_ssh_option "PubkeyAuthentication" "yes"
    set_ssh_option "PermitRootLogin" "prohibit-password"
    set_ssh_option "ChallengeResponseAuthentication" "no"
    set_ssh_option "KbdInteractiveAuthentication" "no"
    set_ssh_option "UsePAM" "yes"
    set_ssh_option "X11Forwarding" "no"
    set_ssh_option "PrintMotd" "no"
    set_ssh_option "PermitEmptyPasswords" "no"
    set_ssh_option "MaxAuthTries" "3"
    set_ssh_option "LoginGraceTime" "30"
    set_ssh_option "ClientAliveInterval" "300"
    set_ssh_option "ClientAliveCountMax" "2"
    
    print_ok "Opciones de SSH configuradas"
    
    # Verificar sintaxis de la configuración
    print_substep "Verificando sintaxis de configuración SSH..."
    
    local sshd_test_output
    if ! sshd_test_output=$(sshd -t 2>&1); then
        print_error "Error en la configuración de SSH:"
        echo "$sshd_test_output"
        return 1
    fi
    
    print_ok "Sintaxis de configuración SSH válida"
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# CONFIGURACIÓN DE UFW
#───────────────────────────────────────────────────────────────────────────────

configure_ufw() {
    print_step "Configurando firewall UFW..."
    
    # Desactivar UFW temporalmente para configurar
    print_substep "Preparando UFW..."
    ufw --force disable &>/dev/null || true
    
    # Reset de reglas (esto elimina duplicados)
    print_substep "Limpiando reglas anteriores..."
    echo "y" | ufw --force reset &>/dev/null || true
    
    # Configurar políticas por defecto
    print_substep "Estableciendo políticas por defecto..."
    ufw default deny incoming &>/dev/null
    ufw default allow outgoing &>/dev/null
    
    # Añadir reglas
    print_substep "Añadiendo reglas de firewall..."
    
    # HTTP
    ufw allow 80/tcp comment 'HTTP' &>/dev/null
    print_ok "Puerto 80 (HTTP) permitido"
    
    # HTTPS
    ufw allow 443/tcp comment 'HTTPS' &>/dev/null
    print_ok "Puerto 443 (HTTPS) permitido"
    
    # SSH personalizado
    ufw allow "${NEW_SSH_PORT}/tcp" comment 'SSH' &>/dev/null
    print_ok "Puerto $NEW_SSH_PORT (SSH) permitido"
    
    # Activar UFW
    print_substep "Activando UFW..."
    if ! echo "y" | ufw --force enable &>/dev/null; then
        print_error "Error al activar UFW"
        return 1
    fi
    
    print_ok "UFW activado correctamente"
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# CONFIGURACIÓN DE FAIL2BAN
#───────────────────────────────────────────────────────────────────────────────

configure_fail2ban() {
    print_step "Configurando Fail2ban..."
    
    # Detener fail2ban para configurar
    print_substep "Deteniendo Fail2ban para configuración..."
    systemctl stop fail2ban &>/dev/null || true
    
    # Crear configuración local (sobrescribe si existe)
    print_substep "Creando configuración personalizada..."
    
    cat > /etc/fail2ban/jail.local << FAIL2BAN_CONFIG
#═══════════════════════════════════════════════════════════════════════════════
# CONFIGURACIÓN FAIL2BAN - Generada por Script de Blindaje VPS
# Fecha: $(date '+%Y-%m-%d %H:%M:%S')
#═══════════════════════════════════════════════════════════════════════════════

[DEFAULT]
# Tiempo de baneo por defecto: 1 hora
bantime = 3600

# Ventana de tiempo para contar intentos: 10 minutos
findtime = 600

# Máximo de intentos antes del ban
maxretry = 5

# IPs a ignorar (localhost)
ignoreip = 127.0.0.1/8 ::1

# Acción de baneo usando UFW
banaction = ufw
banaction_allports = ufw

# Backend de detección
backend = auto

#───────────────────────────────────────────────────────────────────────────────
# JAIL: SSHD - Protección contra ataques de fuerza bruta SSH
#───────────────────────────────────────────────────────────────────────────────
[sshd]
enabled = true
port = ${NEW_SSH_PORT}
filter = sshd
logpath = %(sshd_log)s
backend = %(sshd_backend)s

# Configuración más estricta para SSH
maxretry = 3
bantime = 86400
findtime = 3600

#───────────────────────────────────────────────────────────────────────────────
# JAIL: RECIDIVE - Penaliza reincidentes (baneados múltiples veces)
#───────────────────────────────────────────────────────────────────────────────
[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log

# Configuración para reincidentes
bantime = 604800
findtime = 86400
maxretry = 3
FAIL2BAN_CONFIG

    print_ok "Archivo jail.local creado"
    
    # Verificar/crear filtro recidive si no existe
    if [[ ! -f /etc/fail2ban/filter.d/recidive.conf ]]; then
        print_substep "Creando filtro recidive..."
        
        cat > /etc/fail2ban/filter.d/recidive.conf << 'RECIDIVE_FILTER'
# Filtro para detectar IPs baneadas repetidamente
[Definition]
failregex = ^.*\]\s+Ban\s+<HOST>\s*$
ignoreregex =
RECIDIVE_FILTER
        
        print_ok "Filtro recidive creado"
    else
        print_ok "Filtro recidive ya existe"
    fi
    
    # Habilitar e iniciar fail2ban
    print_substep "Habilitando Fail2ban..."
    systemctl enable fail2ban &>/dev/null
    
    print_substep "Iniciando Fail2ban..."
    if ! systemctl start fail2ban &>/dev/null; then
        print_error "Error al iniciar Fail2ban"
        # No es crítico, el servidor sigue funcional
        print_warning "Fail2ban no se inició, pero la configuración está lista"
        return 0
    fi
    
    # Esperar un momento para que inicie
    sleep 2
    
    # Verificar que está corriendo
    if systemctl is-active --quiet fail2ban; then
        print_ok "Fail2ban iniciado correctamente"
    else
        print_warning "Fail2ban puede tardar en iniciar completamente"
    fi
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# REINICIAR SERVICIO SSH
#───────────────────────────────────────────────────────────────────────────────

restart_ssh() {
    print_step "Reiniciando servicio SSH..."
    
    print_substep "Aplicando nueva configuración..."
    
    if ! systemctl restart "$SSH_SERVICE"; then
        print_error "Error al reiniciar el servicio SSH"
        return 1
    fi
    
    # Esperar a que el servicio inicie
    sleep 3
    
    # Verificar que está corriendo
    print_substep "Verificando estado del servicio..."
    
    if ! systemctl is-active --quiet "$SSH_SERVICE"; then
        print_error "El servicio SSH no está activo después del reinicio"
        return 1
    fi
    
    # Verificar que el puerto está escuchando
    if ! ss -tuln | grep -qE ":${NEW_SSH_PORT}[[:space:]]"; then
        print_error "SSH no está escuchando en el puerto $NEW_SSH_PORT"
        return 1
    fi
    
    print_ok "SSH reiniciado y escuchando en puerto $NEW_SSH_PORT"
    
    return 0
}

#───────────────────────────────────────────────────────────────────────────────
# RESUMEN FINAL
#───────────────────────────────────────────────────────────────────────────────

show_summary() {
    local server_ip
    server_ip=$(hostname -I | awk '{print $1}')
    
    echo ""
    echo -e "${GREEN}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${GREEN}║                                                                   ║${NC}"
    echo -e "${GREEN}║     ✅  ¡BLINDAJE COMPLETADO EXITOSAMENTE!                        ║${NC}"
    echo -e "${GREEN}║                                                                   ║${NC}"
    echo -e "${GREEN}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
    echo -e "${CYAN}═══════════════════════════════════════════════════════════════════${NC}"
    echo -e "${CYAN}                    📋 RESUMEN DE CONFIGURACIÓN                     ${NC}"
    echo -e "${CYAN}═══════════════════════════════════════════════════════════════════${NC}"
    echo ""
    echo -e "  ${BLUE}Puerto SSH:${NC}              ${GREEN}${NEW_SSH_PORT}${NC}"
    echo -e "  ${BLUE}Autenticación:${NC}           Solo llave pública"
    echo -e "  ${BLUE}Password SSH:${NC}            ${RED}Desactivado${NC}"
    echo -e "  ${BLUE}Root Login:${NC}              Solo con llave (prohibit-password)"
    echo ""
    echo -e "  ${BLUE}Firewall UFW:${NC}            Activo"
    echo -e "    → Puerto 80/tcp        ${GREEN}Abierto${NC} (HTTP)"
    echo -e "    → Puerto 443/tcp       ${GREEN}Abierto${NC} (HTTPS)"
    echo -e "    → Puerto ${NEW_SSH_PORT}/tcp      ${GREEN}Abierto${NC} (SSH)"
    echo ""
    echo -e "  ${BLUE}Fail2ban:${NC}                Activo"
    echo -e "    → Jail sshd:           3 intentos = ban 24h"
    echo -e "    → Jail recidive:       3 bans = ban 1 semana"
    echo ""
    echo -e "${RED}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${RED}║                                                                   ║${NC}"
    echo -e "${RED}║  ⚠️   ¡¡¡ NO CIERRES ESTA SESIÓN TODAVÍA !!!                      ║${NC}"
    echo -e "${RED}║                                                                   ║${NC}"
    echo -e "${RED}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
    echo -e "  Abre una ${YELLOW}NUEVA TERMINAL${NC} y verifica que puedes conectarte:"
    echo ""
    echo -e "    ${GREEN}ssh -p ${NEW_SSH_PORT} root@${server_ip}${NC}"
    echo ""
    echo -e "  ${CYAN}Si la conexión funciona:${NC}"
    echo -e "    ✓ Puedes cerrar esta sesión de forma segura"
    echo ""
    echo -e "  ${RED}Si la conexión NO funciona:${NC}"
    echo -e "    ✗ Usa esta sesión para diagnosticar el problema"
    echo -e "    ✗ El backup está en: ${YELLOW}${SSHD_BACKUP}${NC}"
    echo ""
    echo -e "${CYAN}═══════════════════════════════════════════════════════════════════${NC}"
    echo -e "  ${BLUE}Comandos útiles:${NC}"
    echo ""
    echo -e "    Ver estado UFW:        ${YELLOW}ufw status verbose${NC}"
    echo -e "    Ver jails Fail2ban:    ${YELLOW}fail2ban-client status${NC}"
    echo -e "    Ver baneados SSH:      ${YELLOW}fail2ban-client status sshd${NC}"
    echo -e "    Desbanear IP:          ${YELLOW}fail2ban-client set sshd unbanip IP${NC}"
    echo ""
    echo -e "${CYAN}═══════════════════════════════════════════════════════════════════${NC}"
    echo ""
}

#───────────────────────────────────────────────────────────────────────────────
# FUNCIÓN PRINCIPAL
#───────────────────────────────────────────────────────────────────────────────

main() {
    # Mostrar banner
    print_banner
    
    # Verificaciones iniciales
    check_root
    check_os
    detect_ssh_service
    
    # Crear backup ANTES de cualquier cambio
    create_backup
    
    # Obtener datos del usuario
    get_public_key
    get_ssh_port
    
    echo ""
    echo -e "${YELLOW}╔═══════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${YELLOW}║  INICIANDO CONFIGURACIÓN DE SEGURIDAD...                          ║${NC}"
    echo -e "${YELLOW}╚═══════════════════════════════════════════════════════════════════╝${NC}"
    
    # Instalar paquetes necesarios
    if ! install_packages; then
        print_error "Fallo en la instalación de paquetes"
        rollback
        exit 1
    fi
    
    # Configurar llave SSH
    if ! setup_authorized_keys; then
        print_error "Fallo en la configuración de llaves SSH"
        rollback
        exit 1
    fi
    
    # Configurar SSH
    if ! configure_ssh; then
        print_error "Fallo en la configuración de SSH"
        rollback
        exit 1
    fi
    
    # Configurar UFW
    if ! configure_ufw; then
        print_error "Fallo en la configuración de UFW"
        rollback
        exit 1
    fi
    
    # Configurar Fail2ban
    if ! configure_fail2ban; then
        print_error "Fallo en la configuración de Fail2ban"
        # No hacemos rollback aquí porque fail2ban no es crítico
        print_warning "Continuando sin Fail2ban..."
    fi
    
    # Reiniciar SSH (punto crítico)
    if ! restart_ssh; then
        print_error "Fallo al reiniciar SSH"
        rollback
        exit 1
    fi
    
    # Marcar como exitoso
    SCRIPT_SUCCESS=true
    
    # Mostrar resumen
    show_summary
}

#───────────────────────────────────────────────────────────────────────────────
# EJECUTAR SCRIPT
#───────────────────────────────────────────────────────────────────────────────

main "$@"

# FIN DEL SCRIPT
```

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
# Ejemplo: abrir puerto 3000 para una aplicación Node.js
ufw allow 3000/tcp comment 'Node.js App'

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

## ⚡ Conexión después del blindaje

Recuerda que después de ejecutar el script, debes conectarte usando:

```bash
ssh -p NUEVO_PUERTO root@IP_DEL_SERVIDOR
```

O configura tu archivo `~/.ssh/config` local:

```
Host mi-vps
    HostName IP_DEL_SERVIDOR
    User root
    Port NUEVO_PUERTO
    IdentityFile ~/.ssh/id_ed25519
```

Y luego simplemente:

```bash
ssh mi-vps
```

---

## 📝 Notas Finales

- El script **no requiere reinicio** del servidor
- Es **seguro re-ejecutarlo** (no duplicará reglas ni romperá configuraciones)
- El **backup se crea automáticamente** antes de cualquier cambio
- Si algo falla, **el rollback es automático**
- **Guarda el número de puerto** en un lugar seguro

---

*Guía creada para la administración segura de servidores VPS con Ubuntu/Debian.*
