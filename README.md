# Лабораторная работа №4
# Amazon S3 - Облачное хранилище данных

## 1. Цель работы

Целью лабораторной работы является изучение сервиса Amazon S3 (Simple Storage Service) и освоение основных операций объектного хранения данных:

- создание публичного и приватного бакетов
- загрузка объектов через AWS Management Console и AWS CLI
- управление доступом (ACL и Block Public Access)
- включение версионирования объектов
- настройка Lifecycle-политик
- развертывание статического веб-сайта на базе S3

## 2. Теоретические сведения

Amazon S3 является объектным хранилищем данных, предназначенным для хранения файлов любого типа: изображений, логов, резервных копий, статического контента и других данных.

Каждый объект в S3:

- хранится внутри бакета (bucket)
- имеет уникальный ключ (object key)
- может содержать метаданные

В S3 отсутствует традиционная файловая иерархия.  
Папки в интерфейсе представляют собой префиксы в имени ключа объекта.

## 3. Практическая часть

Регион выполнения работы: **eu-central-1 (Frankfurt)**.

### 3.1 Подготовка локальной структуры

Локально была создана следующая структура каталогов:

```
s3-lab/
├── public/
│   ├── avatars/
│   │   ├── user1.jpg
│   │   └── user2.jpg
│   └── content/
│       └── logo.png
├── private/
│   └── logs/
│       └── activity.csv
└── README.md
```

<img width="513" height="360" alt="image" src="https://github.com/user-attachments/assets/6b1429d3-b393-4f47-87ea-ef0a41ea2021" />

### 3.2 Создание бакетов

Были созданы два бакета:

- **cc-lab4-pub-k008** - публичный  
- **cc-lab4-priv-k008** - приватный  

<img width="594" height="159" alt="image" src="https://github.com/user-attachments/assets/d27bbb45-54fd-42da-aebf-6a62320c1117" />

#### Настройки публичного бакета:

- Region: eu-central-1  
- Object Ownership: ACLs enabled  
- Block all public access: отключен  

Параметр Block all public access является глобальным механизмом защиты, который запрещает любой публичный доступ к бакету и объектам, даже если ACL или политики разрешают его.

Приватный бакет был создан с настройками по умолчанию, при которых публичный доступ полностью запрещен.

### 3.3 Загрузка объектов через AWS Management Console

В публичный бакет был загружен объект:

- `avatars/user1.jpg`

При загрузке был выбран параметр Grant public-read access, что позволило сделать объект публично доступным.

<img width="864" height="590" alt="image" src="https://github.com/user-attachments/assets/e2162009-b74e-4e79-a595-ee4fc64b3a76" />

Имя файла: `user1.jpg`  
Object key: `avatars/user1.jpg`

Object key представляет собой полный путь к объекту внутри бакета.

### 3.4 Загрузка объектов через AWS CLI

Были выполнены следующие команды:

```bash
aws s3 cp s3-lab/public/avatars/user2.jpg s3://cc-lab4-pub-k008/avatars/user2.jpg --acl public-read

aws s3 cp s3-lab/public/content/logo.png s3://cc-lab4-pub-k008/content/logo.png --acl public-read

aws s3 cp s3-lab/private/logs/activity.csv s3://cc-lab4-priv-k008/logs/activity.csv
```

<img width="680" height="711" alt="image" src="https://github.com/user-attachments/assets/8e64cdf2-82d7-4e76-87de-d30760f20059" />

#### Разница между командами AWS CLI:

- `cp` - копирование файлов
- `mv` - перемещение (копирование с последующим удалением исходного файла)
- `sync` - синхронизация каталогов

Параметр `--acl public-read` делает объект публично доступным.

### 3.5 Проверка доступа к объектам

Публичный объект:

```
https://cc-lab4-pub-k008.s3.eu-central-1.amazonaws.com/avatars/user1.jpg
```

<img width="1278" height="1246" alt="Снимок экрана 2025-12-08 230116" src="https://github.com/user-attachments/assets/acd26328-fc01-4c19-806b-33c1dea9e364" />

Объект успешно открывается в браузере.

Приватный объект:

```
https://cc-lab4-priv-k008.s3.eu-central-1.amazonaws.com/logs/activity.csv
```

<img width="1599" height="369" alt="Снимок экрана 2025-12-08 230314" src="https://github.com/user-attachments/assets/bf60b8e4-a135-40a7-a7a4-9be740091cca" />

При попытке доступа отображается сообщение **AccessDenied**, что подтверждает корректную настройку приватного бакета.

### 3.6 Версионирование объектов

В обоих бакетах было включено версионирование:

`Properties -> Bucket Versioning -> Enable`

После повторной загрузки измененного файла `logo.png` в консоли отобразились две версии объекта.

В случае отключения версионирования бакет переводится в статус **Suspended**.  
Ранее созданные версии сохраняются, однако новые версии объектов создаваться не будут.

### 3.7 Lifecycle-политика

В приватном бакете была создана Lifecycle-политика со следующими параметрами:

- Prefix: `logs/`
- Через 30 дней -> переход в STANDARD-IA
- Через 365 дней -> переход в Glacier Deep Archive
- Через 1825 дней -> удаление объекта

<img width="467" height="981" alt="image" src="https://github.com/user-attachments/assets/656b11c0-df15-496a-8d96-05dd29a0e436" />

#### Storage Class

Storage Class - это класс хранения данных в Amazon S3, определяющий стоимость хранения и частоту доступа к объектам.

Основные классы хранения:

- STANDARD - для часто используемых данных
- STANDARD-IA - для редко используемых
- GLACIER - архивное хранение
- DEEP ARCHIVE - долгосрочный архив

Использование Lifecycle-политик позволяет автоматически оптимизировать стоимость хранения логов.

### 3.8 Развертывание статического веб-сайта

Был создан третий бакет:

- **cc-lab4-web-k008**
- ACLs enabled
- Block Public Access отключен

В настройках включен режим:

`Properties -> Static website hosting -> Enable`  
Index document: `index.html`

В бакет были загружены файлы:

- index.html
- styles.css
- index.js

<img width="733" height="786" alt="image" src="https://github.com/user-attachments/assets/99c0a4da-d1be-40f7-a551-ea61a235c4a4" />

Все объекты получили доступ `public-read`.

Сайт доступен по адресу:

```
http://cc-lab4-web-k008.s3-website.eu-central-1.amazonaws.com
```

<img width="1233" height="1255" alt="image" src="https://github.com/user-attachments/assets/922fecd2-0dfe-44c4-892e-c2157ba2af5c" />

## 4. Вывод

В ходе выполнения лабораторной работы были изучены основные возможности сервиса Amazon S3:

- создание публичных и приватных бакетов
- управление доступом через ACL и Block Public Access
- загрузка объектов через веб-интерфейс и AWS CLI
- включение и анализ работы версионирования
- настройка Lifecycle-политик для автоматического архивирования
- развертывание статического веб-сайта

Полученные знания позволяют применять Amazon S3 как для хранения данных, так и для размещения статического веб-контента.
