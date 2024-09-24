Para cumplir con la regla del Instituto SANS y el Centro de Seguridad de Internet (CIS) que establece la necesidad de generar alertas sobre intentos fallidos de inicio de sesión con cuentas administrativas, podemos utilizar herramientas como auditd y faillock. Estas herramientas permiten registrar eventos de inicio de sesión fallidos y generar notificaciones o alertas.

###Estrategia:

1. Configurar auditd para registrar eventos de autenticación fallidos de cuentas administrativas.
2. Configurar faillock para registrar intentos fallidos de autenticación.
3. Configurar un script o servicio para generar alertas, por ejemplo, enviando un correo electrónico o utilizando herramientas de monitoreo centralizado como Splunk, ELK Stack, etc.

### Ejemplo de Playbook de Ansible:
Este playbook te ayudará a:

* Configurar auditd para registrar eventos de inicio de sesión fallidos.
* Utilizar faillock para limitar los intentos de autenticación y registrar los fallos.
* Implementar un script para generar una alerta vía correo electrónico cuando haya un intento fallido con una cuenta administrativa.

```
---
- name: Configurar registro y alerta para intentos fallidos de inicio de sesión de cuentas administrativas
  hosts: servidores_linux
  become: yes
  vars:
    admin_accounts: 
      - root
      - admin_sys
      - admin_db
      - admin_network
    alert_email: "admin@empresa.com"  # Cambia a tu dirección de correo
  tasks:

    # 1. Instalar y habilitar auditd
    - name: Instalar auditd
      ansible.builtin.yum:
        name: audit
        state: present

    - name: Iniciar y habilitar auditd
      ansible.builtin.systemd:
        name: auditd
        state: started
        enabled: true

    # 2. Configurar auditd para registrar intentos fallidos de login
    - name: Configurar auditd para registrar intentos fallidos de login
      ansible.builtin.lineinfile:
        path: /etc/audit/rules.d/audit.rules
        state: present
        create: yes
        line: '-w /var/log/faillog -p wa -k login_failures'

    - name: Reiniciar auditd para aplicar reglas
      ansible.builtin.systemd:
        name: auditd
        state: restarted

    # 3. Configurar faillock para limitar y registrar intentos fallidos
    - name: Configurar faillock para registrar y limitar intentos fallidos de autenticación
      ansible.builtin.lineinfile:
        path: /etc/security/faillock.conf
        regexp: '^#?deny'
        line: 'deny=5'
        state: present

    - name: Configurar el tiempo de bloqueo de cuentas tras intentos fallidos
      ansible.builtin.lineinfile:
        path: /etc/security/faillock.conf
        regexp: '^#?unlock_time'
        line: 'unlock_time=600'
        state: present

    # 4. Crear script para alertar sobre intentos fallidos de cuentas administrativas
    - name: Crear script para alerta de intentos fallidos de inicio de sesión
      ansible.builtin.copy:
        dest: /usr/local/bin/alerta_login_fallido.sh
        mode: '0755'
        content: |
          #!/bin/bash
          admin_accounts="{{ admin_accounts | join(' ') }}"
          faillog_output=$(faillock --user $admin_accounts)

          if [ ! -z "$faillog_output" ]; then
              echo "Se detectaron intentos fallidos de inicio de sesión en cuentas administrativas: $faillog_output" \
              | mail -s "Alerta: Intentos fallidos de login" {{ alert_email }}
          fi

    # 5. Crear un cron job para ejecutar el script de alerta periódicamente
    - name: Crear cron job para ejecutar el script cada 5 minutos
      ansible.builtin.cron:
        name: "Alerta de intentos fallidos de login administrativo"
        job: "/usr/local/bin/alerta_login_fallido.sh"
        minute: "*/5"
```


### Explicación del Playbook:

1. Instalar y configurar auditd:
Se instala el servicio auditd para auditar y registrar eventos en el sistema.
Se añade una regla a auditd para monitorear el archivo /var/log/faillog, que contiene información sobre los intentos fallidos de autenticación.
Configurar faillock:

2. Se ajusta la herramienta faillock para limitar los intentos fallidos de inicio de sesión a 5, y bloquear la cuenta por 10 minutos (600 segundos) después de intentos fallidos.
Esta configuración asegura que, además de registrar los fallos, también haya un bloqueo temporal de la cuenta después de varios intentos.
Crear un script de alerta:

3. Se crea un script en /usr/local/bin/alerta_login_fallido.sh que revisa si alguna de las cuentas administrativas ha tenido intentos fallidos de autenticación.
Si se detectan intentos fallidos, el script envía un correo electrónico a la dirección definida en la variable alert_email con los detalles del evento.
Configurar un cron job:

4. Se configura un cron job que ejecuta el script de alerta cada 5 minutos, asegurando una detección rápida de intentos fallidos de inicio de sesión.
   
### Adaptación a tus necesidades:
* Dirección de correo electrónico: Cambia el valor de alert_email a la dirección donde deseas recibir las alertas.
* Frecuencia de la alerta: El cron job está configurado para ejecutarse cada 5 minutos, pero puedes ajustar este valor según tus necesidades.
* Cuentas administrativas: La variable admin_accounts contiene los nombres de las cuentas administrativas que deseas monitorear. Puedes modificarla para incluir las cuentas relevantes en tu sistema.


Este playbook te ayudará a implementar un sistema automatizado de monitoreo y alerta sobre intentos fallidos de inicio de sesión de cuentas administrativas, cumpliendo con las reglas de seguridad recomendadas por el Instituto SANS y CIS.
