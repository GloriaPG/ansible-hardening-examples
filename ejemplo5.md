Para habilitar Microsoft Defender for Cloud (anteriormente llamado Microsoft Cloud Defender o Azure Security Center) en una máquina virtual de Windows o Linux usando Ansible, necesitarás interactuar con Azure a través de módulos específicos de Ansible que manejan la configuración de Microsoft Defender en tu suscripción de Azure.

El proceso general involucra:

1. Habilitar Microsoft Defender for Cloud en la suscripción de Azure.
2. Instalar y habilitar Microsoft Defender en máquinas virtuales, tanto en Windows como en Linux.

Para interactuar con Azure, Ansible utiliza el módulo azure_rm. También necesitarás autenticarte usando Azure Service Principal o Azure Managed Identity.

### 1. Playbook para Habilitar Microsoft Defender for Cloud en la Suscripción de Azure (Así no lo tienen que hacer máquina, por máquina)
El siguiente playbook habilita Microsoft Defender for Cloud para una suscripción de Azure.
```
---
- name: Habilitar Microsoft Defender for Cloud en Azure
  hosts: localhost
  connection: local
  tasks:

    - name: Habilitar Defender para todas las capacidades en la suscripción
      azure.azcollection.azure_rm_security_center_subscription_pricing:
        name: 'Default'
        tier: 'Standard'  # 'Free' or 'Standard'
        azure_credentials: "{{ lookup('azure_credential') }}"
      register: defender_status

    - name: Mostrar estado de Microsoft Defender
      ansible.builtin.debug:
        msg: "Estado de Microsoft Defender: {{ defender_status }}"

```
### Explicación del Playbook:
Habilitar Microsoft Defender for Cloud en la suscripción:

1. Utiliza el módulo azure_rm_security_center_subscription_pricing para activar el servicio. Puedes elegir el nivel de seguridad entre Free y Standard. El nivel Standard incluye funcionalidades como protección avanzada contra amenazas.
2. La autenticación se gestiona usando azure_credentials, que puedes configurar con un Azure Service Principal.
3. Mostrar estado de Microsoft Defender: Muestra en la consola el resultado de la habilitación.

### 2. Playbook para Habilitar Microsoft Defender en Máquinas Virtuales (Windows/Linux)
Una vez que has habilitado Microsoft Defender en la suscripción de Azure, puedes habilitar el agente de Microsoft Defender en las máquinas virtuales. Microsoft Defender debería instalarse y habilitarse automáticamente cuando activas Defender en la suscripción, pero también puedes hacerlo manualmente.
*** a) Para Máquinas Virtuales Windows
```
---
- name: Habilitar Microsoft Defender en una VM de Windows
  hosts: servidores_windows
  become: yes
  tasks:

    - name: Instalar Microsoft Defender si no está presente
      ansible.windows.win_feature:
        name: "Windows-Defender-Features"
        state: present

    - name: Iniciar y habilitar Microsoft Defender
      ansible.windows.win_service:
        name: "WinDefend"
        state: started
        start_mode: auto

    - name: Verificar el estado de Microsoft Defender en Windows
      ansible.windows.win_shell:
        command: "sc query WinDefend"
      register: defender_status

    - name: Mostrar el estado del servicio Microsoft Defender en Windows
      ansible.builtin.debug:
        var: defender_status.stdout_lines
```

*** b) Para Máquinas Virtuales RHEL (Linux)
En las distribuciones de Linux, se puede habilitar Microsoft Defender utilizando el paquete OMS Agent de Microsoft.
```
---
- name: Habilitar Microsoft Defender en una VM de RHEL (Linux)
  hosts: servidores_rhel
  become: yes
  tasks:

    - name: Descargar e instalar el OMS Agent para Microsoft Defender
      ansible.builtin.yum:
        name: https://github.com/microsoft/OMS-Agent-for-Linux/releases/download/OMSAgent_v1.13.35/omsagent-1.13.35-0.universal.x64.sh
        state: present

    - name: Ejecutar el script de instalación del OMS Agent
      ansible.builtin.shell: "sh /path/to/omsagent-1.13.35-0.universal.x64.sh --install"
      args:
        creates: "/opt/microsoft/omsagent"

    - name: Verificar si el agente está corriendo
      ansible.builtin.systemd:
        name: "omsagent"
        state: started
        enabled: yes

    - name: Mostrar el estado del servicio OMS Agent (Microsoft Defender)
      ansible.builtin.systemd:
        name: "omsagent"
        state: started
        enabled: yes
      register: defender_status

    - name: Mostrar el estado del OMS Agent en RHEL
      ansible.builtin.debug:
        var: defender_status
```

Explicación del Playbook:
*** a) Windows:
1. Instalar Microsoft Defender: Usa el módulo win_feature para instalar la funcionalidad de Microsoft Defender en Windows.
2. Iniciar y habilitar el servicio: Usa win_service para asegurarte de que el servicio WinDefend esté activo y configurado para iniciarse automáticamente.
3. Verificar el estado: Ejecuta un comando sc query para verificar el estado del servicio de Microsoft Defender.
*** b) RHEL (Linux):
1. Instalar OMS Agent: Descarga e instala el agente OMS (Operations Management Suite), que incluye la protección de Microsoft Defender para máquinas Linux.
2. Habilitar el servicio: Usa systemd para iniciar y habilitar el servicio del agente en Linux.
3. Verificar el estado: Se asegura de que el servicio omsagent esté corriendo y habilitado.

