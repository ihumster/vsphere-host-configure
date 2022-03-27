# Host Provisioning Ansible module

## Как использовать этот модуль для настройки нового хоста (хостов)
 1. Устанавливаем ESXi на хост (пока вручную)
 2. Настраиваем ему mgmt network (пока вручную)
 3. Добавляем хост в vCenter в Datacenter (пока вручную)
 4. Мувим хост в DSwitch (пока вручную)
 5. Настраиваем group_vars для описание датацентра (пример в файле datacenter_name.yml)
 6. В папке inventory создаем файл <cluster_name>.yml (пример в файле inventory/cluster_name.yml)
 7. В папке group_vars создаем файл (например vault.yml) с кредами для vCenter и шифруем его с помощью ansible-vault encrypt (пример в файле vault.yml)
 8. В корне папки создаем (или актуализируем) плейбук configure.yml для того или иного кластера, указав ссылки на все переменные и список ролей для настрйки хоста, например:

```
 - hosts: all
  connection: local
  gather_facts: false
  vars_files:
    - group_vars/vault.yml                      #<<< ваши креды
    - group_vars/datacenter_name.yml            #<<< настройки общие для всех хостов
  roles:
    - common                                    #<<< Вводит хост в ММ, удаляет Standard Switch, настраивает ntp и ssh
    - network                                   #<<< Создает vmkernels
    - config                                    #<<< Настраивает Advanced Settings
    - datastore                                 #<<< Добавляет все NFS датасторы указаные в group_vars
    - iscsi-vmk                                 #<<< Создает vmk для iSCSI
    - iscsi                                     #<<< Создает static targets
```

Запускаем плейбук:

```
ansible-playbook configure.yml -i inventory/<cluster_name.yml> --vault-password-file=~/.vaultpassword
```
или
```
ansible-playbook configure.yml -i inventory/<cluster_name.yml> --ask-vault-pass
```