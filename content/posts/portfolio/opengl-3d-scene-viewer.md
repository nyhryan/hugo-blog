---
title: Simple OpenGL 3D Scene viewer
description: Studied basics of OpenGL.
date: 2024-02-18T23:59:15+09:00
tags:
  - portfolio
toc: false
showFullContent: false
---

![demo](attachments/demo.gif)

> GitHub: [nyhryan/OpenGL-model-viewer](https://github.com/nyhryan/OpenGL-model-viewer)

I felt like studying bit of computer graphics this winter(Jan 2024). I gave OpenGL a try with the help of this [great site (learnopengl.com)](https://learnopengl.com/)

Understanding how OpenGL system work(state machine) was pretty difficult at the beginning. Furthermore, newer version of OpenGL has this new system called Direct State Access, which made me more confused. However I managed to create this simple scene viewer.

## ⭐&#xFE0F; Features

- Fly around the scene with <kbd>WASD</kbd>, <kbd>Left Control</kbd> to descend, <kbd>Space</kbd> to ascend and Mouse.
- Adjust the light's position and color.
- View the depth map from the light's perspective.
- Edit and hot-reload the main shdaer script while program is running.
- Control some of parameters with [ImGUI](https://github.com/ocornut/imgui) library.