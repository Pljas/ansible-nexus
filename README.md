# Ansible Nexus Node Management

Этот проект позволяет автоматизировать управление Nexus нодами через Ansible.

## Требования

### 1. Установка Python

**Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install python3 python3-pip
```

**macOS:**

```bash
# Через Homebrew
brew install python3
```

### 2. Установка Ansible

После установки Python установите Ansible:

```bash
pip3 install ansible
```

Подробная документация по установке: <https://docs.ansible.com/ansible/latest/installation_guide/index.html>

## Настройка SSH ключей

### Создание SSH ключа ed25519

1. Создайте новый SSH ключ:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. При запросе пути к файлу нажмите Enter (будет использован путь по умолчанию `~/.ssh/id_ed25519`)

3. Введите пароль для ключа (рекомендуется) или оставьте пустым

### Добавление ключа на сервер

**Способ 1: Через ssh-copy-id (Linux/macOS)**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server_ip
```

**Способ 2: Ручное копирование**

```bash
# Скопируйте содержимое публичного ключа
cat ~/.ssh/id_ed25519.pub

# Подключитесь к серверу и добавьте ключ
ssh user@server_ip
mkdir -p ~/.ssh
echo "ВСТАВЬТЕ_СОДЕРЖИМОЕ_ПУБЛИЧНОГО_КЛЮЧА" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

## Настройка проекта

### 1. Редактирование inventory файла

Откройте файл `inventory` и замените параметры:

```ini
[nexus_nodes]
# Замените на ваши данные:
192.168.1.100 ansible_user=root node_id="ваш_node_id" ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Параметры:

- `192.168.1.100` - IP адрес вашего сервера
- `ansible_user=root` - пользователь для подключения (может быть `ubuntu`, `admin` и т.д.)
- `node_id="ваш_node_id"` - ID вашей Nexus ноды
- `ansible_ssh_private_key_file` - путь к приватному SSH ключу

### 2. Исправление прав доступа к директории

Если вы получаете предупреждение о небезопасных правах доступа:

```bash
[WARNING]: Ansible is being run in a world writable directory
```

Исправьте права доступа:

**Linux/macOS/WSL:**

```bash
chmod o-w ~/ansible-nexus
# или для всего проекта:
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
```

## Запуск плейбука

### Доступные теги

Плейбук содержит следующие теги:

- `install` - Установка Docker и зависимостей
- `start` - Запуск Nexus контейнера
- `update` - Обновление и перезапуск контейнера
- `stop` - Остановка и удаление контейнера
- `attach` - Показать команду для подключения к контейнеру

### Команды запуска

**Полная установка и запуск:**

```bash
ansible-playbook playbook.yml --tags "install,start"
```

**Только установка зависимостей:**

```bash
ansible-playbook playbook.yml --tags "install"
```

**Только запуск контейнера:**

```bash
ansible-playbook playbook.yml --tags "start"
```

**Обновление контейнера:**

```bash
ansible-playbook playbook.yml --tags "update"
```

**Остановка контейнера:**

```bash
ansible-playbook playbook.yml --tags "stop"
```

**Показать команду для подключения:**

```bash
ansible-playbook playbook.yml --tags "attach"
```

### Запуск без SSH ключей

Если SSH ключи не настроены, используйте флаг `--ask-pass`:

```bash
ansible-playbook playbook.yml --tags "install,start" --ask-pass
```

Вам будет предложено ввести пароль для подключения к серверу.

### Запуск с sudo паролем

Если пользователь требует sudo пароль:

```bash
ansible-playbook playbook.yml --tags "install,start" --ask-become-pass
```

### Комбинированные флаги

```bash
# Если нужны и SSH пароль, и sudo пароль
ansible-playbook playbook.yml --tags "install,start" --ask-pass --ask-become-pass
```

## Подключение к контейнеру

После запуска контейнера вы можете подключиться к его интерфейсу:

```bash
ssh user@server_ip -t 'docker attach nexus'
```

**Важно:** Для отключения от контейнера без его остановки используйте комбинацию клавиш `Ctrl+P`, затем `Ctrl+Q`.

## Устранение проблем

### Проблема с правами доступа

```
[WARNING]: Ansible is being run in a world writable directory
```

**Решение:**

```bash
chmod o-w /path/to/ansible-nexus
```

### Проблема с inventory

```
[WARNING]: provided hosts list is empty, only localhost is available
```

**Решение:**

1. Проверьте права доступа к директории
2. Убедитесь, что файл `ansible.cfg` существует
3. Проверьте синтаксис файла `inventory`

### Проблема с подключением

```
FATAL: UNREACHABLE
```

**Решение:**

1. Проверьте IP адрес сервера
2. Убедитесь, что SSH ключи настроены правильно
3. Проверьте, что сервер доступен: `ping server_ip`
4. Попробуйте подключиться вручную: `ssh user@server_ip`

### Проблема с Docker модулями

```
ERROR! couldn't resolve module/action 'community.docker.docker_container'
```

**Решение:**

```bash
ansible-galaxy collection install community.docker
```

## Структура проекта

```
ansible-nexus/
├── ansible.cfg          # Конфигурация Ansible
├── inventory            # Список серверов
├── playbook.yml         # Основной плейбук
└── README.md           # Эта документация
```

## Дополнительная информация

- [Документация Ansible](https://docs.ansible.com/)
- [Документация Docker](https://docs.docker.com/)
- [Nexus CLI GitHub](https://github.com/nexusxyz/nexus-cli)

---

**Примечание:** Этот проект предназначен для управления Nexus нодами. Убедитесь, что у вас есть действительный `node_id` перед запуском.
