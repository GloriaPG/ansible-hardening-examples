```
---
- name: Configurar sudoers para usuarios y grupos en RHEL
  hosts: servidores_rhel
  become: yes
  tasks:
    
    - name: Dar permisos de sudo sin contraseña al grupo "wheel"
      ansible.builtin.lineinfile:
        path: /etc/sudoers
        state: present
        regexp: '^%wheel'
        line: '%wheel ALL=(ALL) NOPASSWD:ALL'
        validate: '/usr/sbin/visudo -cf %s'

    - name: Permitir que el grupo "wheel" ejecute solo comandos específicos
      ansible.builtin.lineinfile:
        path: /etc/sudoers
        state: present
        insertafter: '^%wheel ALL=(ALL) NOPASSWD:ALL'
        line: '%wheel ALL=(ALL) /usr/bin/systemctl restart httpd, /usr/bin/systemctl restart nginx'
        validate: '/usr/sbin/visudo -cf %s'

    - name: Denegar el uso de sudo a un usuario específico
      ansible.builtin.lineinfile:
        path: /etc/sudoers
        state: present
        insertafter: '^%wheel ALL=(ALL) NOPASSWD:ALL'
        line: 'usuario_test ALL=(ALL) !ALL'
        validate: '/usr/sbin/visudo -cf %s'
```


Explicación del Playbook:

Dar permisos de sudo sin contraseña al grupo wheel:

Usa lineinfile para modificar el archivo /etc/sudoers y permitir que todos los miembros del grupo wheel puedan ejecutar cualquier comando con sudo sin necesidad de ingresar su contraseña.
La línea que agrega es: %wheel ALL=(ALL) NOPASSWD:ALL.
Restringir el uso de sudo a comandos específicos:

En este caso, también se configura el archivo sudoers para que el grupo wheel pueda ejecutar solo ciertos comandos con sudo, como reiniciar los servicios httpd y nginx usando systemctl.
La línea que agrega es: %wheel ALL=(ALL) /usr/bin/systemctl restart httpd, /usr/bin/systemctl restart nginx.
Denegar el uso de sudo a un usuario específico:

En el último paso, se configura para denegar el uso de sudo a un usuario específico (usuario_test). La línea que se añade es: usuario_test ALL=(ALL) !ALL, lo que impide a ese usuario ejecutar cualquier comando con sudo.
Validación con visudo:
El parámetro validate: '/usr/sbin/visudo -cf %s' se utiliza para asegurarse de que el archivo /etc/sudoers sea válido antes de aplicar los cambios, evitando así posibles errores de sintaxis que puedan bloquear el uso de sudo.


Para windows:

```

---
- name: Administración de usuarios y permisos en Windows con Ansible
  hosts: servidores_windows
  gather_facts: no
  tasks:

    - name: Crear un usuario en Windows
      ansible.windows.win_user:
        name: usuario_test
        password: "P@ssw0rd!"
        state: present
        password_expired: no
        groups:
          - Administrators
        description: "Usuario de prueba para administración"
        
    - name: Crear un grupo personalizado
      ansible.windows.win_group:
        name: GrupoPersonalizado
        description: "Este es un grupo personalizado"
        state: present

    - name: Agregar usuario al grupo personalizado
      ansible.windows.win_group_membership:
        name: GrupoPersonalizado
        members: 
          - usuario_test
        state: present

    - name: Crear una carpeta
      ansible.windows.win_file:
        path: C:\Users\Public\CarpetaCompartida
        state: directory

    - name: Asignar permisos al usuario sobre la carpeta
      ansible.windows.win_acl:
        path: C:\Users\Public\CarpetaCompartida
        user: usuario_test
        permission: fullcontrol
        inherit: yes

    - name: Asignar permisos solo de lectura al grupo personalizado sobre la carpeta
      ansible.windows.win_acl:
        path: C:\Users\Public\CarpetaCompartida
        user: GrupoPersonalizado
        permission: read
        inherit: yes

    - name: Remover usuario de un grupo
      ansible.windows.win_group_membership:
        name: Administrators
        members: 
          - usuario_test
        state: absent
```

Explicación del Playbook:

Crear un usuario en Windows:

Utiliza el módulo win_user para crear un usuario llamado usuario_test con la contraseña P@ssw0rd!. Este usuario se agrega al grupo Administrators y tiene una descripción.
Crear un grupo personalizado:

Usa win_group para crear un grupo llamado GrupoPersonalizado en Windows.
Agregar usuario al grupo personalizado:

Con el módulo win_group_membership, el usuario usuario_test se agrega al grupo GrupoPersonalizado.
Crear una carpeta:

Utiliza el módulo win_file para crear una carpeta en C:\Users\Public\CarpetaCompartida.
Asignar permisos de control total a un usuario:

El módulo win_acl se utiliza para asignar permisos de control total (fullcontrol) al usuario usuario_test sobre la carpeta creada.
Asignar permisos de solo lectura a un grupo:

Con win_acl, se asignan permisos de solo lectura (read) al grupo GrupoPersonalizado sobre la misma carpeta.
Remover un usuario de un grupo:

El módulo win_group_membership se usa para remover al usuario usuario_test del grupo Administrators.

