# sing-box VLESS VPN (TUN)

Набор скриптов для быстрой установки и управления VLESS VPN на базе sing-box с TUN-режимом (весь системный трафик через VPN).

## Быстрый старт

```bash
git clone git@github.com:DenisMRH/vpn-vless.git
cd vpn-vless
sudo VLESS_URL='vless://...' ./install --install-sing-box --start
```

Что произойдёт:
- установит `sing-box` через пакетный менеджер системы
- скопирует скрипты в `/usr/local/bin/`
- сохранит ссылку в `/etc/sing-box/vless.url`
- создаст `sing-box.service` (если пакет не создал)
- применит конфиг и запустит VPN

## Содержимое

| Файл | Назначение |
|------|------------|
| `install` | Установщик: копирует скрипты, настраивает systemd, применяет конфиг |
| `sing-box-apply-vless` | Собирает `/etc/sing-box/config.json` из `vless://` ссылки |
| `vpn-tunnel` | Обёртка для управления VPN (on/off/status/apply) |

## Использование после установки

```bash
sudo vpn-tunnel on           # включить VPN
sudo vpn-tunnel off          # выключить VPN
sudo vpn-tunnel status       # состояние
sudo vpn-tunnel apply-start  # применить новый конфиг и включить
```

### Сменить VLESS-ссылку

```bash
sudo nano /etc/sing-box/vless.url
sudo vpn-tunnel apply-start
```

### Обход VPN для своих сетей и доменов

```bash
sudo nano /etc/sing-box/direct-extras.txt
```

Формат (одна запись на строку):
- `203.0.113.0/24` — CIDR
- `domain:example.com` — точное имя
- `suffix:.lan` — суффикс

```bash
sudo vpn-tunnel apply
sudo vpn-tunnel on
```

## Проверка

```bash
systemctl is-active sing-box        # должно быть: active
curl -sS https://api.ipify.org      # внешний IP (должен отличаться без VPN)
sudo sing-box check -C /etc/sing-box
sudo journalctl -u sing-box -e --no-pager -n 50
```

## Требования

- Linux с systemd
- `sing-box` (установится автоматически с `--install-sing-box` на Debian/Ubuntu/RHEL/Arch)
- Python 3.6+ (для `sing-box-apply-vless`)

## Ограничения

- Транспорт `xhttp` / `splithttp` не поддерживается — используйте ws, tcp, grpc, httpupgrade, quic.
- Автозапуск при загрузке отключён намеренно. Включить: `sudo systemctl enable sing-box`.
