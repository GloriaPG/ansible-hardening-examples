La instalación del agente de Tenable en sistemas Windows y RHEL usando Ansible es un proceso que puede automatizarse utilizando los instaladores provistos por Tenable. 
Estos agentes ayudan a realizar análisis de vulnerabilidades, los cuales pueden formar parte de pruebas DAST (Dynamic Application Security Testing) y SAST (Static Application Security Testing).

A continuación te proporciono ejemplos de playbooks de Ansible que instalan el Tenable Agent en ambos sistemas operativos.

```
---
- name: Instalar Tenable Agent en RHEL
  hosts: servidores_rhel
  become: yes
  tasks:

    - name: Descargar el instalador del agente de Tenable
      ansible.builtin.get_url:
        url: "https://downloads.nessus.org/nessusagent/downloads/10.0.0/RedHatEL6/nessusagent-10.0.0-es6.x86_64.rpm"
        dest: "/tmp/nessusagent.rpm"

    - name: Instalar el agente de Tenable
      ansible.builtin.yum:
        name: /tmp/nessusagent.rpm
        state: present

    - name: Iniciar y habilitar el servicio de Tenable Agent
      ansible.builtin.systemd:
        name: nessusagent
        enabled: yes
        state: started

    - name: Registrar el agente en Tenable.io (reemplaza con tu información)
      ansible.builtin.command:
        cmd: "/opt/nessus_agent/sbin/nessuscli agent link --key=<ACTIVATION_KEY> --groups='Grupo RHEL' --host='cloud.tenable.com' --port=443"
      args:
        creates: "/opt/nessus_agent/sbin/nessuscli"
```

Explicación del Playbook para RHEL:

Descargar el instalador del agente: Utiliza el módulo get_url para descargar el paquete RPM del agente de Tenable desde el repositorio oficial.
Instalar el agente: Usa yum para instalar el paquete descargado.
Iniciar y habilitar el servicio: Inicia y habilita el servicio nessusagent para que se ejecute automáticamente en cada reinicio.
Registrar el agente: Registra el agente en el servidor Tenable.io con una clave de activación y agrégalo a un grupo específico. Reemplaza <ACTIVATION_KEY> por tu clave de activación real.

2. Playbook para Instalar el Agente de Tenable en Windows
   
```
---
- name: Instalar Tenable Agent en Windows
  hosts: servidores_windows
  become: yes
  tasks:

    - name: Descargar el instalador del agente de Tenable para Windows
      ansible.windows.win_get_url:
        url: "https://downloads.nessus.org/nessusagent/downloads/10.0.0/NessusAgent-10.0.0-x64.msi"
        dest: "C:\\Temp\\NessusAgent.msi"

    - name: Instalar el agente de Tenable en Windows
      ansible.windows.win_package:
        path: "C:\\Temp\\NessusAgent.msi"
        state: present
        arguments: "/quiet"

    - name: Registrar el agente en Tenable.io (reemplaza con tu información)
      ansible.windows.win_shell:
        command: "C:\\Program Files\\Tenable\\Nessus Agent\\nessuscli.exe agent link --key=<ACTIVATION_KEY> --groups='Grupo Windows' --host='cloud.tenable.com' --port=443"

    - name: Iniciar el servicio del agente en Windows
      ansible.windows.win_service:
        name: "Tenable Nessus Agent"
        start_mode: auto
        state: started

```


Explicación del Playbook para Windows:

Descargar el instalador del agente: Usa win_get_url para descargar el instalador MSI del agente de Tenable.
Instalar el agente: Con win_package, instala el agente de manera silenciosa utilizando el argumento /quiet.
Registrar el agente: Usa win_shell para ejecutar el comando que registra el agente en Tenable.io. Reemplaza <ACTIVATION_KEY> con tu clave de activación.
Iniciar el servicio del agente: Asegura que el servicio del agente esté iniciado y configurado para iniciarse automáticamente con cada reinicio.
