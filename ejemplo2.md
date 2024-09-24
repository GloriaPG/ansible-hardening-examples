Ejemplo de Playbook para Instalar un Antivirus en Windows

Si vas a instalar un antivirus como Microsoft Defender o cualquier otro antivirus a partir de un archivo .exe o .msi, puedes hacerlo de la siguiente manera:

Playbook: instalar_antivirus_windows.yml
```
---
- name: Instalar antivirus en Windows
  hosts: servidores_windows
  gather_facts: no
  become: yes  # Elevar privilegios en Windows
  tasks:

    - name: Descargar el instalador del antivirus (Ejemplo: un antivirus de terceros)
      ansible.windows.win_get_url:
        url: "https://example.com/antivirus_installer.exe"
        dest: "C:\\Temp\\antivirus_installer.exe"

    - name: Ejecutar el instalador del antivirus
      ansible.windows.win_package:
        path: "C:\\Temp\\antivirus_installer.exe"
        arguments: "/quiet /norestart"
        state: present

    - name: Verificar si el antivirus está instalado
      ansible.windows.win_feature:
        name: Windows-Defender-Features
        state: present
        when: ansible_distribution_version >= "10"

    - name: Reiniciar si es necesario después de la instalación
      ansible.windows.win_reboot:
        msg: "Reiniciando después de instalar el antivirus."
        when: ansible_pkg_mgr_reboot_required
```

Explicación del Playbook:


Descargar el instalador del antivirus:

Utiliza el módulo win_get_url para descargar el instalador del antivirus desde una URL y guardarlo en la carpeta C:\Temp\ en el sistema Windows. Aquí puedes usar la URL del instalador de tu antivirus específico.
Instalar el antivirus:

El módulo win_package se utiliza para ejecutar el instalador del antivirus que has descargado. El archivo ejecutable se encuentra en C:\Temp\antivirus_installer.exe, y los argumentos /quiet /norestart aseguran que la instalación se realice de manera silenciosa y sin reiniciar automáticamente. Cambia estos argumentos según las opciones de instalación del antivirus que estés usando.
Verificar si Microsoft Defender está activado (opcional):

Este paso es opcional y se usa para verificar si las características de Windows Defender (Microsoft Defender) están activadas en versiones modernas de Windows (como Windows 10 o superior). Usa el módulo win_feature para asegurarte de que está presente. Este paso puede omitirse si estás instalando otro antivirus.
Reiniciar si es necesario:

El módulo win_reboot reinicia el servidor si el sistema requiere un reinicio después de la instalación del antivirus. El comando se ejecuta solo si ansible_pkg_mgr_reboot_required es verdadero, lo cual indica que un reinicio es necesario.
