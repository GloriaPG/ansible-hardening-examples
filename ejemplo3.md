Este playbook obtendrá los paquetes instalados en RHEL y las aplicaciones instaladas en Windows, luego imprimirá los resultados y esos resultados se pueden mandar a un archivo .csv, por poner ejemplo.

```
---
- name: Obtener inventarios de paquetes y aplicaciones instalados en RHEL y Windows
  hosts: servidores_rhel:servidores_windows
  gather_facts: no
  tasks:

  # Inventario para servidores RHEL
  - name: Obtener inventario de paquetes instalados en RHEL
    when: ansible_facts['os_family'] == 'RedHat'
    ansible.builtin.shell: "yum list installed"
    register: rhel_packages

  - name: Mostrar los paquetes instalados en RHEL
    when: ansible_facts['os_family'] == 'RedHat'
    ansible.builtin.debug:
      var: rhel_packages.stdout_lines

  # Inventario para servidores Windows
  - name: Obtener inventario de aplicaciones instaladas en Windows
    when: ansible_facts['os_family'] == 'Windows'
    ansible.windows.win_package_facts:

  - name: Mostrar las aplicaciones instaladas en Windows
    when: ansible_facts['os_family'] == 'Windows'
    ansible.builtin.debug:
      var: ansible_facts.packages
```


Explicación del Playbook:

1. Hosts:
Este playbook se ejecuta en servidores RHEL y Windows, agrupados en dos grupos de inventario: servidores_rhel y servidores_windows. Puedes personalizar estos nombres según tu entorno.

2. Obtener paquetes instalados en RHEL:

Se usa el módulo shell con el comando yum list installed para obtener la lista de paquetes instalados en sistemas basados en RHEL. Luego, se registra el resultado en la variable rhel_packages.
La tarea debug muestra el contenido de rhel_packages.stdout_lines para visualizar los paquetes instalados en el servidor.

3. Obtener aplicaciones instaladas en Windows:

 El módulo win_package_facts recupera las aplicaciones instaladas en el sistema Windows.
La tarea debug muestra la variable ansible_facts.packages, que contiene los programas instalados.
