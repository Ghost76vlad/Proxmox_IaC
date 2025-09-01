# Proxmox_IaC
=========================
# Заметки по Home Lab  #
=========================

Все чувствительные данные забираются с сервера Vault
Подключение к серверу Vault происходит через Токен который хранится в окружении ОС 


# Команды для работы проектом

# Основные переменные
# vm_name - Имя создаваемой VM
# vm_memory -Количество ОЗУ
# vm_core - Количество CPU
# vm_networks - IP и GW VM. Эти данные прописываются в CloudInit Proxmox. Оформляется в следующем виде: "IP/Сокращенная_маска,gw=IP_GW"
# disk_type - Тип диска  (unused[n], ide[n], sata[n], scsi[n] or virtio[n])
# vm_users - Имя пользователя в ОС. Создается через CloudInit. Поумолчанию создается пользователь ansible с паролем взятого из Vault-сервера
# vm_passwd - Пароль vm_users


=========================
# Создание VM
=========================

# Создание одной VM
ansible-playbook create_vm_proxmox.yml \
  --extra-vars '{"vm_name": ["test01"], "vm_networks": ["10.10.10.80/24,gw=10.10.10.1"]}' \
  --tags create-vm

# Создание нескольких VM
ansible-playbook create_vm_proxmox.yml \
  -e '{"vm_name": ["test02", "test03"], "vm_networks": ["10.10.10.80/24,gw=10.10.10.1","10.10.10.81/24,gw=10.10.10.1"]}' \
  --tags create-vm

# Создание VM с отличной от default конфига VM
ansible-playbook create_vm_proxmox.yml \
  -e '{"vm_name": ["test02", "test03"], "vm_networks": ["10.10.10.80/24,gw=10.10.10.1","10.10.10.81/24,gw=10.10.10.1"]}' \
  -e "vm_core=4 vm_memory=4096" \
  --tags create-vm

# Примечание: 
- Playbook содержит таски не только по созданию VM, поэтому необходимо использовать --tags
- При запуске плейбука необходимо передовать две обязательные переменные vm_name и vm_networks
- Каждой VM необходимо указывать vm_networks


# Добавление Диска при создании VM

ansible-playbook copy_create_vm_proxmox.yml \
  -e '{"vm_name": ["test01"], "vm_networks": ["10.10.10.80/24,gw=10.10.10.1"]}' \
  -e '{"disk_type": ["scsi1", "scsi2", "scsi3"]}' \
  --tags create-vm,add_disk

ansible-playbook copy_create_vm_proxmox.yml \
  -e '{"vm_name": ["test01", "test02"], "vm_networks": ["10.10.10.80/24,gw=10.10.10.1","10.10.10.81/24,gw=10.10.10.1"]}' \
  -e '{"disk_type": ["scsi1"]}' \
  --tags create-vm,add_disk

# Примечание:
Может быть указано любое количество дисков
Для добавления дисков к VM, при создании указываем тэг add_disk

# Удаление VM

ansible-playbook remove_vm_proxmox.yml -e 'vm_id=["100","101","102","103","104"]'

# Чтобы не писать такие длинные команды, можно их убрать в yml-файл с переменными.
# Например:

Создадим vars_create_ceph.yml в корне проекта:

vm_core: 2
vm_memory: 4096
vm_name:
  - ceph01
  - ceph02
  - ceph03

vm_networks:
  - 10.10.10.23/24,gw=10.10.10.1
  - 10.10.10.24/24,gw=10.10.10.1
  - 10.10.10.25/24,gw=10.10.10.1

disk_type:
  - scsi1
  - scsi2
  - scsi3

Команда для запуска:

 ansible-playbook create_vm_proxmox.yml   -e @vars_create_ceph.yml
