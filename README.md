# Онлайн-тетрадь продаж для аптеки (Django)

Простое, быстрое и надежное веб-приложение для аптеки, полностью заменяющее бумажную тетрадь продаж.

---

## Основные особенности
- **Mobile-first**: Разработано специально под экраны смартфонов (360–430 px). Крупные кнопки и поля (высота от 48 px), числовая клавиатура открывается автоматически (`inputmode="numeric"`), без горизонтального скролла.
- **Дизайн в аптечных тонах**: Фирменный зелёный `#1F7A4D`, мягкий фон `#F3F8F4`, белые карточки со скруглением 16 px, шрифт Inter.
- **Валюта — тенге (₸)**: Числа с удобным разделением разрядов (`184 000 ₸`, `+1 200 ₸`).
- **Никаких лишних полей**: Нет названий лекарств, нет скидок, нет складов. Записывается только цена продажи, закупочная цена и способ оплаты.
- **Быстрый ввод за 3 секунды**: После нажатия «Записать» форма мгновенно очищается, а курсор возвращается в поле цены. Итоги и список обновляются без перезагрузки страницы (Fetch API).
- **Периоды**: Мгновенное переключение «Сегодня» (с 00:00), «Неделя» (с понедельника), «Месяц» (с 1 числа). Часовой пояс строго `Asia/Almaty`.
- **Контроль ошибок**: Возможность отредактировать цену или удалить ошибочную запись (с диалогом подтверждения).
- **Безопасность**: Доступ только по логину и паролю. Посторонние не увидят выручку аптеки.

---

## Структура проекта

```text
├── manage.py                          # Главный скрипт управления Django
├── requirements.txt                   # Зависимости проекта
├── db.sqlite3                         # База данных SQLite (создаётся автоматически)
├── config/                            # Настройки проекта
│   ├── settings.py                    # Конфигурация, часовой пояс Asia/Almaty
│   ├── urls.py                        # Главный роутинг (вход, выход, приложение)
│   ├── wsgi.py
│   └── asgi.py
├── sales/                             # Приложение продаж
│   ├── models.py                      # Модель Sale (sale_price, cost_price, payment_method, created_at)
│   ├── forms.py                       # Форма с валидацией (> 0, >= 0)
│   ├── views.py                       # Главный экран и Fetch API
│   ├── urls.py                        # Эндпоинты продаж
│   ├── utils.py                       # Расчёт периодов (Almaty), агрегаты, форматирование тенге
│   ├── admin.py                       # Панель администратора
│   ├── tests.py                       # Набор автоматических тестов
│   ├── templatetags/
│   │   └── sales_tags.py              # Шаблонные фильтры (|tenge, |profit_display, |sale_time)
│   └── management/commands/
│       └── populate_demo.py           # Команда генерации демо-данных
├── templates/                         # HTML-шаблоны
│   ├── base.html                      # Базовый мобильный каркас
│   ├── sales/
│   │   └── index.html                 # Главный экран (шапка, периоды, карточка, форма, список)
│   └── registration/
│       └── login.html                 # Аккуратная страница входа
└── static/                            # Статические файлы
    ├── css/
    │   └── style.css                  # Чистый CSS в зелёных тонах (#1F7A4D)
    └── js/
        └── app.js                     # Асинхронное добавление, правка, удаление и фильтры
```

---

## Быстрый запуск локально

### 1. Перейдите в папку проекта и создайте виртуальное окружение:
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 2. Установите зависимости:
```bash
pip install -r requirements.txt
```

### 3. Примените миграции:
```bash
python manage.py migrate
```

### 4. Создайте пользователя для входа:
```bash
python manage.py createsuperuser
```
*(Или запустите команду `python manage.py populate_demo`, которая сразу создаст пользователя `demo` с паролем `demo123` и несколько тестовых продаж).*

### 5. Запустите сервер:
```bash
python manage.py runserver 0.0.0.0:8000
```
- Откройте на компьютере: **http://127.0.0.1:8000**
- Откройте с телефона в одной Wi-Fi сети: **http://[IP-компьютера]:8000** (например: `http://192.168.1.15:8000`).

---

## Развёртывание (выкладка в интернет)

### Вариант 1: PythonAnywhere (Самый простой для Django + SQLite)
1. Зарегистрируйтесь на [PythonAnywhere.com](https://www.pythonanywhere.com/).
2. Перейдите в **Consoles** -> **Bash**, склонируйте код или загрузите архив в папку проекта.
3. Создайте виртуальное окружение:
   ```bash
   mkvirtualenv --python=/usr/bin/python3.10 pharmacy-env
   pip install -r requirements.txt
   python manage.py migrate
   python manage.py createsuperuser
   python manage.py collectstatic
   ```
4. Во вкладке **Web**:
   - Нажмите **Add a new web app** (Manual configuration, Python 3.10).
   - В поле **Virtualenv** укажите путь к `pharmacy-env`.
   - В файле конфигурации **WSGI** укажите путь к папке проекта и `application = get_wsgi_application()`.
   - В разделе **Static Files** укажите:
     - URL: `/static/` -> Directory: путь к папке `staticfiles/`.
5. Нажмите **Reload** — сайт доступен в интернете 24/7.

### Вариант 2: VPS (Ubuntu + Nginx + Gunicorn)
1. Установите пакеты:
   ```bash
   sudo apt update && sudo apt install -y python3-pip python3-venv nginx
   ```
2. Склонируйте проект в `/var/www/pharmacy`:
   ```bash
   cd /var/www/pharmacy
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   python manage.py migrate
   python manage.py collectstatic --noinput
   python manage.py createsuperuser
   ```
3. Создайте systemd-сервис для Gunicorn (`/etc/systemd/system/pharmacy.service`):
   ```ini
   [Unit]
   Description=Pharmacy Sales Gunicorn daemon
   After=network.target

   [Service]
   User=www-data
   Group=www-data
   WorkingDirectory=/var/www/pharmacy
   ExecStart=/var/www/pharmacy/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 config.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```
4. Настройте Nginx (`/etc/nginx/sites-available/pharmacy`) с проксированием на `127.0.0.1:8000` и отдачей `/static/` из `/var/www/pharmacy/staticfiles/`.
5. Получите бесплатный SSL-сертификат HTTPS через Let's Encrypt:
   ```bash
   sudo certbot --nginx -d your-pharmacy-domain.kz
   ```

