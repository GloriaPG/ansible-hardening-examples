Para remover puertos inseguros en máquinas virtuales de Azure y AWS usando Ansible, puedes interactuar con los grupos de seguridad de red (NSG) en Azure y los grupos de seguridad (Security Groups) en AWS. Estos playbooks ayudarán a bloquear puertos inseguros (por ejemplo, el puerto 23 de Telnet, 21 de FTP, 445 de SMB, etc.)
También se puede realizar máquina por máquina, ya sea windows o Linux, sin embargo en gestión de cloud lo más usado es hacerlo por SG.
### 1. Playbook para Remover Puertos Inseguros en Azure

En Azure, gestionamos los grupos de seguridad de red (NSG). Para eliminar reglas que permiten puertos inseguros, usamos el módulo azure_rm_securitygroup.
```
---
- name: Remover puertos inseguros en grupos de seguridad de red en Azure
  hosts: localhost
  connection: local
  tasks:
    - name: Obtener detalles del NSG en Azure
      azure.azcollection.azure_rm_securitygroup_info:
        resource_group: "mi_grupo_recursos"
        name: "mi_nsg"
      register: azure_nsg

    - name: Eliminar reglas para puertos inseguros (FTP, Telnet, SMB)
      azure.azcollection.azure_rm_securitygroup:
        resource_group: "mi_grupo_recursos"
        name: "mi_nsg"
        security_rules:
          "{{ item }}"
      with_items: "{{ azure_nsg.network_security_groups[0].security_rules | selectattr('destination_port_range', 'in', ['23', '21', '445']) | list | map(attribute='name') | list }}"

    - name: Mostrar el estado de las reglas del NSG después de remover los puertos inseguros
      ansible.builtin.debug:
        var: azure_nsg.network_security_groups[0].security_rules

```

Explicación del Playbook:

1. Obtener detalles del NSG: Utiliza el módulo azure_rm_securitygroup_info para obtener la información del grupo de seguridad en Azure y almacenarla en la variable azure_nsg.
2. Eliminar reglas inseguras: Usa azure_rm_securitygroup para eliminar las reglas que permiten el acceso a puertos inseguros como 21 (FTP), 23 (Telnet) y 445 (SMB).
3. Mostrar el estado del NSG: Después de modificar el NSG, muestra las reglas restantes para confirmar que los puertos inseguros fueron eliminados.

### 2. Playbook para Remover Puertos Inseguros en AWS
En AWS, los grupos de seguridad (Security Groups) controlan el tráfico de entrada y salida. Usaremos el módulo ec2_security_group para modificar las reglas y bloquear puertos inseguros.
```
---
- name: Remover puertos inseguros en grupos de seguridad de AWS
  hosts: localhost
  connection: local
  tasks:
    - name: Obtener detalles del grupo de seguridad en AWS
      amazon.aws.ec2_group_info:
        filters:
          group-name: "mi_grupo_seguridad"
      register: aws_sg

    - name: Eliminar reglas inseguras (FTP, Telnet, SMB)
      amazon.aws.ec2_security_group:
        name: "mi_grupo_seguridad"
        region: "us-west-2"
        purge_ingress: no
        purge_egress: no
        rules:
          - proto: tcp
            ports: [23, 21, 445]
            cidr_ip: "0.0.0.0/0"
            rule_desc: "Remover puertos inseguros"

    - name: Mostrar las reglas del grupo de seguridad después de remover los puertos inseguros
      ansible.builtin.debug:
        var: aws_sg.security_groups[0].ip_permissions

```
Explicación del Playbook:

1. Obtener detalles del grupo de seguridad: Usa ec2_group_info para obtener la configuración actual del grupo de seguridad en AWS.
2. Eliminar reglas inseguras: Con ec2_security_group, elimina las reglas que permiten el acceso a los puertos 21 (FTP), 23 (Telnet), y 445 (SMB). El parámetro purge_ingress: no asegura que solo se modifiquen las reglas de los puertos inseguros y no todas las reglas de tráfico entrante.
3. Mostrar las reglas de seguridad: Después de modificar las reglas, el playbook muestra el estado actual del grupo de seguridad para confirmar los cambios.



4. 
