# Проектная работа
Для поднятия данной проектной работы в виде небольшой инфраструктуры требуется установить (для Ubuntu 22.04):
1. Установить и настроить сеть на Virtualbox 7.0:
```
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --dearmor --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg
sudo apt-get update
sudo apt-get install virtualbox-7.0
sudo vi /etc/vbox/networks.conf
* 192.168.50.0/24
```
2. Установить vagrant:
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```
3. Установить git и openssh-server:
```
sudo apt update && sudo apt install git openssh-server
```
Копируем разворачиваемую проекную работу с моего репозитория git:
```
cd ~
mkdir Otus-homeworks
cd Otus-homeworks
git clone https://github.com/altyn-kenzhebaev/final-project.git
```
4. Подготавливаем наш hosted_machine и генерируем ssh ключи для проекта:
```
ssh-keygen -f /home/altynbek/Otus-homeworks/final-project/ansible/roles/cluster-setup/files/private_key
mkdir ~/.ssh
mv /home/altynbek/Otus-homeworks/final-project/ansible/roles/cluster-setup/files/private_key.pub ~/.ssh/authorized_keys
vi /etc/sudoers.d/local
altynbek     ALL=(ALL)     NOPASSWD: ALL
ssh-keygen -f /home/altynbek/Otus-homeworks/final-project/ansible/roles/backup-client/files/private_key
mkdir /home/altynbek/Otus-homeworks/final-project/ansible/roles/backup-setup/files
mv /home/altynbek/Otus-homeworks/final-project/ansible/roles/backup-client/files/private_key.pub /home/altynbek/Otus-homeworks/final-project/ansible/roles/backup-setup/files/authorized_keys
sed -i '1s/./command\=\"borg serve \-\-restrict\-to\-path \/var\/backup"\,restrict\ &/' /home/altynbek/Otus-homeworks/final-project/ansible/roles/backup-setup/files/authorized_keys
```
5. В целях легкого развертывания ВМ проектной работы предварительно скачиваем весомые пакеты ПО:
```
mkdir /home/altynbek/Otus-homeworks/final-project/ansible/roles/grafana-setup/files/
wget -O /home/altynbek/Otus-homeworks/final-project/ansible/roles/grafana-setup/files/grafana-enterprise-10.1.2-1.x86_64.rpm https://dl.grafana.com/enterprise/release/grafana-enterprise-10.1.2-1.x86_64.rpm
wget -O /home/altynbek/Otus-homeworks/final-project/ansible/roles/syslog-setup/files/elasticsearch-8.10.1-x86_64.rpm https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.10.1-x86_64.rpm
wget -O /home/altynbek/Otus-homeworks/final-project/ansible/roles/syslog-setup/files/logstash-8.10.1-x86_64.rpm https://artifacts.elastic.co/downloads/logstash/logstash-8.10.1-x86_64.rpm
```
6. Устанавливаем ansible и коллекцию prometheus:
```
sudo apt update && sudo apt install pip
pip install ansible
export PATH=$PATH:/home/altynbek/.local/bin
vi /home/altynbek/.ansible/collections/ansible_collections/prometheus/prometheus/meta/runtime.yml
requires_ansible: "~=2.15.0"
```
7. Разворачиваем телеграм-бот и копируем его bot_token и chat_id, шифруем bot_token, используя ansible-vault и вставляем данные в /home/altynbek/Otus-homeworks/final-project/group_vars/all.yml
Инструкции по разворачиванию alermanager и телеграм бота:
https://velenux.wordpress.com/2022/09/12/how-to-configure-prometheus-alertmanager-to-send-alerts-to-telegram/
```
echo 'strong_pass' > /home/altynbek/Otus-homeworks/final-project/.vault_pass.txt
chmod 0400 /home/altynbek/Otus-homeworks/final-project/.vault_pass.txt
ansible-vault encrypt_string --vault-password-file /home/altynbek/Otus-homeworks/final-project/.vault_pass.txt 'NUMBERS:GENERATED_API_KEY' --name 'bot_token'
```
В текущей директории final-project есть несколько папок и файлов, Ознакомимся с содержимым:
```
ls -l
README.md
ansible
Vagrantfile
```
Здесь:
- ansible - папка с плэйбуками
- README.md - файл с данным руководством
- Vagrantfile - файл описывающий виртуальную инфраструктуру для `Vagrant`
Запускаем развертывание проектной работы командой:
```
# vagrant up
```
Vagrant создает ВМ следующей структурой:
```
# vagrant status
Current machine states:

nginx                     running (virtualbox)
mysql                     running (virtualbox)
pcmk1                     running (virtualbox)
pcmk2                     running (virtualbox)
syslog                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
Инфрастуктура создана выполяем провижининг ВМ командой:
```
cd ansible
ansible-playbook final-project.yml --vault-password-file /home/altynbek/Otus-homeworks/final-project/.vault_pass.txt
```
#Proverka
curl -XGET -u elastic:o-MSyhsim47gLFvT1Qmx 'https://10.10.0.200:9200/_all/_search?q=*&pretty' -k 