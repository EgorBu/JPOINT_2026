---
theme: seriph
title: Агентостроительный завод
info: |
  ## Агентостроительный завод: путь от одного агента к тысячам
  JPoint 2026 | Егор Булычев
author: Егор Булычев
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
download: true
editor: false
layout: cover
background: /materials/main_image.png
---

# Агентостроительный завод

## Путь от одного агента к тысячам

### Как построить инфраструктуру для масштабного запуска кодовых агентов

<div class="abs-br m-6 text-lg opacity-80">
  JPoint 2026 · Егор Булычев
</div>

---
layout: image-right
image: /materials/qr-code.png
backgroundSize: contain
---

### Кому и зачем

* Многие из вас уже пользуются кодовыми агентами — GigaIDE, Codex, Cursor, Claude Code...
* Я расскажу, какой путь надо пройти, чтобы сделать сильную агентную кодовую модель
* А конкретно в этом докладе я расскажу про масштабирование запуска кодовых агентов

> Этот доклад — про **кухню**: как запустить 1000 агентов параллельно и не сжечь кластер

---
layout: image-right
image: /materials/egor_image.png
---

### Кто я?
* Работаю в GigaCode в Сбере
* Работал в Huawei - занимался кодовыми моделями
* Работал в JetBrains
* Работал в стартапе, занимавшемся ML4Code еще до того, как это стало модным в 2017~2019 годах

<!-- https://qr-online.ru/ -->

---
class: flex flex-col justify-start
---

### Что мы делаем?

<div class="grid grid-cols-2 gap-10 mt-6">

<div>

#### Prod

* GigaIDE
* plugin'ы для JB/VSCode
* GitVerse
* агентизация внутри Сбербанка

</div>

<div>

#### RnD

* Полный цикл обучения кодовых моделей
  * агентная
  * автодополнение
* Подготовка данных
* Подготовка бенчмарков и обмеры

</div>

</div>

<blockquote class="mt-8 text-left text-sm opacity-90 border-l-4 border-current/25 pl-4 italic max-w-none">
В корпоративном секторе России, в том числе в Сбере, по объёму кода и числу разработчиков лидируют JVM-экосистемы — поэтому мы целенаправленно усиливаем JVM-специфичные навыки моделей.
</blockquote>


---
layout: image-right
image: /materials/meme.png
backgroundSize: contain
---

### О чём пойдёт речь

1. 1 агент
2. Данные и обмеры
3. 100 агентов
4. Планы на 1000 агентов


---

### Почему 1000 агентов?

- Мы живем в век ИИ - а это ~~век~~ 4 года масштабирования и скорости изменений
- Больше данных + больше модель + качественное обучение -> лучше результаты
- Цикл: **сбор данных → обучение → обмер модели** — должен быть быстрым
- 1000 параллельных агентов ≈ минимальная планка для быстрых итераций
- Цель: модель с близкими к SoTA-результатами на кодовых бенчмарках

---

### Зачем масштабировать?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">20</th>
      <th style="width: 25%;">200</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Замер бенчмарка (500 задач)</b></td>
      <td>9~14 часов</td>
      <td>~2 часа</td>
      <td>&lt; 1 часа</td>
    </tr>
    <tr>
      <td><b>Генерация 100к решений задач</b></td>
      <td>~100 дней</td>
      <td>~10 дней</td>
      <td>~дни</td>
    </tr>
  </tbody>
</table>

> **СПОЙЛЕР:** лучшая модель у того, у кого быстрее итерации

---

### Зачем масштабировать?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">20</th>
      <th style="width: 25%;">200</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Сбор данных</b></td>
      <td>100 дней</td>
      <td>10 дней</td>
      <td>2 дня</td>
    </tr>
    <tr>
      <td><b>Обучение</b></td>
      <td>7 дней</td>
      <td>7 дней</td>
      <td>7 дней</td>
    </tr>
    <tr>
      <td><b>Обмеры</b></td>
      <td>~1 день</td>
      <td>~2 часа</td>
      <td>~2 часа</td>
    </tr>
    <tr style="background-color: #f9f9f9; font-weight: bold;">
      <td>Суммарно</td>
      <td>~107 дней</td>
      <td>~17 дней</td>
      <td>~9 дней</td>
    </tr>
    <tr style="background-color: #eef; font-weight: bold;">
      <td>Ускорение</td>
      <td>1</td>
      <td>~6×</td>
      <td>~11×</td>
    </tr>
  </tbody>
</table>

<br>

**Зачем:** За то же самое время можно проверить в ~10 раз больше гипотез.

---
layout: image-right
image: /materials/1agent.png
backgroundSize: contain
---

## Один агент

1. **1 агент**
    * Особенность кодовой области для агентов
    * Агентный фремворк
2. Данные и обмеры
3. 100 агентов
4. Планы на 1000 агентов

---

### Особенность кодовой области для агентов

В отличие от математики, текста, изображений:

- **Нужна изоляция**: каждая SWE задача == свой контейнер с окружением.
- **Риски**: запущенный код на компьютере с неограниченными правами == потенциальные проблемы.
- **Автоматическая верификация**: автоматически проверяем тесты проходят или нет.

За дополнительную сложность делаем возможным автоматическое масштабирование и верифицирование.

---

### Особенность кодовой области для агентов

<img src="/materials/meta_safety.png" class="h-[450px] mx-auto" />


---

### Анатомия запуска SWE задачи - три фазы задачи

1. **Подготовка**: клонировать репозиторий, установить зависимости
2. **Решение**: запускается LLM + агентный фреймворк + изолированное окружение -> и агент вносит изменения
3. **Проверка**: запуск тестов, сбор результатов

> грубо говоря, один агент работает над кодовой задачей в течение 5~20 минут

---

### Агентный фремворк
#### Упрощенный взгляд

<div class="grid grid-cols-[2fr_3fr] gap-8 items-center">

<div>

- Получает задачу: «исправь баг» / «добавь фичу»
- Агентный цикл
- Набор инструментов (чтение файлов, редактирование, bash)


> LLM вне агентного фремворка

</div>

<div class="flex items-center justify-center">

<img src="/materials/simple_agent_2.png" class="max-w-full max-h-[640px]" />

</div>

</div>

---

### Агентный фреймворк Claude Code

<img src="/materials/claude_pirate.png" class="h-[450px] mx-auto" />


---

### Claude Сode
<!-- note: тут добавляем подробности-->
<img src="/materials/simple_agent.png" class="h-[450px] mx-auto" />

---

### Claude Code под 🔍

<!-- 1. Пользователь: отправляет запросы, утверждает разрешения, проверяет выходные данные. 
2. Интерфейсы: интерактивный интерфейс командной строки, автономный интерфейс командной строки (claude -p), агент SDK и IDE/рабочий стол/браузер. Все поверхности питают один и тот же контур. 
3. Цикл агента: итерационный цикл вызова модели, отправки инструментов и сбора результатов, реализованный как асинхронный генератор queryLoop() в query.ts.  
4. Система разрешений: оценка правила «сначала запрет» (permissions.ts), автоматический классификатор ML и перехват на основе перехватчиков (types/hooks.ts). 
5. Инструменты: до 54 встроенных инструментов (19 безусловных, 35 зависящих от флагов функций и типа пользователя), собранных с помощью assembleToolPool() (tools.ts), объединенных с инструментами, предоставляемыми MCP. Плагины вносят свой вклад косвенно через серверы MCP и реестр навыков/команд. 
6. Состояние и постоянство: в основном транскрипты сеансов JSONL только для добавления (sessionStorage.ts), глобальная история запросов (history.ts) и файлы боковой цепи субагента. 
7. Среда выполнения: выполнение оболочки с дополнительной изолированной программной средой (shouldUseSandbox.ts), операции с файловой системой, получение данных из Интернета, подключения к серверу MCP и удаленное выполнение.-->

<img src="/materials/not_simple_agent.png" class="h-[450px] mx-auto" />

---

### Claude Code под 🔍

<!-- Исходник-сетка: materials/not_simple_agent_grid.png (2x2 копий диаграммы с красной рамкой на своей зоне). Нарезка: convert -crop 512x199+<x>+<y>. -->

<img src="/materials/not_simple_agent_1.png" alt="Surface Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_2.png" alt="Core Layer + State Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_3.png" alt="Safety / Action Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_4.png" alt="Backend Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

<!-- ---

### С чем придется столкнуться при разработке агентного фреймворка

<img src="/materials/eng_task.png" class="h-[450px] mx-auto" />

> за скобками - бизнес логика, проверка изменений на качество, A/B тестирование и т.д. -->

---

### Оценка трудозатрат

<img src="/materials/top_vs_dyi_agent.png" class="h-[450px] mx-auto" />

---

### TLDR - 1 агент

узнали
* зачем нужна изоляция для задачи 
* зачем нужна автоверификация решения задачи
* как устроен агентный фреймворк


---
layout: image-right
image: /materials/10_agents.png
backgroundSize: contain
---

# Данные и обмеры

1. 1 агент
2. **Данные и обмеры**
    * сбор задач для агентов
    * обмеры моделей
    * данные для обучения
    * инфраструктура
3. 100 агентов
4. Планы на 1000 агентов

<!-- ---

### Сбор задач для агентов

<div class="grid grid-cols-2 gap-8">

<div>

#### Зачем собирать задачи?
* Наша цель - сделать крутую кодовую агентную модель.
* Как - дать ей возможность прорешать как можно больше задач.

</div>

<div>

#### Где взять SWE задачи?
* Сбор реальных задач.
* Сбор синтетических задач.

**Требования к SWE задачам:**
* верифицируемость (есть тесты)
* воспроизводимость (docker-образ)
* разнообразие (языки, типы багов)

</div>

</div> -->

---

### Сбор реальных задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>искать связанные issues ⟷ PR в репозиториях</td>
</tr>
<tr>
<td><span style="visibility: hidden">Клонирование репозитория</span></td>
<td><span style="visibility: hidden">проверять наличие тестов в коммитах, связанных с PR</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Подготовка образа</span></td>
<td><span style="visibility: hidden">по правилам или билд агентом</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация корректности задачи</span></td>
<td><div style="visibility: hidden">

- наличие «золотого» патча
- до применения «золотого» патча какие-то тесты падают (**fail-to-pass**)
- после применения «золотого» патча проходят
- регрессионные тесты (**pass-to-pass**) должны до и после проходить

</div></td>
</tr>
</tbody>
</table>

---

### Сбор реальных задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>искать связанные issues ⟷ PR в репозиториях</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверять наличие тестов в коммитах, связанных с PR</td>
</tr>
<tr>
<td><span style="visibility: hidden">Подготовка образа</span></td>
<td><span style="visibility: hidden">по правилам или билд агентом</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация корректности задачи</span></td>
<td><div style="visibility: hidden">

- наличие «золотого» патча
- до применения «золотого» патча какие-то тесты падают (**fail-to-pass**)
- после применения «золотого» патча проходят
- регрессионные тесты (**pass-to-pass**) должны до и после проходить

</div></td>
</tr>
</tbody>
</table>

---

### Сбор реальных задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>искать связанные issues ⟷ PR в репозиториях</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверять наличие тестов в коммитах, связанных с PR</td>
</tr>
<tr>
<td>Подготовка образа</td>
<td>по правилам или билд агентом</td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация корректности задачи</span></td>
<td><div style="visibility: hidden">

- наличие «золотого» патча
- до применения «золотого» патча какие-то тесты падают (**fail-to-pass**)
- после применения «золотого» патча проходят
- регрессионные тесты (**pass-to-pass**) должны до и после проходить

</div></td>
</tr>
</tbody>
</table>

---

### Сбор реальных задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>искать связанные issues ⟷ PR в репозиториях</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверять наличие тестов в коммитах, связанных с PR</td>
</tr>
<tr>
<td>Подготовка образа</td>
<td>по правилам или билд агентом</td>
</tr>
<tr>
<td>Верификация корректности задачи</td>
<td>

- наличие «золотого» патча
- до применения «золотого» патча какие-то тесты падают (**fail-to-pass**)
- после применения «золотого» патча проходят
- регрессионные тесты (**pass-to-pass**) должны до и после проходить

</td>
</tr>
</tbody>
</table>

> На самом деле даже сложнее - используют LLM для оценки постановки задачи в issues и т.д.

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td><span style="visibility: hidden">PRs with tests</span></td><td style="text-align: right"><span style="visibility: hidden">8 593 722</span></td><td style="text-align: right"><span style="visibility: hidden">101 958</span></td></tr>
<tr><td><span style="visibility: hidden">PR linked with issue and test</span></td><td style="text-align: right"><span style="visibility: hidden">805 598</span></td><td style="text-align: right"><span style="visibility: hidden">50 797</span></td></tr>
<tr><td><span style="visibility: hidden">Repo based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">583 809</span></td><td style="text-align: right"><span style="visibility: hidden">21 692</span></td></tr>
<tr><td><span style="visibility: hidden">Successful tasks w/ F2P</span></td><td style="text-align: right"><span style="visibility: hidden">41 349</span></td><td style="text-align: right"><span style="visibility: hidden">4 006</span></td></tr>
<tr><td><span style="visibility: hidden">Issue text based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">32 079</span></td><td style="text-align: right"><span style="visibility: hidden">3 617</span></td></tr>
</tbody>
</table>

<div style="visibility: hidden">

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

</div>

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td>PRs with tests</td><td style="text-align: right">8 593 722</td><td style="text-align: right">101 958</td></tr>
<tr><td><span style="visibility: hidden">PR linked with issue and test</span></td><td style="text-align: right"><span style="visibility: hidden">805 598</span></td><td style="text-align: right"><span style="visibility: hidden">50 797</span></td></tr>
<tr><td><span style="visibility: hidden">Repo based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">583 809</span></td><td style="text-align: right"><span style="visibility: hidden">21 692</span></td></tr>
<tr><td><span style="visibility: hidden">Successful tasks w/ F2P</span></td><td style="text-align: right"><span style="visibility: hidden">41 349</span></td><td style="text-align: right"><span style="visibility: hidden">4 006</span></td></tr>
<tr><td><span style="visibility: hidden">Issue text based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">32 079</span></td><td style="text-align: right"><span style="visibility: hidden">3 617</span></td></tr>
</tbody>
</table>

<div style="visibility: hidden">

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

</div>

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td>PRs with tests</td><td style="text-align: right">8 593 722</td><td style="text-align: right">101 958</td></tr>
<tr><td>PR linked with issue and test</td><td style="text-align: right">805 598</td><td style="text-align: right">50 797</td></tr>
<tr><td><span style="visibility: hidden">Repo based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">583 809</span></td><td style="text-align: right"><span style="visibility: hidden">21 692</span></td></tr>
<tr><td><span style="visibility: hidden">Successful tasks w/ F2P</span></td><td style="text-align: right"><span style="visibility: hidden">41 349</span></td><td style="text-align: right"><span style="visibility: hidden">4 006</span></td></tr>
<tr><td><span style="visibility: hidden">Issue text based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">32 079</span></td><td style="text-align: right"><span style="visibility: hidden">3 617</span></td></tr>
</tbody>
</table>

<div style="visibility: hidden">

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

</div>

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td>PRs with tests</td><td style="text-align: right">8 593 722</td><td style="text-align: right">101 958</td></tr>
<tr><td>PR linked with issue and test</td><td style="text-align: right">805 598</td><td style="text-align: right">50 797</td></tr>
<tr><td>Repo based filtering</td><td style="text-align: right">583 809</td><td style="text-align: right">21 692</td></tr>
<tr><td><span style="visibility: hidden">Successful tasks w/ F2P</span></td><td style="text-align: right"><span style="visibility: hidden">41 349</span></td><td style="text-align: right"><span style="visibility: hidden">4 006</span></td></tr>
<tr><td><span style="visibility: hidden">Issue text based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">32 079</span></td><td style="text-align: right"><span style="visibility: hidden">3 617</span></td></tr>
</tbody>
</table>

<div style="visibility: hidden">

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

</div>

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td>PRs with tests</td><td style="text-align: right">8 593 722</td><td style="text-align: right">101 958</td></tr>
<tr><td>PR linked with issue and test</td><td style="text-align: right">805 598</td><td style="text-align: right">50 797</td></tr>
<tr><td>Repo based filtering</td><td style="text-align: right">583 809</td><td style="text-align: right">21 692</td></tr>
<tr><td>Successful tasks w/ F2P</td><td style="text-align: right">41 349</td><td style="text-align: right">4 006</td></tr>
<tr><td><span style="visibility: hidden">Issue text based filtering</span></td><td style="text-align: right"><span style="visibility: hidden">32 079</span></td><td style="text-align: right"><span style="visibility: hidden">3 617</span></td></tr>
</tbody>
</table>

<div style="visibility: hidden">

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

</div>

---

### Сбор реальных задач

#### Воронка задач на примере SWE-rebench V2

<table>
<thead>
<tr><th style="text-align: left">Stage</th><th style="text-align: right">PRs</th><th style="text-align: right">Repos</th></tr>
</thead>
<tbody>
<tr><td>PRs</td><td style="text-align: right">29 511 758</td><td style="text-align: right">145 306</td></tr>
<tr><td>PRs with tests</td><td style="text-align: right">8 593 722</td><td style="text-align: right">101 958</td></tr>
<tr><td>PR linked with issue and test</td><td style="text-align: right">805 598</td><td style="text-align: right">50 797</td></tr>
<tr><td>Repo based filtering</td><td style="text-align: right">583 809</td><td style="text-align: right">21 692</td></tr>
<tr><td>Successful tasks w/ F2P</td><td style="text-align: right">41 349</td><td style="text-align: right">4 006</td></tr>
<tr><td>Issue text based filtering</td><td style="text-align: right">32 079</td><td style="text-align: right">3 617</td></tr>
</tbody>
</table>

> ~0.1% PR, 2% репозиториев подходят для задач,
> примерно 9 задач на репозиторий

---

### Сбор синтетических задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>фильтрация по языкам и т.д.</td>
</tr>
<tr>
<td><span style="visibility: hidden">Клонирование репозитория</span></td>
<td><span style="visibility: hidden">проверка наличия тестов</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Подготовка образа</span></td>
<td><span style="visibility: hidden">по правилам или билд агентом</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация образа</span></td>
<td><span style="visibility: hidden">все тесты должны проходить - это базовый образ</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Логика подготовки задачи</span></td>
<td><div style="visibility: hidden">

- найти покрытые тестами функции
- повредить их (получить «золотой» патч)
- получить **fail-to-pass** & **pass-to-pass** тесты
- подготовить образ для задачи

</div></td>
</tr>
</tbody>
</table>

<div style="visibility: hidden">

> Логика подготовки задачи может быть разнообразной - от правил до LLM

</div>

---

### Сбор синтетических задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>фильтрация по языкам и т.д.</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверка наличия тестов</td>
</tr>
<tr>
<td><span style="visibility: hidden">Подготовка образа</span></td>
<td><span style="visibility: hidden">по правилам или билд агентом</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация образа</span></td>
<td><span style="visibility: hidden">все тесты должны проходить - это базовый образ</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Логика подготовки задачи</span></td>
<td><div style="visibility: hidden">

- найти покрытые тестами функции
- повредить их (получить «золотой» патч)
- получить **fail-to-pass** & **pass-to-pass** тесты
- подготовить образ для задачи

</div></td>
</tr>
</tbody>
</table>

<div style="visibility: hidden">

> Логика подготовки задачи может быть разнообразной - от правил до LLM

</div>

---

### Сбор синтетических задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>фильтрация по языкам и т.д.</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверка наличия тестов</td>
</tr>
<tr>
<td>Подготовка образа</td>
<td>по правилам или билд агентом</td>
</tr>
<tr>
<td><span style="visibility: hidden">Верификация образа</span></td>
<td><span style="visibility: hidden">все тесты должны проходить - это базовый образ</span></td>
</tr>
<tr>
<td><span style="visibility: hidden">Логика подготовки задачи</span></td>
<td><div style="visibility: hidden">

- найти покрытые тестами функции
- повредить их (получить «золотой» патч)
- получить **fail-to-pass** & **pass-to-pass** тесты
- подготовить образ для задачи

</div></td>
</tr>
</tbody>
</table>

<div style="visibility: hidden">

> Логика подготовки задачи может быть разнообразной - от правил до LLM

</div>

---

### Сбор синтетических задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>фильтрация по языкам и т.д.</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверка наличия тестов</td>
</tr>
<tr>
<td>Подготовка образа</td>
<td>по правилам или билд агентом</td>
</tr>
<tr>
<td>Верификация образа</td>
<td>все тесты должны проходить - это базовый образ</td>
</tr>
<tr>
<td><span style="visibility: hidden">Логика подготовки задачи</span></td>
<td><div style="visibility: hidden">

- найти покрытые тестами функции
- повредить их (получить «золотой» патч)
- получить **fail-to-pass** & **pass-to-pass** тесты
- подготовить образ для задачи

</div></td>
</tr>
</tbody>
</table>

<div style="visibility: hidden">

> Логика подготовки задачи может быть разнообразной - от правил до LLM

</div>

---

### Сбор синтетических задач

<table>
<thead>
<tr><th style="width: 35%">Этап</th><th>Как сделать</th></tr>
</thead>
<tbody>
<tr>
<td>Парсинг гитхаба</td>
<td>фильтрация по языкам и т.д.</td>
</tr>
<tr>
<td>Клонирование репозитория</td>
<td>проверка наличия тестов</td>
</tr>
<tr>
<td>Подготовка образа</td>
<td>по правилам или билд агентом</td>
</tr>
<tr>
<td>Верификация образа</td>
<td>все тесты должны проходить - это базовый образ</td>
</tr>
<tr>
<td>Логика подготовки задачи</td>
<td>

- найти покрытые тестами функции
- повредить их (получить «золотой» патч)
- получить **fail-to-pass** & **pass-to-pass** тесты
- подготовить образ для задачи

</td>
</tr>
</tbody>
</table>

> Логика подготовки задачи может быть разнообразной - от правил до LLM

---

### Сбор синтетических задач

<div class="grid grid-cols-2 gap-6">

<div>

### Оригинал

```java
List<Double> normalize(List<Double> xs) {
    if (xs.isEmpty()) return List.of();
    double min = Collections.min(xs);
    double max = Collections.max(xs);
    // Все значения одинаковые -> нули
    if (min == max)
        return Collections.nCopies(xs.size(), 0.0);
    var out = new ArrayList<Double>();
    for (double x : xs)
        out.add((x - min) / (max - min));
    return out;
}
```

</div>

<div>

### Синтетика (bugged)

```java
List<Double> normalize(List<Double> xs) {
    if (xs.isEmpty()) return List.of();
    double min = Collections.min(xs);
    double max = Collections.max(xs);
    // Эта часть кода была удалена


    var out = new ArrayList<Double>();
    for (double x : xs)
        out.add((x - min) / (max - min));
    return out;
}
```

</div>

</div>

---

### Сбор синтетических задач
#### Воронка задач на примере SWE-smith

<table>
<thead>
<tr>
  <th>Стратегия</th>
  <th style="text-align: right">Репозиториев</th>
  <th style="text-align: right">Кандидаты</th>
  <th style="text-align: right">Задачи</th>
  <th style="text-align: right">Процент успешных</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Процедурная (AST-трансформации)</td>
  <td style="text-align: right">121</td>
  <td style="text-align: right">38 866</td>
  <td style="text-align: right">15 641</td>
  <td style="text-align: right"><strong>40.2%</strong></td>
</tr>
<tr>
  <td>LLM (Modify + Rewrite + PR Mirroring)</td>
  <td style="text-align: right">128</td>
  <td style="text-align: right">50 792</td>
  <td style="text-align: right">24 404</td>
  <td style="text-align: right"><strong>48.0%</strong></td>
</tr>
<tr>
  <td>Комбинирование LLM + процедурная</td>
  <td style="text-align: right">124</td>
  <td style="text-align: right">10 416</td>
  <td style="text-align: right">10 092</td>
  <td style="text-align: right"><strong>96.9%</strong></td>
</tr>
<tr style="font-weight: 600; background: #f7f7f7">
  <td>Итого</td>
  <td style="text-align: right">129</td>
  <td style="text-align: right">100 074</td>
  <td style="text-align: right">50 137</td>
  <td style="text-align: right">50.1%</td>
</tr>
</tbody>
</table>

> ~388 задач на репозиторий

---

### Сбор задач: синтетика vs реальные

<table>
<thead>
<tr>
  <th style="width: 28%"></th>
  <th>Синтетика (SWE-smith)</th>
  <th>Реальные (SWE-rebench V2)</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Репозиториев</td>
  <td>129</td>
  <td>3 617</td>
</tr>
<tr>
  <td>Задач</td>
  <td>50 137</td>
  <td>32 079</td>
</tr>
<tr>
  <td>Задач на репозиторий</td>
  <td>~388</td>
  <td>~9</td>
</tr>
<tr>
  <td>Источник бага</td>
  <td>AST-трансформации, LLM-генерация, комбинации</td>
  <td>реальные PR с issue на GitHub</td>
</tr>
<tr>
  <td>Разнообразие багов</td>
  <td>ограничено набором трансформаций / стилем LLM</td>
  <td>отражает реальную разработку</td>
</tr>
<tr>
  <td>Issue / постановка</td>
  <td>сгенерирована LLM, сгенерирована по правилу</td>
  <td>написана человеком</td>
</tr>
</tbody>
</table>

> **Синтетика** — много, легко, однообразно. **Реальные** — мало, сложно, разнообразно.

---

### Обмер моделей

> Нельзя улучшить то, что не умеешь измерять. (с)

Задачи собраны — дальше нужно понимать, **что модель реально умеет**: где стало лучше после обучения, где регресс, где упёрлись в потолок.



---

### Автоматический обмер моделей

> на примере SWE-бенчмарков (SWE-MERA, SWE-bench Pro, ...):

1. **Задача**: описание проблемы в issue
2. **Окружение**: Образ(фиксированный commit и зависимости) + команда тестов
3. **Верификация**:
   - Команда для запуска тестов
   - Тесты для проверки проблемы (Fail-to-Pass)  
   - Регрессионные тесты (Pass-to-Pass)


---

### SWE-bench verified
<img src="/materials/SWE-bench.png" class="h-[450px] mx-auto" />

> замеряют связку - модель + агентный фремворк

<!-- ---

### Ручной обмер моделей

* Это “слепое” сравнение моделей
    * Пользователь задаёт один и тот же запрос двум моделям и не знает, какая есть какая. Голосует за лучший ответ → так собирается сравнительная метрика качества (без маркетинга и бенчмарков от вендоров).

* Рейтинг строится через Elo (как в шахматах)
    * Модели получают рейтинг на основе побед/поражений в парах. Это даёт более “живую” оценку, чем статические тесты, потому что учитывает реальные пользовательские запросы.

* Нет проверки на “правильность”, только предпочтение -->

---

### Ручной обмер моделей - LLM Arena
<img src="/materials/llm-arena-code.png" class="h-[450px] mx-auto" />

> Предпочтения людей != выбор правильного ответа

---

### LLM Arena Leaderboard

<img src="/materials/arena.png" class="h-[450px] mx-auto" />

> рейтинг 1572 выигрывает у 1530 в ~56% случаев

> Opus 4.7 (1572) выиграет у gpt-4o (1368)  в ~76% случаев


---

### TLDR: Обмеры моделей

<table>
<thead>
<tr>
  <th style="width: 22%"></th>
  <th>Автоматический</th>
  <th>Ручной</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Область</td>
  <td>верифицируемые задачи: тесты, компилятор, линтер</td>
  <td>субъективные предпочтения: читаемость, стиль, UX</td>
</tr>
<tr>
  <td>Критерий</td>
  <td>объективный сигнал (pass/fail)</td>
  <td>голос человека</td>
</tr>
<tr>
  <td>Стоимость</td>
  <td>дёшево, скейлится, воспроизводимо</td>
  <td>дорого, медленно, шумно</td>
</tr>
<tr>
  <td>Примеры</td>
  <td>SWE-bench, SWE-MERA, SWE-bench Pro</td>
  <td>LLM Arena, Copilot Arena</td>
</tr>
</tbody>
</table>

> Для кодовых агентов основной источник сигнала — автоматический; но ручной тоже нужен, чтобы не получить "гениального" аутиста.

> Та же инфраструктура (Docker + агент + тесты) переиспользуется и в генерации данных →

---

### Обмеры моделей ⟷ генерация данных

| Критерий | Авто-обмеры моделей | Генерация данных |
| --- | --- | --- |
| Docker-окружение |✅ | ✅ |
| Работа агента | ✅ | ✅ |
| Тесты / верификация | ✅ | ✅ |
| Результат | Pass rate | Диалоги |

<!-- ---

### Инфраструктура
* 10-20 разработчиков-исследователей
* несколько серверов мощных с кучей CPU/RAM/disc
* docker'ы и т.д. - все доступно
* есть определенные ограничения безопасности - например, не выставлять наружу порты

> команда долго жила на ней

> но дальше будет описание того, что меня сподвигло все поменять -->

---

### Инфраструктура
<img src="/materials/10man_1server.png" class="h-[450px] mx-auto mt-8" />

---

### Что может пойти не так?
<img src="/materials/bare_problems.png" class="h-[450px] mx-auto mt-8" />


---
layout: image-right
image: /materials/this_is_fine.png
backgroundSize: contain
---

### Что может пойти не так?
TLDR
* Нет масштабирования пайплайнов
* Инфраструктурный ад


---

### TLDR: Данные и обмеры

узнали
* как собирать реальные и синтетические задачи для агентов
* как обмерять модели
* почему обмер модели эквивалентен генерации для SWE задач
* что заставило переосмыслить инфраструктуру ради масштабирования 

---
layout: image-right
image: /materials/think.png
backgroundSize: contain
---

#### Сотни агентов 


1. 1 агент
2. Данные и обмеры
3. **100 агентов**
    * Инфраструктура
4. Планы на 1000 агентов


<!-- ---

### Bare Metal vs Kubernetes

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 40%;">Критерий / Задача</th>
      <th style="width: 30%;">Bare Metal</th>
      <th style="width: 30%;">Kubernetes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Масштабирование</b><br><span style="font-size: 12px;">(запуск множества агентов)</span></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс<br><span style="font-size: 12px;">(можно запускать на многих серверах сразу)</span></td>
    </tr>
    <tr>
      <td><b>CPU/RAM</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс<br><span style="font-size: 12px;">(учитывает лимиты при запуске)</span></td>
    </tr>
    <tr>
      <td><b>Изоляция</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс<br><span style="font-size: 12px;">(работает ТОЛЬКО с контейнерами)</span></td>
    </tr>
    <tr>
      <td><b>Масштабный LLM inference</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
    </tr>
    <tr>
      <td><b>Диски</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
    </tr>
    <tr>
      <td><b>Многошаговые пайплайны</b><br><span style="font-size: 12px;">(DAG: подготовка → агент → тесты)</span></td>
      <td style="background-color: #e6ffe6;">✅ Плюс<br><span style="font-size: 12px;">(всё просто делается на Python)</span></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
    </tr>

  </tbody>
</table> -->


---

### Запланированная инфраструктура
<img src="/materials/stage3.png" class="w-[950px] mx-auto" />

<!-- ---

### Зачем нужен сервис LLM proxy?

Раньше
* 1 vllm проброшенная справлялась с 10-30 агентами на сервере

Нужно
* пробрасывать много vllm для десятков-сотен-тысяч агентов
* балансировать нагрузку между ними
* не надо каждому агенту указывать ip:port, будет единый для всех -->

---

### Зачем нужен сервис LLM proxy?

Раньше
<img src="/materials/before_llm_proxy.png" class="w-[950px] mx-auto" />

---

### Зачем нужен сервис LLM proxy?

Сейчас
<img src="/materials/llm_proxy.png" class="w-[950px] mx-auto" />

> load balancing, агент приклеивается к одной LLM для кэширования запросов, подключение/отключение LLM под нагрузкой и т.д.
<!-- TODO перерисовать картинки -->

---

### Как решить проблему с хранением артефактов

* S3 - главное хранилище артефактов
  * "бесконечное" хранилище
  * разделяем ответственность с инфраструктурными командами
* диски на серверах
  * для текущих задач только

---

### Зачем нам нужен DAG?

<div class="grid grid-cols-[3fr_4fr] gap-8 items-center mt-4">

<div>

<ul>
<li>Мы хотим <strong>честные обмеры</strong> — метрика должна отражать умения модели, а не дыры в стенде</li>
<li style="visibility: hidden">Если агент и тесты живут в <strong>одном контейнере</strong> — агент может подсмотреть/подправить тесты и выбить "100%" на бенче</li>
<li style="visibility: hidden">Разделение на шаги (агент → изоляция артефактов → тесты в чистом окружении) убирает эту лазейку</li>
</ul>

</div>

<div>

<img src="/materials/SWE-bench-broken.png" class="max-h-[520px] w-full object-contain" />

</div>

</div>

---

### Зачем нам нужен DAG?

<div class="grid grid-cols-[3fr_4fr] gap-8 items-center mt-4">

<div>

<ul>
<li>Мы хотим <strong>честные обмеры</strong> — метрика должна отражать умения модели, а не дыры в стенде</li>
<li>Если агент и тесты живут в <strong>одном контейнере</strong> — агент может подсмотреть/подправить тесты и выбить "100%" на бенче</li>
<li style="visibility: hidden">Разделение на шаги (агент → изоляция артефактов → тесты в чистом окружении) убирает эту лазейку</li>
</ul>

</div>

<div>

<img src="/materials/SWE-bench-broken.png" class="max-h-[520px] w-full object-contain" />

</div>

</div>

---

### Зачем нам нужен DAG?

<div class="grid grid-cols-[3fr_4fr] gap-8 items-center mt-4">

<div>

<ul>
<li>Мы хотим <strong>честные обмеры</strong> — метрика должна отражать умения модели, а не дыры в стенде</li>
<li>Если агент и тесты живут в <strong>одном контейнере</strong> — агент может подсмотреть/подправить тесты и выбить "100%" на бенче</li>
<li>Разделение на шаги (агент → изоляция артефактов → тесты в чистом окружении) убирает эту лазейку</li>
</ul>

</div>

<div>

<img src="/materials/SWE-bench-broken.png" class="max-h-[520px] w-full object-contain" />

</div>

</div>

---

### DAG == Честный обмер

<img src="/materials/eval_pod.png" class="h-[600px] mx-auto" />

---

### Где найти DAG для K8s?
Отсутствие многошаговых пайплайнов решается через...
<img src="/materials/argo.png" class="w-[950px] mx-auto" />

---

<img src="/materials/argo2.png" class="w-[950px] mx-auto" />
...искал медь, а нашел золото

---

### Итог: Инфраструктура мечты как мы изначально видели ее
<!-- 
<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Критерий / Задача</th>
      <th style="width: 22%;">Bare Metal</th>
      <th style="width: 23%;">Kubernetes</th>
      <th style="width: 30%;">K8s + LLM Proxy + S3 + Argo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Масштабирование</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>
    <tr>
      <td><b>CPU/RAM</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>
    <tr>
      <td><b>Изоляция</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>
    <tr>
      <td><b>Масштабный LLM inference</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>
    <tr>
      <td><b>Диски</b></td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>
    <tr>
      <td><b>Многошаговые пайплайны</b></td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
      <td style="background-color: #ffe6e6;">❌ Минус</td>
      <td style="background-color: #e6ffe6;">✅ Плюс</td>
    </tr>

  </tbody>
</table> -->

<img src="/materials/stage3.png" class="w-[950px] mx-auto" />


---

### ...пора!
<img src="/materials/time_to_scale.png" class="w-[950px] mx-auto" />



---

<!-- 
# За пределами
* разворачивание кубера
* docker registry
* настройка доступов
* обучение работы с k8s/argo
* настройка соединения LLM <-> k8s
* и многое другое
~1 месяц рабочего времени

 ---
-->

### Первые острые углы

#### Большой маштаб делает систему непрозрачной

<img src="/materials/why_monitoring.png" class="w-[850px] mx-auto" />

---

### Большой маштаб делает систему непрозрачной
#### Решение: Мониторинг

<img src="/materials/monitoring.png" class="w-[850px] mx-auto" />


---

### Много задач == проблемы с отправкой 

<img src="/materials/laptop_submit_problem.png" class="w-[850px] mx-auto" />

<!-- У argo есть клевая фишка - batching
* но у API есть лимиты
* нельзя послать 10к задач сразу
* сделали посылку небольшими батчами
* несколько часов ждать у ноутбука 
Посылка небольшими батчами имеет свою особенность
* она ~~дико тормозная~~ медленная
* посылка тысяч манифестов, где каждый манифест несколько задач содержит, занимает часы
* нельзя выключать ноут, чувствительно к проблемам с сетью  -->

---

### Много задач == проблемы с отправкой 
#### Решение: Нода-оркестратор

<img src="/materials/stage4v2.png" class="w-[650px] mx-auto" />


---

### Зависимость от внешнего Docker сервиса
<img src="/materials/network_limits.png" class="w-[850px] mx-auto" />


---

### Решение: Свое Docker Registry с кэшированием

<img src="/materials/docker_cache.png" class="w-[850px] mx-auto" />

---

### TLDR: Минимальная инфраструктура мечты

<img src="/materials/final_arch.png" class="w-[850px] mx-auto" />

---

### Теперь о пайплайнах 

<img src="/materials/pipelinesv2.png" class="w-[850px] mx-auto" />

---

### Базовый элемент пайплайнов - задача

<img src="/materials/one_task.png" class="w-[850px] mx-auto" />

---

### Базовый элемент решения задачи
#### Межконтейнерное взаимодействие агента и репозитория

<img src="/materials/agent_mcp3.png" class="w-[850px] mx-auto" />

---

### Все ли было легко? Определенно нет

<img src="/materials/break_me.png" class="w-[850px] mx-auto" />

---

### С чем столкнулись в процессе разработки?

<img src="/materials/problems2.png" class="w-[850px] mx-auto" />

---

### Где живут проблемы?

<img src="/materials/problems.png" class="w-[750px] mx-auto" />

> неожиданный "сюрприз" - 100% проблем с лимитами по железу из-за дисков и сети
---

### TLDR: Сотни агентов

* Построили минимальную инфраструктуры мечты
* Перевели пайплайны генерации и обмеров с агентами на K8s
* Держим 150-300 агентов работающих в кластере постоянно
* Обмерили десятки моделей на десятках тысяч задач
* Больше 100к задач прорешали агенты только за последнюю неделю

> всего за 4 месяца

---
layout: image-right
image: /materials/dream.png
backgroundSize: contain
---

#### Тысячи агентов 


1. 1 агент
2. Данные и обмеры
3. 100 агентов
4. **Планы на 1000 агентов**


---

### Тысячи агентов

<table>
<thead>
<tr>
  <th style="width: 25%">Направление</th>
  <th>Что делаем</th>
</tr>
</thead>
<tbody>
<tr>
  <td><strong>Больше масштаб</strong></td>
  <td>
    • пересобрать кластер под наши нагрузки<br/>
    • докидывать ресурсы в кластер
  </td>
</tr>
<tr>
  <td><span style="visibility: hidden"><strong>Больше задач</strong></span></td>
  <td><span style="visibility: hidden">
    • раскручивать историю с билд-агентами<br/>
    • больше парсить GitHub<br/>
    • синтетические задачи (LLM + процедурные)<br/>
    • больше реальных задач
  </span></td>
</tr>
<tr>
  <td><span style="visibility: hidden"><strong>Больше пайплайнов</strong></span></td>
  <td><span style="visibility: hidden">
    • более сложные пайплайны для решения задач<br/>
    • больше агентных фреймворков<br/>
    • online RL
  </span></td>
</tr>
</tbody>
</table>

---

### Тысячи агентов

<table>
<thead>
<tr>
  <th style="width: 25%">Направление</th>
  <th>Что делаем</th>
</tr>
</thead>
<tbody>
<tr>
  <td><strong>Больше масштаб</strong></td>
  <td>
    • пересобрать кластер под наши нагрузки<br/>
    • докидывать ресурсы в кластер
  </td>
</tr>
<tr>
  <td><strong>Больше задач</strong></td>
  <td>
    • раскручивать историю с билд-агентами<br/>
    • больше парсить GitHub<br/>
    • синтетические задачи (LLM + процедурные)<br/>
    • больше реальных задач
  </td>
</tr>
<tr>
  <td><span style="visibility: hidden"><strong>Больше пайплайнов</strong></span></td>
  <td><span style="visibility: hidden">
    • более сложные пайплайны для решения задач<br/>
    • больше агентных фреймворков<br/>
    • online RL
  </span></td>
</tr>
</tbody>
</table>

---

### Тысячи агентов

<table>
<thead>
<tr>
  <th style="width: 25%">Направление</th>
  <th>Что делаем</th>
</tr>
</thead>
<tbody>
<tr>
  <td><strong>Больше масштаб</strong></td>
  <td>
    • пересобрать кластер под наши нагрузки<br/>
    • докидывать ресурсы в кластер
  </td>
</tr>
<tr>
  <td><strong>Больше задач</strong></td>
  <td>
    • раскручивать историю с билд-агентами<br/>
    • больше парсить GitHub<br/>
    • синтетические задачи (LLM + процедурные)<br/>
    • больше реальных задач
  </td>
</tr>
<tr>
  <td><strong>Больше пайплайнов</strong></td>
  <td>
    • более сложные пайплайны для решения задач<br/>
    • больше агентных фреймворков<br/>
    • online RL
  </td>
</tr>
</tbody>
</table>

---

### Online RL

* каждую задачу надо решать несколько раз
* чтобы получить батч для обучения нужно 128 (задач) * 8 (решений) 
  * т.е. нужно минимум 1024 агента бегущих параллельно

<img src="/materials/RL_pipeline.png" class="w-[900px] mx-auto" />

---

### Спасибо за внимание!

<div class="grid grid-cols-2 gap-8 items-center mt-12">

<div class="text-center">

#### Telegram

<div class="text-3xl mt-4">@codegorka</div>

</div>

<div class="text-center">

#### QR на презентацию

<img src="/materials/qr-code.png" class="w-[260px] mx-auto mt-2" />

</div>

</div>

---

### Бонус: дашборды кастомные под себя
<img src="/materials/agent_dashboard.png" class="w-[850px] mx-auto" />
<!-- 
сделали под себя кастомные дашборды - сразу видно pass rate, сколько бежало агентов и тд -->

---

### Распределение длительности работы агентов
<img src="/materials/agent_time_distribution.png" class="w-[850px] mx-auto" />
 
 > оно подсвечивает, если что-то идет не так

---

### Время ответа LLM
<img src="/materials/vllm_proxy_monitoring.png" class="w-[850px] mx-auto" />

> подсвечивает нагрузку на GPU сервера

<!-- TODO новая схема работы  -->

---

### Спасибо за внимание!

<div class="grid grid-cols-2 gap-8 items-center mt-12">

<div class="text-center">

#### Telegram

<div class="text-3xl mt-4">@codegorka</div>

</div>

<div class="text-center">

#### QR на презентацию

<img src="/materials/qr-code.png" class="w-[260px] mx-auto mt-2" />

</div>

</div>