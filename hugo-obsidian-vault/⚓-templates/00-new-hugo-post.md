<%*
const title = (await tp.system.prompt("Enter title of the post")).trim();
const titleSlug = title.toLowerCase().trim()
		.normalize("NFD").replace(/[\u0300-\u036f]/g, "")
        .replace(/[^a-z0-9\s-]/g,  " ").trim()
        .replace(/[\s-]+/g, "-");
await tp.file.rename(titleSlug);
await tp.file.move(`/🦊-hugo-blog/posts/${titleSlug}`);
-%>
---
title: <% title %>
description:
date: <% tp.file.creation_date("YYYY-MM-DD")%>
draft: false
showFullContent: false
tags:
---

# <% title %>

<% tp.file.cursor() %>