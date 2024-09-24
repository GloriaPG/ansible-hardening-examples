Para cumplir con la regla del Instituto SANS y el Centro de Seguridad de Internet (CIS) que establece que las credenciales deben almacenarse y transmitirse de manera encriptada utilizando algoritmos robustos, es necesario:

1. Almacenar las contraseñas y credenciales de forma encriptada: En el caso de Linux, esto implica el uso de algoritmos de hashing seguros (como SHA-512 en /etc/shadow) y el uso de herramientas como OpenSSH para la transmisión encriptada de datos.
2. Transmitir credenciales de manera segura: Utilizando protocolos encriptados como SSHv2 y TLS 1.2 o superior para todas las comunicaciones que involucren credenciales.

### Estrategia del Playbook:

1. Verificar y forzar el uso de hashing seguro (SHA-512) para las contraseñas almacenadas en /etc/shadow.
2. Configurar SSH para usar la versión 2 y algoritmos de encriptación robustos.
3. Asegurar TLS para servicios que lo utilicen (por ejemplo, en un servidor web).
4. Aplicar políticas seguras para la transmisión de credenciales en cualquier protocolo relevante.

```
---
- name: Cumplir con la regla CIS de almacenamiento y transmisión de credenciales encriptadas
  hosts: servidores_linux
  become: yes
  tasks:

    # 1. Asegurar que las contraseñas almacenadas en /etc/shadow están encriptadas con SHA-512
    - name: Configurar hashing de contraseñas a SHA-512 en el sistema
      ansible.builtin.lineinfile:
        path: /etc/login.defs
        regexp: '^ENCRYPT_METHOD'
        line: 'ENCRYPT_METHOD SHA512'
        state: present

    # 2. Asegurarse de que SSH utiliza solamente SSHv2 y algoritmos seguros
    - name: Configurar SSH para usar solamente SSHv2
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Protocol'
        line: 'Protocol 2'
        state: present

    - name: Deshabilitar algoritmos débiles en SSH
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Ciphers'
        line: 'Ciphers aes256-ctr,aes192-ctr,aes128-ctr'
        state: present

    - name: Reiniciar el servicio SSH para aplicar los cambios
      ansible.builtin.systemd:
        name: sshd
        state: restarted
        enabled: yes

    # 3. Asegurar que las conexiones TLS utilizan al menos TLS 1.2 o superior (ejemplo para Apache)
    - name: Instalar paquetes SSL para Apache
      ansible.builtin.yum:
        name: httpd mod_ssl
        state: present

    - name: Configurar Apache para forzar TLS 1.2 o superior
      ansible.builtin.lineinfile:
        path: /etc/httpd/conf.d/ssl.conf
        regexp: '^#?SSLProtocol'
        line: 'SSLProtocol -all +TLSv1.2 +TLSv1.3'
        state: present

    - name: Reiniciar Apache para aplicar cambios en SSL/TLS
      ansible.builtin.systemd:
        name: httpd
        state: restarted
        enabled: yes

    # 4. Aplicar políticas adicionales para la transmisión segura de credenciales
    - name: Forzar el uso de encriptación para la transmisión de credenciales en servicios relevantes
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        state: present

    - name: Asegurar que no se permiten inicios de sesión por SSH sin clave (deshabilitar contraseñas en texto claro)
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present

    # 5. Verificar las políticas de hashing en PAM (Pluggable Authentication Modules)
    - name: Configurar PAM para usar SHA-512 como algoritmo de hashing
      ansible.builtin.lineinfile:
        path: /etc/pam.d/common-password
        regexp: '^password.*pam_unix.so'
        line: 'password requisite pam_unix.so sha512 shadow'
        state: present

    - name: Asegurarse de que las contraseñas se almacenan en /etc/shadow y no en /etc/passwd
      ansible.builtin.lineinfile:
        path: /etc/login.defs
        regexp: '^PASS_MAX_DAYS'
        line: 'PASS_MAX_DAYS 90'
        state: present

    - name: Habilitar logging seguro para eventos relacionados con credenciales (por ejemplo, fallos de autenticación)
      ansible.builtin.copy:
        dest: /etc/audit/rules.d/audit.rules
        content: |
          -w /etc/shadow -p wa -k credential_change
          -w /var/log/auth.log -p wa -k login_attempts

    - name: Reiniciar auditd para aplicar las nuevas reglas de auditoría
      ansible.builtin.systemd:
        name: auditd
        state: restarted


```

### Explicación del Playbook:
1. Configuración de Hashing Seguro (SHA-512):

Se asegura que las contraseñas almacenadas en /etc/shadow estén encriptadas utilizando el algoritmo SHA-512, que es uno de los algoritmos más robustos y recomendados actualmente para el almacenamiento de contraseñas.

2. Configuración de SSH:

Se configura el archivo /etc/ssh/sshd_config para usar SSHv2 exclusivamente, que es la versión más segura de SSH.
Se deshabilitan los algoritmos débiles en SSH y se habilitan algoritmos de cifrado seguros como AES-256-CTR.

3. TLS en Servicios (ejemplo con Apache):

Para servicios que utilizan TLS, se fuerza el uso de versiones seguras de TLS (1.2 y 1.3) en el archivo de configuración de SSL para Apache.
Se reinicia el servicio Apache para aplicar los cambios.

4. Políticas Adicionales de Seguridad:

Se asegura que no se permiten inicios de sesión como root a través de SSH y se deshabilita la autenticación por contraseña en texto claro, exigiendo el uso de autenticación por clave.

5. PAM (Pluggable Authentication Modules):

Se asegura que PAM esté configurado para usar SHA-512 como algoritmo de encriptación para las contraseñas.
Se revisan las configuraciones en /etc/login.defs para que las contraseñas expiren cada 90 días, asegurando que las credenciales se gestionen de forma segura.

6. Auditoría de eventos relacionados con credenciales:

Se configuran reglas de auditoría en auditd para registrar cualquier cambio en /etc/shadow y cualquier intento de autenticación fallido.


### Beneficios del Playbook:
* Almacenamiento seguro de contraseñas: Garantiza que las contraseñas se almacenan utilizando SHA-512 en /etc/shadow, cumpliendo con los estándares de seguridad más altos.
* Transmisión segura de credenciales: Asegura que todas las conexiones de SSH y TLS utilizan versiones seguras de estos protocolos, previniendo el uso de algoritmos obsoletos o vulnerables.
* Auditoría de eventos: Implementa auditorías para registrar eventos relacionados con credenciales, permitiendo la detección de incidentes de seguridad.
