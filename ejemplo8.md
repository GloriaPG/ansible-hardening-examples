Para asegurarte de que los usuarios y contraseñas en sistemas Linux se almacenen y transmitan de manera segura utilizando protocolos como SSHv2 y SSL/TLS 1.2 o superior, puedes configurar tanto SSH para la transmisión de contraseñas de manera segura, como servicios que usen SSL/TLS.

A continuación te presento un playbook de Ansible que cubre los siguientes aspectos:

1. Configurar SSH para asegurarse de que utiliza la versión 2 del protocolo.
2. Habilitar y configurar SSL/TLS 1.2 o superior en servicios como Apache (si fuera necesario).
3. Forzar autenticación en servicios a través de SSH y asegurarse de que las contraseñas están cifradas.

```
---
- name: Configurar SSH y SSL/TLS para asegurar el almacenamiento y transmisión de usuarios y contraseñas
  hosts: servidores_linux
  become: yes
  tasks:

    # 1. Configuración de SSH para usar SSHv2
    - name: Asegurarse de que SSH está usando el protocolo 2
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Protocol'
        line: 'Protocol 2'
        state: present
      notify: 
        - restart ssh

    - name: Asegurarse de que la autenticación por contraseña está permitida solo a través de SSH
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication yes'
        state: present
      notify: 
        - restart ssh

    - name: Desactivar autenticación por contraseña para el usuario root (recomendada)
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin prohibit-password'
        state: present
      notify: 
        - restart ssh

    # 2. Configuración de SSL/TLS 1.2 o superior en Apache (ejemplo)
    - name: Instalar Apache si no está presente (opcional)
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Asegurarse de que SSL/TLS 1.2 o superior está habilitado en Apache
      ansible.builtin.lineinfile:
        path: /etc/httpd/conf.d/ssl.conf
        regexp: '^#?SSLProtocol'
        line: 'SSLProtocol -all +TLSv1.2 +TLSv1.3'
        state: present
      notify:
        - restart apache

    - name: Deshabilitar cifrados inseguros en Apache
      ansible.builtin.lineinfile:
        path: /etc/httpd/conf.d/ssl.conf
        regexp: '^#?SSLCipherSuite'
        line: 'SSLCipherSuite HIGH:!aNULL:!MD5'
        state: present
      notify:
        - restart apache

    # 3. Asegurarse de que las contraseñas almacenadas están cifradas (utilizando SHA512 en shadow)
    - name: Verificar que las contraseñas se almacenan cifradas usando SHA512
      ansible.builtin.lineinfile:
        path: /etc/login.defs
        regexp: '^#?ENCRYPT_METHOD'
        line: 'ENCRYPT_METHOD SHA512'
        state: present

    # Handlers para reiniciar servicios
  handlers:
    - name: restart ssh
      ansible.builtin.systemd:
        name: sshd
        state: restarted

    - name: restart apache
      ansible.builtin.systemd:
        name: httpd
        state: restarted

```


### Explicación del Playbook

Configurar SSH para usar el protocolo SSHv2:

1. Se asegura de que el archivo de configuración de SSH /etc/ssh/sshd_config esté configurado para utilizar SSHv2, que es más seguro que SSHv1.
Se habilita la autenticación por contraseña a través de SSH, pero deshabilitamos el inicio de sesión con contraseña para el usuario root por seguridad adicional.
Configurar SSL/TLS 1.2 o superior (en este caso para Apache):

2. Si tienes servicios como Apache que necesitan SSL/TLS, se asegura de que se esté utilizando TLS 1.2 o superior y se desactivan los cifrados inseguros.
Las configuraciones de SSL y los cifrados se ajustan en el archivo /etc/httpd/conf.d/ssl.conf.
Almacenamiento seguro de contraseñas:

3. Configura el sistema para almacenar las contraseñas usando SHA512 (un algoritmo de hash fuerte) en el archivo /etc/login.defs.
   
### Consideraciones adicionales:
1. Reiniciar servicios: Después de realizar cambios en la configuración de SSH o Apache, se reinician los servicios respectivos para aplicar los cambios.
2. Seguridad adicional para root: Se deshabilita la autenticación por contraseña para el usuario root, lo que mejora la seguridad.
