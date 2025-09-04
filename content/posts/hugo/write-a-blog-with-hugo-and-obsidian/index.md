---
title: Write a blog with Hugo and Obsidian
description: Make your own blog with Hugo and Obsidian!
date: 2025-08-24T17:28:42+09:00
categories:
  - guide
tags:
  - hugo
---

> References:
> - [oscarmlage - Hugo and Obsidian](https://oscarmlage.com/posts/hugo-and-obsidian/)
> - [4rkal - My Obsidian + Hugo blogging setup](https://4rkal.com/posts/obsidian-hugo/)
> - [Hugomods - Docker](https://docker.hugomods.com/docs/introduction/)

## 0. Prerequisite

- Install [Docker Engine](https://docs.docker.com/engine/install/)
- Install [Obsidian](https://obsidian.md/)

## 1. Create Hugo site

```shell
docker run -v ${PWD}:src \
	hugomods/hugo:exts-non-root hugo new site <blog-dir-name>
```

To create Hugo site's folder, open up the terminal and run the following `docker run` command. It will create a new folder `<blog-dir-name>` having all the needed Hugo site files generated.

```yaml
# compose.yaml
name: hugo-blog

services:
  # Builds hugo site
  build:
    image: hugomods/hugo:exts-non-root
    command: build
    volumes:
      - "./:/src"
  # Serves hugo site; Live preview
  serve:
    image: hugomods/hugo:exts-non-root
    command: server --poll 1000ms --buildDrafts --noHTTPCache --renderToMemory
    volumes:
     - "./:/src"
     - "~/hugo_cache:/tmp/hugo_cache"
    ports:
     - "1313:1313"
```

Create a new `compose.yaml` file inside the new blog directory you've just created.

> ⚠&#xFE0F; Due to [docker-hugo\[#61\]: Using Docker for Dev](https://github.com/klakegg/docker-hugo/issues/61) issue, `--watch` option might not work.
> I have added `--poll 1000ms` flag so that the Hugo container watches my directory properly. (On Windows 10)

Then, open up the terminal and run `docker compose up -d`.

## 2. Create Obsidian Vault

Create a new Obsidian vault. (Or you can use your current vault).

`content` directory in the Hugo site we made earlier needs to be linked to the directory inside Obsidian vault. Create a symbolic link using your operating system's command.

```powershell
New-Item -ItemType SymbolicLink `
	-Path E:\my-vault\hugo-blog `
	-Target D:\Dev\Personal\hugo-blog\content
```

For Windows PowerShell, `-path` is <u>the directory inside Obsidian Vault</u>, `-target` is <u>the `content` directory of the Hugo site created earlier</u>.

## 3. Using `Templater` plugin

Each Hugo post needs to be in unique folder. 

```plaintext
.
└── content/
    └── posts/
        ├── my-first-post/
        │   ├── index.md
        │   └── image01.png
        └── my-second-post/
            └── index.md
```

Hugo follows the structure above. Therefore, each time we write a new post, we need to create a new directory with post's title, and `index.md` inside. Also, the directory's name has to be sluggified.

Using `Templater` Obsidian community plugin, we can make our lives easier.

```js
<%*
let title = await tp.system.prompt("Enter title of the post");
title = title.trim()
const titleSlug = title
		// make lower case and trim
		.toLowerCase().trim()
		// remove accents from charaters
		.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
		// replace invalid chars with spaces
        .replace(/[^a-z0-9\s-]/g, " ").trim()
        // replace multiple spaces or hyphens with a single hyphen
		.replace(/[\s-]+/g, "-");

await tp.file.rename(titleSlug);
await tp.file.move(`/20-🦊-hugo-blog/posts/${titleSlug}/index`);
-%>
---
title: <% title %>
description:
date: <% tp.file.creation_date("YYYY-MM-DDTHH:mm:ssZ")%>
draft: false
showFullContent: false
tags:
---

# <% title %>
```

Create a new template file. Using this template, we can create a new blog post directory and its `index.md`.

Use `Create new note from template` and select the template you've just created to write a new blog post.

![](attachments/Obsidian_FakraXQdCq.png)

Obsidian will prompt a new title. This is what `tp.system.prompt(...)` does!

![](attachments/Obsidian_ZIy6HuS2Ti.png)

Pressing enter will create a new directory and `index.md` inside.

> Sometimes, creating note from the template, while `serve` Hugo container is running, might not read by Hugo server. If so, just simply restart the `serve` container.

## 4. Attaching images

![](attachments/Pasted%20image%2020250824182120.png)

Under Obsidian settings, go to `File and links` and you will find the setting above. Changing this will create `attachments` folder inside each new blog post directory.

![](attachments/Pasted%20image%2020250824182242.png)

Also set `New link format` to `Relative path to file` so that the link to the images will be like `attachments/image_name.png`.

![](attachments/MiXgu5xYnD.png)

Attachments will be living under the same post's directory now.