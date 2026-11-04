---
aliases:
  - Flet
tags:
  - learning
  - dev/frontend
date: 2026-09-27
---
**Sources**: [Flet](https://flet.dev/)

**Related:** [[Python]], [[Flutter]], [[Frontend]]

---

## Description

_Flet_ is a ``Python`` **framework for building web, desktop, and mobile apps without prior experience in** ``frontend`` **development**. Use **ready-made controls** to create your interface, connect your ``Python`` libraries and application logic, and handle user interactions without writing ``HTML``, ``CSS``, or ``JavaScript``.

---

## Examples

### Clicker

```python title:main.py
import flet as ft


def main(page: ft.Page):
    counter = ft.Text("0", size=50, data=0)

    def increment_click(e):
        counter.data += 1
        counter.value = str(counter.data)

    page.floating_action_button = ft.FloatingActionButton(
        icon=ft.Icons.ADD, on_click=increment_click
    )
    page.add(
        ft.SafeArea(
            expand=True,
            content=ft.Container(
                content=counter,
                alignment=ft.Alignment.CENTER,
            ),
        )
    )


ft.run(main)

```

---

## Claude Sessions
