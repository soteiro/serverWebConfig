# Server Provisioning Automation (CentOS 9/10)

Este repositorio contiene la automatización necesaria para configurar un servidor desde cero, dejándolo listo para servir aplicaciones web modernas (Node.js, Bun) con altos estándares de seguridad.

Está diseñado para ser ejecutado en servidores **CentOS 9 Stream** o **CentOS 10**.

## 🚀 ¿Qué hace este Playbook?

El proceso se divide en 4 fases automáticas:

### 1. Sistema y Rendimiento (`tasks/system.yml`)
*   **Actualización Total:** Ejecuta `dnf update` para asegurar que el SO tenga los últimos parches.
*   **Usuario dsoto:** Crea el usuario de despliegue con permisos `sudo` sin contraseña.
*   **SWAP de 2GB:** Configura memoria de intercambio para evitar que el servidor se caiga por falta de RAM (OOM Killer).
*   **SSH Keys:** Instala tu llave pública para acceso seguro.

### 2. Seguridad y Hardening (`tasks/security.yml`)
*   **Firewall (Iptables):** Cierra todos los puertos por defecto (**DROP**) y abre solo los esenciales (22, 80, 443).
*   **SSH Hardening:** Desactiva el acceso al usuario `root` y prohíbe el uso de contraseñas (solo llaves SSH).
*   **Fail2Ban:** Bloquea automáticamente IPs que intenten atacar el puerto SSH.
*   **SELinux:** Configurado para permitir que Nginx funcione como proxy hacia tu app.
*   **Kernel Hardening:** Ajustes de red para prevenir ataques tipo SYN flood y redirecciones maliciosas.

### 3. Entorno JavaScript Seguro (`tasks/js_env.yml`)
*   **Motores:** Instala Node.js 24 (LTS) y Bun.
*   **Gestores:** Instala PNPM (preferido sobre npm).
*   **Seguridad JS:** Bloquea globalmente los scripts `postinstall` (`ignore-scripts`), mitigando ataques en la cadena de suministro de paquetes.

### 4. Servidor Web (`tasks/web.yml`)
*   **Nginx Proxy:** Configurado para recibir tráfico en el puerto 80/443 y enviarlo a tu app en `localhost:3333`.
*   **HTTPS:** Redirección automática de HTTP a HTTPS.
*   **Caché:** Configuración de `max-age` de 1 semana para recursos estáticos.
*   **Let's Encrypt:** Instala Certbot para que puedas generar certificados reales fácilmente.

---

## 🛠 Guía para Principiantes (¿Cómo lo uso?)

No necesitas ser un experto en Ansible para usar esto. Sigue estos pasos:

### 1. Requisitos Previos en tu PC (Local)
Debes tener instalado Ansible en tu computadora:
*   **macOS:** `brew install ansible`
*   **Ubuntu/Debian:** `sudo apt install ansible`
*   **Windows:** Usa WSL2 (Ubuntu) e instala ansible ahí.

Asegúrate de tener tu llave SSH en `~/.ssh/melkor.pub`.

### 2. Preparar el Servidor (Remoto)
Debes tener la IP del servidor y acceso inicial como `root` (o un usuario con sudo).

### 3. Ejecutar la Magia
Desde la carpeta de este repositorio, corre el siguiente comando reemplazando `[IP_DEL_SERVER]` por la IP real:

```bash
    ansible-playbook provision.yml -e "host=[IP_DEL_SERVER]" -u root
```

*   `-e "host=..."`: Le dice a Ansible a qué servidor conectarse.
*   `-u root`: Indica que se conecte como root (solo la primera vez, después entrarás con `dsoto`).

---

## 📂 Estructura del Proyecto
*   `provision.yml`: El "director de orquesta".
*   `tasks/`: Contiene los módulos de configuración por separado.
*   `templates/`: Archivos de configuración que Ansible "rellena" dinámicamente (como el de Nginx).

## ⚠️ Notas Importantes
1.  **Certificados SSL:** El playbook genera un certificado temporal (autofirmado) para que Nginx arranque. Una vez que tu dominio apunte al server, corre:
    `sudo certbot --nginx -d tu-dominio.com`
2.  **Tu App:** Tu aplicación debe escuchar en el puerto **3333** para que Nginx pueda encontrarla.
3.  **Acceso SSH:** Una vez terminado el proceso, el acceso `root` quedará bloqueado. Deberás entrar como:
    `ssh dsoto@[IP_DEL_SERVER] -i ~/.ssh/melkor`
