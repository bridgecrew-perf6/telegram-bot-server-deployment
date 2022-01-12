# telegram-bot-server-deployment

This repository will help you launch your Telegram Bot Server on a dedicated server using Docker Compose and set up a secure protocol connection `https`

If you are using this tutorial I will assume that you already have `Docker` and `Docker Compose` installed on your dedicated server.

## docker-compose.yml

```
version: '3.7'

services:
  telegram-bot-server:
    image: aiogram/telegram-bot-api:5.5
    container_name: bot-server
    environment:
      TELEGRAM_API_ID: "you-api-id-here"
      TELEGRAM_API_HASH: "you-api-hash-here"
      TELEGRAM_STAT: 1
    volumes:
      - telegram-bot-server-data:/var/lib/telegram-bot-api
    ports:
      - 8081:8081
      - 8082:8082
    restart: always

volumes:
  telegram-bot-server-data:

```
In principle, this could have ended, input 
 - `<host-ip>:8081/bot<bot-token>/getMe` - viewing basic information on the bot
 -  `<host-ip>:8082` - viewing information on the server
