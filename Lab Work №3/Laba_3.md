Лабораторная работа №3
---
___Тема:___ Использование принципов проектирования на уровне методов и классов
___Цель работы:___ Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.

Диаграмма контейнеров
---
![alt text](./CD.svg)

Диаграмма компонентов
---
![alt text](./компонент1.svg)

Диаграмма последовательностей
---
![alt text](./SequenceDiagram.svg)

Модель БД
---
![alt text](./BD.svg)
1. __Пользователи (Users):__  
Хранит информацию о зарегистрированных пользователях.

__Поля:__  
 - user_id (PK, INT) — Уникальный идентификатор пользователя.
 - username (VARCHAR) — Имя пользователя.
 - email (VARCHAR) — Электронная почта.
 - password_hash (VARCHAR) — Хэш пароля.
 - created_at (DATETIME) — Дата и время регистрации.

__Тип связи:__  
 "Один ко многим" (1:N)

__Описание:__  
Каждый пользователь может создать несколько серверов, где он будет владельцем. Один сервер обязательно принадлежит одному пользователю.

2. __Серверы (Servers):__  
Хранит информацию о созданных серверах.

__Поля:__  
 - server_id (PK, INT) — Уникальный идентификатор сервера.
 - name (VARCHAR) — Название сервера.
 - owner_id (FK, INT) — Идентификатор владельца сервера (ссылается на Users.user_id).
 - created_at (DATETIME) — Дата и время создания.
 - description (TEXT) — Описание сервера.

__Тип связи:__   
"Один ко многим" (1:N)

__Описание:__  
Один сервер может содержать несколько текстовых каналов, каждый из которых уникален для этого сервера.
Например, сервер "Общение" может включать текстовые каналы "Общий", "Мемы", "Игры".

3. __Текстовые каналы (Text_Channels):__  
Хранит информацию о текстовых каналах внутри серверов.

__Поля:__  
 - channel_id (PK, INT) — Уникальный идентификатор канала.
 - server_id (FK, INT) — Идентификатор сервера (ссылается на Servers.server_id).
 - name (VARCHAR) — Название текстового канала.
 - created_at (DATETIME) — Дата создания канала.

__Тип связи:__  
"Один ко многим" (1:N)

__Описание:__  
Каждый текстовый канал может содержать множество сообщений, отправленных разными пользователями.
Например, в канале "Общий" могут быть сообщения от нескольких пользователей.

4. __Сообщения (Messages):__  
Хранит текстовые сообщения, отправленные в каналах.

__Поля:__  
 - message_id (PK, INT) — Уникальный идентификатор сообщения.
 - channel_id (FK, INT) — Идентификатор текстового канала (ссылается на Text_Channels.channel_id).
 - user_id (FK, INT) — Идентификатор автора сообщения (ссылается на Users.user_id).
 - content (TEXT) — Содержание сообщения.
 - sent_at (DATETIME) — Время отправки сообщения.

__Тип связи:__  
"Один ко многим" (1:N)

__Описание:__  
Один пользователь может отправить множество сообщений, но каждое сообщение принадлежит только одному пользователю.
Например, пользователь "Иван" может отправить 5 сообщений в разных каналах.

5. __Участники сервера (Server_Members):__  
Хранит связь между пользователями и серверами, в которых они участвуют.

__Поля:__  
 - membership_id (PK, INT) — Уникальный идентификатор записи.
 - server_id (FK, INT) — Идентификатор сервера (ссылается на Servers.server_id).
 - user_id (FK, INT) — Идентификатор пользователя (ссылается на Users.user_id).
 - role (VARCHAR) — Роль пользователя на сервере (например, администратор, модератор, участник).
 - joined_at (DATETIME) — Дата и время присоединения к серверу.

__Тип связи:__  
"Один ко многим" (1:N)

__Описание:__  
Один пользователь может быть участником нескольких серверов. Например, пользователь "Аня" может быть участником серверов "Игры" и "Музыка".
Один сервер может содержать нескольких участников. Например, сервер "Игры" может включать "Иван", "Аня" и "Сергей".

Применение основных принципов разработки
---

__Принцип KISS (Keep It Simple, Stupid)__   
Пример: Простая функция для получения списка пользователей сервера.  

```
# Серверный код (FastAPI)
from fastapi import FastAPI, HTTPException

app = FastAPI()

# Простой хранилище данных
servers = {
    1: {"users": ["Alice", "Bob", "Charlie"]}
}

@app.get("/servers/{server_id}/users")
def get_users(server_id: int):
    if server_id not in servers:
        raise HTTPException(status_code=404, detail="Server not found")
    return servers[server_id]["users"]
```
Объяснение:  
Код решает только одну задачу (KISS).  
Нет лишней сложности или дополнительных проверок.  

__Принцип YAGNI (You Aren't Gonna Need It)__  
Пример: Создание текстового канала без предварительной реализации роли администратора, которая пока не нужна.  
```
@app.post("/servers/{server_id}/channels")
def create_channel(server_id: int, channel_name: str):
    if server_id not in servers:
        raise HTTPException(status_code=404, detail="Server not found")
    
    # Добавляем канал
    servers[server_id].setdefault("channels", []).append(channel_name)
    return {"message": f"Channel '{channel_name}' created"}
```
Объяснение:  
Реализована минимальная функциональность без учета пока ненужных ролей.  

__Принцип DRY (Don't Repeat Yourself)__  
Пример: Вынесение проверки существования сервера в отдельную функцию.  
```
def get_server_or_404(server_id: int):
    if server_id not in servers:
        raise HTTPException(status_code=404, detail="Server not found")
    return servers[server_id]

@app.get("/servers/{server_id}/users")
def get_users(server_id: int):
    server = get_server_or_404(server_id)
    return server["users"]

@app.post("/servers/{server_id}/channels")
def create_channel(server_id: int, channel_name: str):
    server = get_server_or_404(server_id)
    server.setdefault("channels", []).append(channel_name)
    return {"message": f"Channel '{channel_name}' created"}
```
Объяснение:  
Проверка существования сервера реализована единожды и переиспользуется.  

__Принципы SOLID__  
1. S - Single Responsibility Principle 
```
class UserManager:
    def get_users(self, server):
        return server["users"]

class ChannelManager:
    def add_channel(self, server, channel_name):
        server.setdefault("channels", []).append(channel_name)
```
Объяснение:  
UserManager отвечает только за пользователей, а ChannelManager только за каналы.  
2. O - Open/Closed Principle  
```
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send(self, message):
        pass

class EmailNotifier(Notifier):
    def send(self, message):
        print(f"Sending email: {message}")

class SMSNotifier(Notifier):
    def send(self, message):
        print(f"Sending SMS: {message}")
```
Объяснение: 
Теперь можно добавить TelegramNotifier, не меняя код Notifier  
3. L - Liskov Substitution Principle  
```
class BaseChannel(ABC):
    """Абстрактный базовый класс для каналов"""
    @abstractmethod
    def connect(self):
        pass

class TextChannel(BaseChannel):
    """Текстовый канал поддерживает отправку сообщений"""
    def connect(self):
        print("Connecting to text channel")

    def send_message(self, message):
        print(f"Message sent: {message}")

class VoiceChannel(BaseChannel):
    """Голосовой канал поддерживает голосовое соединение, но не текстовые сообщения"""
    def connect(self):
        print("Connecting to voice channel")
```  
Объяснение:  
TextChannel и VoiceChannel не ломают логику базового класса.  
4. I - Interface Segregation Principle  
```    
class TextMessaging(ABC):
    @abstractmethod
    def send_message(self, message):
        pass

class VoiceCommunication(ABC):
    @abstractmethod
    def start_voice_call(self):
        pass

class TextChannel(TextMessaging):
    def send_message(self, message):
        print(f"Message sent: {message}")

class VoiceChannel(VoiceCommunication):
    def start_voice_call(self):
        print("Voice call started")
```  
Объяснение:  
Классы реализуют только те методы, которые им нужны.  
5. D - Dependency Inversion Principle
```    
class NotificationService:
    def __init__(self, notifier: Notifier):
        self.notifier = notifier  # Теперь можно передавать любой тип уведомлений

    def notify(self, message):
        self.notifier.send(message)

email_service = NotificationService(EmailNotifier())
sms_service = NotificationService(SMSNotifier())
```  
Объяснение:  
Можно легко подменять Notifier, не изменяя NotificationService.  
