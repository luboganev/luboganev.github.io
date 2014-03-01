---
layout: page
permalink: /playground/
title: Playground
description: "A playground for me here. Just do no click on anything unless you know what you are doing."
modified: 2013-09-11
tags: [playground]
---

A playground for me here. Just do no click on anything unless you know what you are doing.

<div markdown="0"><a href="chiponlineapp://" class="btn">Root schema, kein pfad oder parameter</a></div>

<div markdown="0"><a href="chiponlineapp://containerIdBeitrag/67984814" class="btn">Valid Id, Kein Fallback Parameter</a></div>

<div markdown="0"><a href="chiponlineapp://containerIdBeitrag/679848140" class="btn">Invalid Id, Kein Fallback Parameter</a></div>

<div markdown="0"><a href="chiponlineapp://containerIdBeitrag/67984814/?fallback=http%3A%2F%2Fwww.chip.de%2Fnews%2FSamsung-Galaxy-S5-Alle-Fakten-und-Hands-on_67984814.html" class="btn">Valid Id, URL encoded fallback parameter</a></div>

<div markdown="0"><a href="chiponlineapp://containerIdBeitrag/679848140/?fallback=http%3A%2F%2Fwww.chip.de%2Fnews%2FSamsung-Galaxy-S5-Alle-Fakten-und-Hands-on_67984814.html" class="btn">Invalid Id, URL encoded fallback parameter</a></div>

<div markdown="0"><a href="chiponlineapp://containerIdBeitrag/679848140/?fallback=http://www.chip.de/news/Samsung-Galaxy-S5-Alle-Fakten-und-Hands-on_67984814.html" class="btn">Invalid Id, fallback parameter</a></div>