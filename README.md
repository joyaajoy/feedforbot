FeedForBot
==========

[![PyPI](https://img.shields.io/pypi/v/feedforbot.svg)](https://pypi.python.org/pypi/feedforbot)
[![PyPI](https://img.shields.io/pypi/dm/feedforbot.svg)](https://pypi.python.org/pypi/feedforbot)
[![Docker Pulls](https://img.shields.io/docker/pulls/shpaker/feedforbot)](https://hub.docker.com/r/shpaker/feedforbot)
[![PyPI](https://img.shields.io/badge/code%20style-black-000000.svg)](href="https://github.com/psf/black)

Forward links from RSS/Atom feeds to messengers

Installation
------------

```commandline
pip install feedforbot -U
```

Usage
-----

### From code

```python
import asyncio

from feedforbot import Scheduler, TelegramBotTransport, RSSListener


def main():
  loop = asyncio.new_event_loop()
  asyncio.set_event_loop(loop)
  scheduler = Scheduler(
    '* * * * *',
    listener=RSSListener('https://www.debian.org/News/news'),
    transport=TelegramBotTransport(
      token='123456789:AAAAAAAAAA-BBBB-CCCCCCCCCCCC-DDDDDD',
      to='@channel',
    )
  )
  scheduler.run()
  loop.run_forever()

if __name__ == '__main__':
  main()
```

### CLI

#### Save to file `config.yml` data

```yaml  
---
cache:
  type: 'files'
schedulers:
  - listener:
      type: 'rss'
      params:
        url: 'https://habr.com/ru/rss/all/all/?fl=ru'
    transport:
      type: 'telegram_bot'
      params:
        token: '123456789:AAAAAAAAAA-BBBB-CCCCCCCCCCCC-DDDDDD'
        to: '@tmfeed'
        template: |-
          <b>{{ TITLE }}</b> #habr
          {{ ID }}
          <b>Tags</b>: {% for category in CATEGORIES %}{{ category }}{{ ", " if not loop.last else "" }}{% endfor %}
          <b>Author</b>: <a href="https://habr.com/users/{{ AUTHORS[0] }}">{{ AUTHORS[0] }}</a>
  - listener:
      type: 'rss'
      params:
        url: 'https://habr.com/ru/rss/news/?fl=ru'
    transport:
      type: 'telegram_bot'
      params:
        token: '123456789:AAAAAAAAAA-BBBB-CCCCCCCCCCCC-DDDDDD'
        to: '@tmfeed'
        template: |-
          <b>{{ TITLE }}</b> #habr
          {{ ID }}
          <b>Tags</b>: {% for category in CATEGORIES %}{{ category }}{{ ", " if not loop.last else "" }}{% endfor %}
  - listener:
      type: 'rss'
      params:
        url: 'http://www.opennet.ru/opennews/opennews_all.rss'
    transport:
      type: 'telegram_bot'
      params:
        token: '123456789:AAAAAAAAAA-BBBB-CCCCCCCCCCCC-DDDDDD'
        to: '@tmfeed'
        disable_web_page_preview: yes
        template: |-
          <b>{{ TITLE }}</b> #opennet
          {{ URL }}

          {{ TEXT }}
```

#### Start script

```commandline
feedforbot --verbose config.yml
```

### Docker 

#### Docker Hub

```commandline
docker run shpaker/feedforbot --help
```

#### GHCR

```commandline
docker run ghcr.io/shpaker/feedforbot --help
```

VPS Installation Guide
----------------------

This section provides a step-by-step guide to deploy **FeedForBot** on a Linux VPS (Ubuntu/Debian).

### Prerequisites

- A VPS running Ubuntu 20.04+ or Debian 11+
- Python 3.10 or newer
- `pip` package manager

### Option 1: Install with pip and run as a systemd service

**Step 1 – Update your system and install Python**

```commandline
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv
```

**Step 2 – Create a dedicated user (recommended)**

```commandline
sudo useradd --system --create-home feedforbot
sudo su - feedforbot
```

**Step 3 – Install FeedForBot**

```commandline
pip install feedforbot -U
```

**Step 4 – Create a configuration file**

Create `/home/feedforbot/config.yml` with your feed and transport settings:

```yaml
---
cache:
  type: 'files'
schedulers:
  - listener:
      type: 'rss'
      params:
        url: 'https://example.com/rss'
    transport:
      type: 'telegram_bot'
      params:
        token: '123456789:AAAAAAAAAA-BBBB-CCCCCCCCCCCC-DDDDDD'
        to: '@yourchannel'
```

**Step 5 – Test the bot manually**

```commandline
feedforbot --verbose /home/feedforbot/config.yml
```

**Step 6 – Create a systemd service to run it automatically**

Exit back to your sudo user, then create the service file:

```commandline
exit
sudo nano /etc/systemd/system/feedforbot.service
```

Paste the following content:

```ini
[Unit]
Description=FeedForBot RSS to Messenger Bot
After=network.target

[Service]
Type=simple
User=feedforbot
ExecStart=/home/feedforbot/.local/bin/feedforbot /home/feedforbot/config.yml
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Step 7 – Enable and start the service**

```commandline
sudo systemctl daemon-reload
sudo systemctl enable feedforbot
sudo systemctl start feedforbot
sudo systemctl status feedforbot
```

---

### Option 2: Run with Docker

**Step 1 – Install Docker**

```commandline
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable --now docker
```

**Step 2 – Create a configuration file**

Create `config.yml` in a convenient directory, e.g. `/opt/feedforbot/config.yml`, with your feed and transport settings (see the CLI section above for an example).

**Step 3 – Run the container**

Using Docker Hub:

```commandline
docker run -d \
  --name feedforbot \
  --restart unless-stopped \
  -v /opt/feedforbot/config.yml:/config.yml \
  shpaker/feedforbot /config.yml
```

Or using GHCR:

```commandline
docker run -d \
  --name feedforbot \
  --restart unless-stopped \
  -v /opt/feedforbot/config.yml:/config.yml \
  ghcr.io/shpaker/feedforbot /config.yml
```

**Step 4 – Check logs**

```commandline
docker logs -f feedforbot
```
