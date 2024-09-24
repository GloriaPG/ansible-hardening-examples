La regla del Centro de Seguridad de Internet (CIS), recomendada por el Instituto SANS, establece que las cuentas privilegiadas deben ser únicas según el rol y perfil del usuario. Para cumplir con esta regla, las cuentas de administración (root, sudoers, etc.) deben configurarse y gestionarse de manera que cada cuenta tenga privilegios específicos, y no se compartan entre múltiples usuarios. Además, estas cuentas deben estar claramente diferenciadas por roles y responsabilidades.

A continuación, te doy un ejemplo de playbook de Ansible para cumplir con esta recomendación. Este playbook:

1. Crea cuentas privilegiadas basadas en roles específicos.
2. Configura sudoers para garantizar que los privilegios de administración se otorgan de acuerdo con el rol del usuario.
3. Asegura que no haya cuentas compartidas para roles críticos, como administradores de sistemas o bases de datos.

```
---
- name: Cumplir con la regla CIS sobre cuentas privilegiadas
  hosts: servidores_linux
  become: yes
  vars:
    privileged_users:
      - { username: "admin_sys", role: "Administrador del sistema", shell: "/bin/bash", sudo_privileges: true }
      - { username: "admin_db", role: "Administrador de base de datos", shell: "/bin/bash", sudo_privileges: false }
      - { username: "admin_network", role: "Administrador de red", shell: "/bin/bash", sudo_privileges: true }
  tasks:
  
    # 1. Crear usuarios privilegiados con nombres únicos basados en roles
    - name: Crear usuarios privilegiados de acuerdo a su rol
      ansible.builtin.user:
        name: "{{ item.username }}"
        comment: "{{ item.role }}"
        shell: "{{ item.shell }}"
        state: present
      loop: "{{ privileged_users }}"
    
    # 2. Configurar sudoers para los usuarios que requieren privilegios de sudo
    - name: Configurar sudoers para usuarios con privilegios administrativos
      ansible.builtin.lineinfile:
        path: /etc/sudoers
        state: present
        create: yes
        regexp: "^{{ item.username }}"
        line: "{{ item.username }} ALL=(ALL) NOPASSWD:ALL"
      loop: "{{ privileged_users | selectattr('sudo_privileges', 'equalto', true) }}"

    # 3. Asegurar que no existan cuentas compartidas de root o administrador
    - name: Verificar que no hay cuentas compartidas de root o usuarios administradores duplicados
      ansible.builtin.command: "grep -E '^root:|admin' /etc/passwd"
      register: admin_accounts
      changed_when: false

    - name: Mostrar los usuarios administrativos existentes
      ansible.builtin.debug:
        var: admin_accounts.stdout_lines

    - name: Asegurar que root solo tiene un único usuario
      ansible.builtin.fail:
        msg: "Se detectó una cuenta de root compartida o duplicada"
      when: admin_accounts.stdout_lines | length > 1

    # 4. Configurar políticas adicionales de seguridad para usuarios administrativos
    - name: Asegurarse de que las contraseñas de los usuarios privilegiados expiran
      ansible.builtin.command: "chage -M 90 {{ item.username }}"
      loop: "{{ privileged_users }}"
    
    - name: Asegurarse de que los usuarios administrativos bloquean la cuenta después de 5 intentos fallidos
      ansible.builtin.lineinfile:
        path: /etc/security/faillock.conf
        regexp: '^#?deny'
        line: 'deny=5'
        state: present

```

### Explicación del Playbook:
1. Creación de usuarios privilegiados:
Utiliza la variable privileged_users para definir usuarios basados en roles específicos como admin_sys, admin_db, y admin_network.
Cada usuario tiene asignado un comentario con su rol y un shell específico. Los usuarios son creados o actualizados en el sistema.

2. Configuración de sudoers:

Para los usuarios que requieren privilegios de sudo (sudo_privileges: true), se les otorga acceso completo en el archivo /etc/sudoers con la directiva NOPASSWD, para que no necesiten proporcionar su contraseña cada vez que utilicen sudo.
Verificación de cuentas compartidas:

Se verifica que no haya cuentas de root duplicadas o compartidas. Si se encuentran varias entradas relacionadas con root en el archivo /etc/passwd, el playbook fallará, indicando una posible vulnerabilidad de seguridad.
Si la verificación encuentra más de una cuenta de root o usuarios duplicados con privilegios, detendrá la ejecución del playbook y mostrará un mensaje de error.

3. Políticas adicionales de seguridad:

Contraseñas expiran: Se asegura que las contraseñas de los usuarios privilegiados expiran después de 90 días, usando chage -M 90.
Bloqueo después de intentos fallidos: Se configura el archivo /etc/security/faillock.conf para bloquear la cuenta después de 5 intentos fallidos de autenticación.

