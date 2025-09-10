---
title: Simple Discord bot with Discord.js
description: Discord bot written in JavaScript.
date: 2024-02-18T23:45:05+0900
tags:
  - portfolio
toc: false
showFullContent: false
---

![Bot feature; Emoji Enlarger 🔎](attachments/bot.gif)

- [GitHub repo](https://github.com/nyhryan/atai-bot-2023)

Since I got more familiar with asynchronous JavaScript, I tried making discord bot again with [Discord.js](https://discord.js.org/) I used `git` to maintain this project, `TypeScript` and [`Bun`](https://bun.sh/) as main language/runtime, which all I learned from `Open Source SW, Web Programming` course, 2023 2nd semester.
- Actually I used `node.js` in `Web Programming` course. It was pretty easy to move to Bun runtime since it is almost compatible with `node.js`.
- The bot is hosted on Google Cloud Compute Engine's free tier VM.
    - Bot process is running with `PM2`.

It was quite challenging experience since the official [Discord.js guide](https://discordjs.guide/) only covers basics. I had to go through all over the various API documentations.

Initially I started this project with JavaScript, however I did not like loosely typed JavaScript. That is why this project is based on TypesSript.

## ⭐&#xFE0F; Features

- Sends back enlarged emoji as embedded image when user sends an emoji to the chat.
    - Saves enlarged emoji image as cache(File IO), this saves image conversion time when the user sends the same emoji later again.
- Check ping in miliseconds.
- Change bot's user status with command.
- Check server/user info with command.
