+++
title = "Obsidian + ЛитРес: ищем книги не выходя из заметок"
draft = false
date = 2026-03-11
[taxonomies]
categories = ["obsidian"]
tags = ["book", "obsidian"]
+++

Если ты ведёшь библиотеку в Obsidian — эта доработка для тебя.

## Проблема

Стандартный плагин [Book Search](https://github.com/anpigon/obsidian-book-search-plugin)
ищет книги через Google Books и Naver. Для русскоязычных книг это работает плохо:
половина книг не находится, обложки отсутствуют, данные неполные.

## Решение

Я добавил поддержку **ЛитРес** — крупнейшего русскоязычного книжного сервиса.
Теперь поиск работает по названию, автору и ISBN прямо из базы ЛитРес.
Регистрация не нужна.

## Как это выглядит

1. Нажимаешь иконку в ribbon или вызываешь команду `Create new book note`
2. Вводишь название или автора — появляются результаты с обложками
3. Выбираешь книгу — создаётся готовая заметка с метаданными

Поле `cover`, `author`, `publisher`, `isbn` — всё заполняется автоматически.

## Установка

Через BRAT (бета-плагины для Obsidian):

1. Установи плагин [BRAT](https://github.com/TfTHacker/obsidian42-brat)
2. BRAT → Add Beta Plugin
3. Вставь: `https://github.com/malexple/obsidian-book-search-plugin`

## Шаблон для русских книг

```
---
tag: 📚Книга
title: "{{title}}"
author: [{{author}}]
publisher: {{publisher}}
publish: {{publishDate}}
isbn: {{isbn13}}
cover: {{coverUrl}}
status: не читал
created: {{DATE:YYYY-MM-DD}}
---

![обложка|150]({{coverUrl}})

# {{title}}
```

## Dataview: своя книжная полка

```dataview
TABLE WITHOUT ID
"![|60](" + cover + ")" as Обложка,
link(file.link, title) as Название,
author as Автор,
status as Статус
FROM #📚Книга
SORT status DESC
```

---

Если пользуешься — поставь ⭐ на GitHub, это помогает проекту.
[github.com/malexple/obsidian-book-search-plugin](https://github.com/malexple/obsidian-book-search-plugin)