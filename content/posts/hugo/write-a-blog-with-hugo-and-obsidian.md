---
title: Write a blog with Hugo and Obsidian
description: Make your own blog with Hugo and Obsidian!
date: 2025-08-24T17:28:42+09:00
keywords:
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

```yaml {title="compose.yaml"}
name: hugo-blog

services:
  build:
    image: hugomods/hugo:exts-non-root
    command: build --minify --gc
    volumes:
      - "./:/src"

  serve:
    image: hugomods/hugo:exts-non-root
    command: >-
	    server --poll 1000ms --buildDrafts --noHTTPCache --renderToMemory 
	           --baseURL http://127.0.0.1 --port 13130 
    volumes:
     - "./:/src"
     - "~/hugo_cache:/tmp/hugo_cache"
    ports:
     - "13130:13130"
```

Create a new `compose.yaml` file inside the new blog directory you've just created.

> ⚠&#xFE0F; Due to [docker-hugo\[#61\]: Using Docker for Dev](https://github.com/klakegg/docker-hugo/issues/61) issue, `--watch` option might not work.
> I have added `--poll 1000ms` flag so that the Hugo container watches my directory properly. (On Windows 10)

Then, open up the terminal and run `docker compose up -d`.

## 2. Create Obsidian Vault

Create a new Obsidian vault. I have created one inside my hugo site repository, named `hugo-obsidian-vault`.

![&#96;content/&#96; to &#96;(obsidian vault)/🦊-hugo-blog&#96; symlink](attachments/Pasted%20image%2020250904235711.png)

`content` directory in the Hugo site we made earlier needs to be linked to the directory inside Obsidian vault. Create a symbolic link using your operating system's command.

![&#96;assets/attachments/&#96; to &#96;(obsidian vault)/attachments&#96; symlink](attachments/Pasted%20image%2020250905011121.png)

Then I have created another symlink; `assets/attachments` linked to `attachments` folder inside Obsidian vault. All of attachment files will be located under `assets/attachments` now.

## 3. Using `Templater` plugin

```
.
└── content/
    └── posts/
        ├── my-first-post.md
        └── my-second-post.md
```

Hugo follows the structure above. Each post's title needs to be sluggified.

Using `Templater` Obsidian community plugin, we can make our lives easier.

```js {title="00-new-hugo-post.md"}
<%*
let title = await tp.system.prompt("Enter title of the post");
title = title.trim();
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
await tp.file.move(`/🦊-hugo-blog/posts/${titleSlug}`);
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

Create a new template file. Templater will read this template and create a new Obsidian note under `posts/`. Format of frontmatter might vary depending on the theme you are using.

Use `Create new note from template` and select the template you've just created to write a new blog post.

![](attachments/Obsidian_FakraXQdCq.png)

Obsidian will prompt a new title. This is what `tp.system.prompt(...)` does!

![](attachments/Pasted%20image%2020250905023303.png)

When you type the title, Templater will create a new note. (I have manually moved the new note from `posts/` to `posts/hugo` folder.)

## 4. Attaching images

![Attachments will be saved at &#96;(Vault root)/attachments/&#96;](attachments/Pasted%20image%2020250904233946.png)

In `File and links` settings, you can define where Obsidian stores attachments files. Since I have symlinked `assets/attachments` to Obsidian vault's `attachments` folder, I have picked that one from the settings window.

Images in the `assets/attachments` can be processed using [Asset Pipelines](https://gohugo.io/about/features/#asset-pipelines). You can refer to the articles like this one; [Perfect Image Processing with Hugo - Ryan Bagley](https://rb.ax/blog/perfect-image-processing-with-hugo/)

```go {open=false, title="layouts/_markup/render-image.html"}
{{ $image := resources.Get (urls.Parse .Destination).Path }}

{{- if and (ne .Page.Kind "section") (.Page.Section ) }}
  {{- $random := (substr (md5 .Destination) 0 5) }}
  <figure class="center" role="figure" aria-label="{{- .Text -}}">
    <input type="checkbox" id="zoomCheck-{{$random}}" hidden>
    <label for="zoomCheck-{{$random}}">
      <img class="zoomCheck" loading="lazy" decoding="async"
          src="{{- $image.RelPermalink | safeURL -}}" alt="{{- .Text -}}"/>
    </label>
    <figcaption>{{- .Text -}}</figcaption>
  </figure>
{{- else }}
  <img loading="lazy" decoding="async"
       src="{{- $image.RelPermalink | safeURL -}}" alt="{{- .Text -}}"/>
{{- end }}
```

```css {open=false, title="themes/terminal/assets/css/terminal.css"}
/* Placeholder file for your custom settings. */
/* You can get the color scheme variables from https://panr.github.io/terminal-css/ */

/* Click and zoom image effect */
@media screen and (min-width: 769px) {
  .post-content input[type="checkbox"]:checked~label>img {
    transform: scale(1.6);
    cursor: zoom-out;
    position: relative;
    z-index: 999;
  }

  .post-content img.zoomCheck {
    transition: transform 0.15s ease;
    z-index: 999;
    cursor: zoom-in;
  }
}

figure {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;

  & figcaption {
    font-size: 0.875rem;
    font-style: italic;
    text-align: center;
  }
}
```

I referred to this article([Image Zoom-In effect with HUGO](https://adityatelange.in/blog/hugo-image-zoom-in/)) to make images clickable and zoomable.