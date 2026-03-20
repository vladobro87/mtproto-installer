# MTProto Proxy (Fake TLS) + Traefik

Один порт **4443**: по SNI трафик к домену маскировки (например `1c.ru`) уходит в MTProxy, остальное можно отдавать другим сервисам через Traefik.

- **Telemt** — современный MTProxy (Rust, distroless), поддерживает Fake TLS.
- **Traefik** — маршрутизация TCP по SNI с TLS passthrough.

## Установка на сервере (всё тянется с GitHub)

```bash
curl -sSL https://raw.githubusercontent.com/vladobro87/mtproto-installer/main/install.sh | bash
```

Скрипт установит Docker (если нужно), скачает `docker-compose.yml`, конфиги Traefik и шаблон Telemt из репозитория [vladobro87/mtproto-installer](https://github.com/vladobro87/mtproto-installer), сгенерирует секрет, подставит домен маскировки и запустит контейнеры. В конце выведет ссылку вида `tg://proxy?server=...&port=4443&secret=...` — добавьте её в Telegram (Настройки → Данные и память → Использовать прокси).

- Домен маскировки по умолчанию: `1c.ru`. Интерактивно можно ввести другой; без TTY: `FAKE_DOMAIN=sberbank.ru curl -sSL ... | bash`.
- Каталог установки по умолчанию: `./mtproxy-data`. Другой: `INSTALL_DIR=/opt/mtproxy curl -sSL ... | bash`.

## Локальный запуск (клонирование репозитория)

После `git clone https://github.com/vladobro87/mtproto-installer.git && cd mtproto-installer` можно запустить `./install.sh` — скрипт по умолчанию качает файлы с того же GitHub. Либо настроить вручную и поднять без скрипта:

1. Сгенерируйте секрет: `openssl rand -hex 16`. Скопируйте `telemt.toml.example` в `telemt.toml`, подставьте секрет и при необходимости домен в `censorship.tls_domain`.
2. В `traefik/dynamic/tcp.yml` домен в `HostSNI(...)` должен совпадать с `censorship.tls_domain` в `telemt.toml`.
3. Запуск: `docker compose up -d`.
4. Ссылка: `tg://proxy?server=ВАШ_IP&port=4443&secret=ВАШ_СЕКРЕТ`.
