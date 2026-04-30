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

<!--
всем привет, меня зовут егор

сегодня я расскажу про наш опыт построения агентостроительного завода на пути к тысячам агентом, заодно разные интересные нюнасы встретившиеся в процессе разработки
-->

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

<!--
уверен, что большинство из вас использует ллмки в своей работе. 
Также уверен, что у вас были размышления - есть какие-то данные, какое-то обучение - бабах происходит какая-то магия и получается ллмка, которая отлично пишет код. 

я сегодня как раз приоткрою завесу над частью магии при подготовке хороших кодовых моделей.

qr код для презентации еще в конце слайдов будет, обратите внимание на номер слайдов внизу - если захотите задать вопрос потом про конкретный слайд
-->

---
layout: image-right
image: /materials/egor_image.png
---

### Кто я?
* Работаю в GigaCode в Сбере
* Работал в Huawei - занимался кодовыми моделями
* Работал в JetBrains
* Работал в стартапе, занимавшемся ML4Code еще до того, как это стало модным в 2017~2019 годах

<!--
я начал заниматься машинным обучением на исходном коде, еще до того, как это стало супер модным сейчас - в 2017 году в стартапе. 

Тогда стартап не взлетел и разорился, но его новая итерация подняла полмиллиарда долларов финансирования на новом хайпе. 

Почти весь мой трудовой опыт связан с разработкой инструментов для работы с кодом, анализа и машинного обучения на коде в JB, Huawei и сбербанке
-->

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

<!--
в России java является лидером по популярности, поэтому с сбербанке наши продукты целятся в java - ide & plugin'ы делаются в первую очередь с прицелом на них.

Также мы в исследовательском подразделении много времени уделяем java на всех стадиях от подготовоки данных до обмеров и обучения
-->

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

<!--
План сегодняшнего выступления условно на 4 части разделен

Расскажу вводные про 1 кодового агента, тк это база для следующих частей

Потом расскажу про подготовку данных и обмеры моделей, заодно упомяну прошлую итерацию инфраструктуры, которая дала много идей, как текущую итерацию инфраструктуры стоит делать

Про текущую инфраструктуру и про дальнейшие планы
-->

---

### Почему 1000 агентов?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">10</th>
      <th style="width: 25%;">100</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Генерация 100к решений агентных задач</b></td>
      <td>~100 дней</td>
      <td>~10 дней</td>
      <td>~дни</td>
    </tr>
    <tr>
      <td><span style="visibility: hidden"><b>Обучение</b></span></td>
      <td><span style="visibility: hidden">7 дней</span></td>
      <td><span style="visibility: hidden">7 дней</span></td>
      <td><span style="visibility: hidden">7 дней</span></td>
    </tr>
    <tr>
      <td><span style="visibility: hidden"><b>Обмеры</b></span></td>
      <td><span style="visibility: hidden">~1 день</span></td>
      <td><span style="visibility: hidden">~2 часа</span></td>
      <td><span style="visibility: hidden">~2 часа</span></td>
    </tr>
    <tr style="background-color: #f9f9f9; font-weight: bold;">
      <td><span style="visibility: hidden">Суммарно</span></td>
      <td><span style="visibility: hidden">~107 дней</span></td>
      <td><span style="visibility: hidden">~17 дней</span></td>
      <td><span style="visibility: hidden">~9 дней</span></td>
    </tr>
    <tr style="background-color: #eef; font-weight: bold;">
      <td><span style="visibility: hidden">Ускорение</span></td>
      <td><span style="visibility: hidden">1</span></td>
      <td><span style="visibility: hidden">~6×</span></td>
      <td><span style="visibility: hidden">~11×</span></td>
    </tr>
  </tbody>
</table>

<br>

<div style="visibility: hidden">

**Зачем:** За то же самое время можно проверить в ~10 раз больше гипотез.
> **СПОЙЛЕР:** лучшая модель у того, у кого быстрее итерации

</div>

<!--
Как оценить влияние количества агентов? через влияние на длительность различных этапов в нашем цикле подготовки модели. Если с 20 агентами мы будем решать 100к задач больше 3х месяцев, то с 1000 агентов это должно будет занимать дни
-->

---

### Зачем масштабировать?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">10</th>
      <th style="width: 25%;">100</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Генерация 100к решений агентных задач</b></td>
      <td>~100 дней</td>
      <td>~10 дней</td>
      <td>~дни</td>
    </tr>
    <tr>
      <td><b>Обучение</b></td>
      <td>7 дней</td>
      <td>7 дней</td>
      <td>7 дней</td>
    </tr>
    <tr>
      <td><span style="visibility: hidden"><b>Обмеры</b></span></td>
      <td><span style="visibility: hidden">~1 день</span></td>
      <td><span style="visibility: hidden">~2 часа</span></td>
      <td><span style="visibility: hidden">~2 часа</span></td>
    </tr>
    <tr style="background-color: #f9f9f9; font-weight: bold;">
      <td><span style="visibility: hidden">Суммарно</span></td>
      <td><span style="visibility: hidden">~107 дней</span></td>
      <td><span style="visibility: hidden">~17 дней</span></td>
      <td><span style="visibility: hidden">~9 дней</span></td>
    </tr>
    <tr style="background-color: #eef; font-weight: bold;">
      <td><span style="visibility: hidden">Ускорение</span></td>
      <td><span style="visibility: hidden">1</span></td>
      <td><span style="visibility: hidden">~6×</span></td>
      <td><span style="visibility: hidden">~11×</span></td>
    </tr>
  </tbody>
</table>

<br>

<div style="visibility: hidden">

**Зачем:** За то же самое время можно проверить в ~10 раз больше гипотез.
> **СПОЙЛЕР:** лучшая модель у того, у кого быстрее итерации

</div>

<!--
На скорость обучения количество агентов не влияет - 

тут конечно всякие вариации могут быть из-за размеров модели, 

но 7 дней довольно взвешенная оценка
-->

---

### Зачем масштабировать?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">10</th>
      <th style="width: 25%;">100</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Генерация 100к решений агентных задач</b></td>
      <td>~100 дней</td>
      <td>~10 дней</td>
      <td>~дни</td>
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
      <td><span style="visibility: hidden">Суммарно</span></td>
      <td><span style="visibility: hidden">~107 дней</span></td>
      <td><span style="visibility: hidden">~17 дней</span></td>
      <td><span style="visibility: hidden">~9 дней</span></td>
    </tr>
    <tr style="background-color: #eef; font-weight: bold;">
      <td><span style="visibility: hidden">Ускорение</span></td>
      <td><span style="visibility: hidden">1</span></td>
      <td><span style="visibility: hidden">~6×</span></td>
      <td><span style="visibility: hidden">~11×</span></td>
    </tr>
  </tbody>
</table>

<br>

<div style="visibility: hidden">

**Зачем:** За то же самое время можно проверить в ~10 раз больше гипотез.
> **СПОЙЛЕР:** лучшая модель у того, у кого быстрее итерации

</div>

<!--
на обмер 1 модели количество агентов влияет, но сами обмеры довольно маленькая часть по длительности

с другой стороны в процессе обучения у нас десятки и сотни промежуточных моделей получаются, и здесь ускорение позволяет лучше мониторить все ли идет хорошо

почему падение по времени от 200 к 1000 агентов не такое существенное - тк есть ряд длинных задач, где агент может работать в районе часа или двух
-->

---

### Зачем масштабировать?

<table style="width: 100%; table-layout: fixed;">
  <thead>
    <tr>
      <th style="width: 25%;">Параллельно бегущие агенты</th>
      <th style="width: 25%;">10</th>
      <th style="width: 25%;">100</th>
      <th style="width: 25%;">1&nbsp;000</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Генерация 100к решений агентных задач</b></td>
      <td>~100 дней</td>
      <td>~10 дней</td>
      <td>~дни</td>
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
> **СПОЙЛЕР:** лучшая модель у того, у кого быстрее итерации

<!--
как мы видим - ускорение процесса сбора данных может ускорить на порядок весь цикл. А что нам это дает? Что за тот же самый календарный год можно будет проверить гораздо больше гипотез, что дает возможность найти более хороший общий рецепт для модели и получить лучше качество
-->

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

<!--
так, хорошо, идем теперь к части доклада про одного кодового агента, какие тут есть нюансы, как они влияют на все
-->

---

### Особенность кодовой области для агентов

В отличие от математики, текста, изображений:

- **Риски**: запущенный код на компьютере с неограниченными правами == потенциальные проблемы.
- **Нужна изоляция**: каждая SWE задача == свой контейнер с окружением.
- **Автоматическая верификация**: автоматически проверяем тесты проходят или нет.
  - это путь к масштабированию за счет которого, LLM показывают выдающиеся результаты.

<!--
как вы может слышали - remote code execution - это одна из самых критичных уязвимостей которая может быть

а в кодовой области мы буквально даем агенту писать код и запускать его - поэтому надо обязательно подумать над изоляцией, чтобы агент не набедокурил. 

Также как например и в математике код позволяет автоматически верифицировать решение через тесты - что открывает дорогу к масштабированию
-->

---

### Особенность кодовой области для агентов

<img src="/materials/meta_safety.png" class="h-[450px] mx-auto" />

<!--
про риски хочется вспомнить забавный пример, как директор по безопасности в мете дала доступ агенту - и наблюдала, как он удаляет всю ее переписку в почте
-->

---

### Анатомия запуска SWE задачи - три фазы задачи

1. **Подготовка**: клонировать репозиторий, установить зависимости
2. **Решение**: запускается LLM + агентный фреймворк + изолированное окружение -> и агент вносит изменения
3. **Проверка**: запуск тестов, сбор результатов

> грубо говоря, один агент работает над кодовой задачей в течение 5~20 минут

<!--
запуск каждой задачи разделяется довольно логично на 3 части

- подготовка задачи - клонировать репу, установить все - в идеале это делается один раз при подготовке образа для задачи
- решение задачи, когда агент что-то делает и пытается решить проблему 
- дальше проверка - берутся изменения агента, применяются к чистому репозиторию, запускаются тесты, проверяется, что проблема пропала
-->

---

### Агентный фремворк
#### Упрощенный взгляд

<div class="grid grid-cols-[2fr_3fr] gap-8 items-center">

<div>

- Получает задачу: «исправь баг» / «добавь фичу»
- Агентный цикл
- Набор инструментов (чтение файлов, редактирование, bash)
- Окружение


> LLM вне агентного фремворка

</div>

<div class="flex items-center justify-center">

<img src="/materials/simple_agent_2.png" class="max-w-full max-h-[640px]" />

</div>

</div>

<!--
упрощенный взгляд на агентный фремворк выглядит так

агент получает какую-то задачу
посылает сообщение в бэкенд с ллмкой, получает сообщение с инструментами дял вызова, вызвает их, получает обратную связь от среды, посылает в бэкенд

ну и работает так до того, как не достигнет лимита по длине контекста, количеству шагов или вызова инструмента для остановки агентного цикла
-->

---

### Агентный фреймворк Claude Code

<img src="/materials/claude_pirate.png" class="h-[450px] mx-auto" />

<!--
давайте теперь посмотрим на продукт, с которым многие из вас знакомы - claude code - спасибо за такую возможность неожиданному облачному бэкапу от этих ребят
-->

---

### Claude Сode
<!-- note: тут добавляем подробности-->
<img src="/materials/simple_agent.png" class="h-[450px] mx-auto" />

<!--
как можно заметить, появилось несколько новых подсистем
* система разрешений, взаимосвязанная с инструментами
* различные интерфейсы
* система для хранения состояния агента
-->

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

<!--
если рассмотреть еще под лупой, то каждая из систем декомпозируется еще на ряд подсистем

давайте их рассмотрим одну за другой
-->

---

### Claude Code под 🔍



<img src="/materials/not_simple_agent_1.png" alt="Surface Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

<!--
интерфейсы или клиентская поверхность

тут видно, что есть целая россыпь возможностей взаимоедйствия от различныъ вариантов CLI до интеграций с приложениями типа IDE/браузер или SDK 

все они взаимодействуют с агентным циклом
-->

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_2.png" alt="Core Layer + State Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

<!--
агентный цикл я подсвечу вместе с памятью
тут сразу есть важные нюансы - пайплайн сжатия контекста - без него агенты умирали бы на относительно длинных диалогах и задачах

и многослойная память в которой есть систем промпты, состояние диалога (чтобы можно было вернуться к нему позже), файлы с заметками типа `CLAUDE.md` и суммаризация работы субагентов
-->

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_3.png" alt="Safety / Action Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

<!--
следующая система - инструменты и система разрешений
по сути это место, где расширяется функционал агентного фремворка, адаптируется под конкретные проекты - plugin'ы, MCP инструменты, скиллы, хуки - это в дополнении к больше чем полусотне встроенных инструментов

и все это тесно взаимосвязано с системой разрешений - что можно делать, что нельзя + если присмотреться, то есть даже автоматический классификатор для определения опасности команды на основе ллмок - чтобы попытаться, например, предотвратить неожиданное удаление почты, как мы видели раньше
-->

---

### Claude Code под 🔍

<img src="/materials/not_simple_agent_4.png" alt="Backend Layer" class="max-h-[70vh] w-auto max-w-full mx-auto object-contain rounded-lg shadow" />

<!--
ну и последнее, но не по важности, возможность взаимодействовать, как с локальной средой, так и с удаленными средами в облаках бесшовно или другими внешними системами типа поисковых движков
-->

---

### Оценка трудозатрат

<img src="/materials/top_vs_dyi_agent.png" class="h-[450px] mx-auto" />

<!--
какие выводы можно сделать еще из этой информации?

что простой агентный фремворк можно быстро и дешево написать

но сделать популярный продукт для разработчиков может потребовать усилий десятков разработчиков на протяжении многих месяцев
-->

---

### TLDR - 1 агент

мы узнали
* зачем нужна изоляция для задачи 
* зачем нужна автоверификация решения задачи
* как устроен агентный фреймворк

<!--
давайте подведем черту под этой секцией доклада

изоляция нужна, чтобы минимизировать вероятность проблем

наличие тестов и линтеров дает возможность масштабирования без ручного труда

ну и что агентный фремворк можно написать как в нескольких файлах за несколько дней, так и может потребоваться месяцы работы команды инженеров
-->

---
layout: image-right
image: /materials/10_agents.png
backgroundSize: contain
---

### Данные и обмеры

1. 1 агент
2. **Данные и обмеры**
    * сбор задач для агентов
    * обмеры моделей
    * данные для обучения
    * инфраструктура
3. 100 агентов
4. Планы на 1000 агентов

<!-- ---
В этой секции я раскрою еще несколько важных частей для получения хорошей кодовой модели

Сбор задач для агентов становится большой задачей, тк надо покрыть все домены, языки, типы задач

как измерять прогресс, как получать данные для обучения - ну и нашу прошлую итерацию инфраструктуры
 -->

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

<!--
начнем со сбора задач - есть два подхода - сбор реальных и синтетических задач

для реальных задач все начинается с парсинга гитхаба на котором уже больше 200 миллионов репозитоиев
ищем PR и связанные issues - в одном решение, в другом постановка проблемы
-->

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

<!--
когда нашли кандидата - то следующий шаг в воронке отбора - проверить, что есть тесты в PR которые тестируют описанную проблему в issues
-->

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

<!--
если есть тесты - идем на следующий этап - бидим образ для задачаи - либо по правилам, либо при помощи билд агента
-->

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

<!--
и финальная верификация задачи - должен быть патч с решением из PR, до него тесты какие-то должны падать (это показатель, что есть проблема), после применения решения - все должно проходить.

Также фиксируются регрессионные тесты - они должны проходить и там, и там - проверка, что решение не добавило новых проблем в других местах
-->

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

<!--
давайте теперь посмотрим на примере датасета swe rebench v2 как сильно влияют различные шаги при фильтрации - начинаем с почти 30 миллионов PR в 150к реп
-->

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

<!--
фильрация по тестам порядка 70% PR и 30% реп
-->

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

<!--
наличие связи с описанной проблемой уменьшает количество PR в 10 раз, а реп в 2 раза
-->

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

<!--
фильтрация по количеству звезд, лицензий еще снижает количество PR  и реп
-->

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

<!--
дальше билдятся задачи - этот шаг срезает еще в 15 раз количество PR'ов и в 5 раз количество репозиториев
-->

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

<!--
и последний этап фильтрации - попросить ллмку отфильтровать совсем уж мусорные описания проблем

и такая воронка дает нам на выходе по сути одну тысячную от входных PR, одну пятидесятую для реп

в среднем получаем 9 задач на репозиторий

давайте теперь посмотрим на альтернативный подход с синтетическими задачами
-->

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

<!--
он начинается также - начинаем парсить гитхаб по определенным правилам типа фильтры по языкам, популярности, лицензиям
-->

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

<!--
следующим шагом проверяем наличие тестов - клонируем репу для этого
-->

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

<!--
билдим образ - тоже либо по правилам, либо билд агентом
-->

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

<!--
здесь отличие с реальными задачами - базовый образ от которого будем делать задачи должен быть абсолютно рабочим
-->

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

<!--
и дальше начинается бльшой спектр возможностей, как подготовить задачу - от внесения процедурных изменений на основе модификаций абстрактных синтаксических деревьев, до различной логики с ллмками для повреждения кода
Но и- повреждений мы должны получить reverse patch - чтобы привести репозиторий в рабочее состояние + повреждения должны быть покрыты тестами
на выходе должны получить образ под конкретную задачу со всем необходимым как и для реальной задачи
-->

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

### Процедурная AST-трансформация

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

<!--
пример как можно сгенерировать задачу - в огромной репе в какой-то функции повредить немного ее, чтобы в каких-то важных условиях функция начинала работать некорректно

здесь конкретно выкинули условие на равенство всех элементов, что приведет к делению на ноль
-->

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

<!--
давайте теперь рассмотрим воронку подготовки задач на примере датасета SWE-smith
у них 3 стратегии генерации задач
* процедурная - несколько эвристик по внесению повреждения кода, 40% успешных кандидатов получается
* LLM - например что-то повредить или написать задачу по PR с тестами - тут уже 48% успеха
* комбинирование - берутся успешные кандидаты двух предыдущих стратегий и совмещаются - тут почти 100% успех из-за этого

суммарно из 129 репозитиев они больше 50к задач получили, или 388 задач на репозиьорий

давайте теперь сравним подходы
-->

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

<!--
сходу можно заметить, что количество задач синтетических на репозиторий существенно больше, чем реальных. Синтетические задачи ограничены наборами стратегий - они могут давать слишком похожие задачи. Когда реальные не ограничены ничем. 

но я бы добавил, что не надо противопоставлять эти подходы их можно и нужно совмещать
-->

---

### Обмер моделей

**Нельзя улучшить то, что не умеешь измерять.** (с)

Задачи собраны — дальше нужно понимать, **что модель реально умеет**: где стало лучше после обучения, где регресс, где упёрлись в потолок.

<!--
переходим к следующему элементу нашего цикла подготовки моделей - обмерам качества. 

Чтобы понять, есть ли прогресс, надо как-то этот прогресс измерять
-->

---

### Автоматический обмер моделей

> на примере SWE-бенчмарков (SWE-MERA, SWE-bench Pro, ...):

1. **Задача**: описание проблемы в issue
2. **Окружение**: Образ (фиксированный commit и зависимости)
3. **Верификация**:
   - Команда для запуска тестов
   - Тесты для проверки проблемы (Fail-to-Pass)  
   - Регрессионные тесты (Pass-to-Pass)

<!--
как выглядит одна задача в случае автоматического обмера моделей - глаза вас не обманывают - она выглядит по сути идентично тем задачам, которые мы рассматривали в предыдущей секции - постановка задачи, образ, алгоритм верификации.

В каждом бенчмарке обычно от 300 до 700 задач, бенчмарки отличаются алгоритмами сбора - по языкам, где-то ручная валидация проблемы
-->

---

### SWE-bench verified
<img src="/materials/SWE-bench.png" class="h-[450px] mx-auto" />

> замеряют связку - модель + агентный фремворк

<!--
давайте рассмотрим один из самых популярных бенчмарков - swe bench verified - можно заметить, что тут в рейтинге идет связка модель + агентный фремворк. Это, кстати, одна из причин, почему многие вкладывают усилия в агентные фремворки - они дают возможность вытянуть еще несколько процентов качества из модели за счет разных трюков
-->

---

### Ручной обмер моделей - LLM Arena
<img src="/materials/llm-arena-code.png" class="h-[450px] mx-auto" />

> Предпочтения людей != выбор правильного ответа

<!--
альтернатива автоматическим методам - ручной обмер - когда человек пишет запрос, получает ответ от 2х скрытых моделей (assistent a и b), выбирает тот, который ему больше нравится

должен заметить, что в отличие от автоматического ответа тут именно предпочтение человека играет роль, человек может отдать выбор неправильному ответу - например который будет поддерживать его точку зрения про плоскую землю
-->

---

### LLM Arena Leaderboard

<img src="/materials/arena.png" class="h-[450px] mx-auto" />

> рейтинг 1572 выигрывает у 1530 в ~56% случаев

> Opus 4.7 (1572) выиграет у gpt-4o (1368)  в ~76% случаев

<!--
ручной обмер основывается на Эло рейтинге, который был придуман для шахмат. Точно также как и в шахматах нет возможности заставить человека все модели сравнить между собой для всех вопросов. Поэтому рейтинг рассчитывается на тех сравнениях что есть - и получается одно число.

Дальше можно сравнить между собой рейтинги, чтобы получить оценку в каком проценте случаев одна модель выиграет у другой. опус 4.7 выиграет у опуса  4.5 внизу таблицы в 56% случаев. А у gpt-4o несколькьми страницами ниже в 76%
-->

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

<!--
давайте подведем черту под этой секцией
я упомянул два пути для обмера модели
* автоматический на основе прохождения тестов или линтеров - он отлично скейлится и воспроизводится
* ручной - на основе предпочтений людей, который позволяет замерять сложно формализуемые вещи типа предпочтений ответов людей. Дорогой, медленный, шумный, но важный чтобы не получить модель аутиста, которая не умеет общаться

добавлю что ручные замеры тоже пытаются автоматизировать через ллмки - это позволяет промежуточные результаты замерять. Но конечную оценку должны все таки люди выдавать
-->

---

### Обмеры моделей ⟷ генерация данных

| Критерий | Авто-обмеры моделей | Генерация данных |
| --- | --- | --- |
| Docker-окружение |✅ | ✅ |
| Работа агента | ✅ | ✅ |
| Тесты / верификация | ✅ | ✅ |
| Результат | Pass rate | Диалоги |

<!-- 

как я уже провел параллель, что задачи для агентов по сути одинаковые для обмеров и генерации решений

также проведу параллель, что в случае кодовых агентов нет разницы между решение задачи для бенчмарка или для сбора тренировочных данных. 

В одном случае мы созраняем диалоги для последующего использования, в другом нам нужно посчитать процент решенных задач

 -->

---

### Прошлая инфраструктура
<img src="/materials/10man_1server.png" class="h-[450px] mx-auto mt-8" />

<!--
так, теперь небольшая подводка к следующей секции - наша прошлая итерация инфраструктуры долго выполняла свою роль. У нас было несколько мощных серверов, докеры и тд. И 10-20 человек в зависимости от задач, которые их активно использовали
-->

---

### Что может пойти не так?
<img src="/materials/bare_problems.png" class="h-[450px] mx-auto mt-8" />

<!--
со временем мы начали упираться во все большое число острых углов
* постоянно заканчивалось место на дисках
* пайплайны конфликтовали за ресурсы - и либо тормозили, когда не хватало цпу, либо умирали, когда память заканчивалаьс
* запуск больших генераций требовал подготовить окружение на каждом из серверов, разделить задачи, запустить пайплайны, те куча ручной рабоыт
* кто-то все таки иногда запускал агентов без изоляции - которые поднимали для тестов какие-то сервисы на 0 - приходили безопасники, прокидывали обучающий градиент
-->

---
layout: image-right
image: /materials/this_is_fine.png
backgroundSize: contain
---

### Что может пойти не так?
TLDR
* Нет масштабирования пайплайнов
* Инфраструктурный ад

<!--
если просуммировать, то все эти проблемы не давали самого главного - масштабироваться

что нас сподвигло пересмотреть нашу архитектуру
-->

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

<!--
так, давайте теперь поговорим про текущую инфраструктуру, которая нам уже позволяет гонять сотни агентов параллельно, которая возникла в результате того, что предыдущая итерация морально устарела - и это заставило нас переосмыслить все
-->

---

### Запланированная инфраструктура
<img src="/materials/stage3.png" class="w-[950px] mx-auto" />

<!--
изначальные наши мысли про текущую инфраструктуры были такими
* нам нужен кубер - автоматическое масштабирование, изоляция из коробки и еще миллион клевых фич
* в нем будут жить наши серваки, которые будут доступны исследователям через единый API
* а вот llm proxy, argo dag'и и s3 я чуть подробнее расскажу на следующих слайдах
-->

---
-->
---

### Зачем нужен сервис LLM proxy?

Раньше
<img src="/materials/before_llm_proxy.png" class="w-[950px] mx-auto" />

<!--
начнем с llm proxy - на прошлой итерации он был по сути не нужен - на одном серваке помещалось несколько десятков агентов, которые сервак с гпушками вполне себе тянул
-->

---

### Зачем нужен сервис LLM proxy?

Сейчас
<img src="/materials/llm_proxy.png" class="w-[950px] mx-auto" />

> load balancing, агент приклеивается к одной LLM для кэширования запросов, подключение/отключение LLM под нагрузкой и т.д.

<!--
но мы масштабируемся - у нас будет большое количество серверов, огромное число агентов

и llm proxy нужен, чтобы скрывать за собой кластер для инференса. 

Также он решает ряд важных моментов - балансировку нагрузки на инференс сервера, перенаправление агента на тот же самый инференс сервер для использования кэшей - ну и всякие ништяки типа добавление или удаление LLM серверов под нагрузкой
-->

---

### Зачем S3?

> одна из главных прошлых проблем - постоянные проблемы с местом на дисках
* S3 - главное хранилище артефактов
  * "бесконечное" хранилище
  * разделяем ответственность с инфраструктурными командами
* диски на серверах
  * для текущих задач только

<!--
одна из главных проблем предыдущей инфраструктуры решается через перенос артефактов различных пайплайнов на S3

диски на серверах только для хранения образов и текущих данных задач
-->

---

### Зачем нам нужен DAG?

<div class="grid grid-cols-[3fr_4fr] gap-8 items-center mt-4">

<div>

<ul>
<li>Мы хотим <strong>честные обмеры</strong> — метрика должна отражать умения модели, а не дыры в логике обмера</li>
<li style="visibility: hidden">Если агент и тесты живут в <strong>одном контейнере</strong> — агент может подсмотреть/подправить тесты и выбить "100%" на бенче</li>
<li style="visibility: hidden">Разделение на шаги (агент → изоляция артефактов → тесты в чистом окружении) убирает эту лазейку</li>
</ul>

</div>

<div>

<img src="/materials/SWE-bench-broken.png" class="max-h-[520px] w-full object-contain" />

</div>

</div>

<!--
откуда появляется потребность в направленном ацикличном графе? Одни ребята заморочились и проверили, можно ли получить высокий процент качества на бенчмарках, не решая задачи -- и выяснили, что если не изолировать нормально агента от логики валидации, то агент может смухлевать
а мы хотим честные обмеры
-->

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

<!--
как это может сделать агент - подправив логику тестов, подменив какие-то бинарники и тд
-->

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

<!--
через разделение работы агента и запуска тестов по различным контейнерам мы можем минимизировать такие проблемы. А эта последовательность шагов по сути и есть направленный ацикличный граф - нам надо какие-то контейнеры вызывать после каких-то других
-->

---

### DAG == Честный обмер

<img src="/materials/eval_pod.png" class="h-[600px] mx-auto" />

<!--
если визуализировать то, что я сказал про честный обмер - то у нас получается такая связка - работает агент, решает задачу в какмо-то репозитории

на следующий этап передается дифф извлеченный из репозитория - при этот этот дифф надо еще очистить от модификаций тестов и тд

дальше запускается чистый репозиторий, применяется чистый дифф, запускаются тесты, получается честная верификация решения
-->

---

### Где найти DAG для K8s?
Отсутствие многошаговых пайплайнов решается через...
<img src="/materials/argo.png" class="w-[950px] mx-auto" />

<!--
ну и очень удачно для кубера уже DAG были сделаны в проекте Argo
-->

---

<img src="/materials/argo2.png" class="w-[950px] mx-auto" />
...искал медь, а нашел золото

<!--
очень удачно - argo имеет кучу дополнительных плюсов для нас 
на уровне DAG'ов - можно выстраивать сложные пайплайны с параллелизмом и условным запуском контейнеров - например ты не знаешь, сколько нужно ресурсов для новой задачи - можно сделать последовательность, где в случае падения предыдущей поды будет запускаться новая с увлеиченным количеством ресурсов

ну и клевые фишки с retry и таймайтами
-->

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

<!--
в общем так мы видели изначальную инфраструктуру

кубер - контейнеризаиця и масштабирование
прокси - масштабирование и оптимизация инференса
argo dag - для честных обмеров
s3 - для хранения артефактов

кажется, что
-->

---

### ...пора!
<img src="/materials/time_to_scale.png" class="w-[950px] mx-auto" />

<!--
кубер, арго, s3, куча докер образов для задач и бенчмарком - разве может что-то пойти не так?

на самом деле вполне себе
-->

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

<!--
у нас куча серверов, сотни задач бегущих параллельно, куча разных сервисов - и когда что-то идет не так - просто невозможно зайти на каждую отдельную задачу и дебажить, особенно когда проблемы возникают именно под нагрузкой
-->

---

### Большой маштаб делает систему непрозрачной
#### Решение: Мониторинг

<img src="/materials/monitoring.png" class="w-[850px] mx-auto" />

<!--
соотвественно первым чем мы обросли - это мониторинг
этой святой троицы минимально достаточно для того чтобы гораздо лучше понимать, что идет не так

прометеус чтобы метрики все возможные собирать
графана - чтобы визуализировать метрики
локи - поисковик по логам из тысяч задач
-->

---

### Много задач == проблемы с отправкой 

<img src="/materials/laptop_submit_problem.png" class="w-[850px] mx-auto" />

<!--
много задач - я имею в виду десятки и сотни тысяч задач
посылка батчами занимает много часов, становится чувствительна к проблемам с сетью, нельзя ноут выключать

в общем крайне неудобно
-->

---

### Много задач == проблемы с отправкой 
#### Решение: Нода-оркестратор

<img src="/materials/stage4v2.png" class="w-[650px] mx-auto" />

<!--
но решение довольно простое - через вспомогательную ноду-оркестратор, куда данные (даже для десятков и сотен тысяч задач это меньше гигабайта) копируются за десятки секунд
а дальше она уже может сколь угодно долго посылать эти задачи в кластер

на схеме эта нода как раз под значком кубер

также подсвечу что сбоку мониторинг на схеме появился
-->

---

### Зависимость от внешней сети
<img src="/materials/network_limits.png" class="w-[850px] mx-auto" />

<!--
еще одна проблема - при запуске пайплайнов с кучей задач мы буквально сразу упирались во входной канал с внешней сетью

на графике видно, как мы запускали и останавливали эксперименты - разгон до 100% мгновенный
-->

---

### Решение: Свое Docker Registry с кэшированием

<img src="/materials/docker_cache.png" class="w-[850px] mx-auto" />

<!--
решение тоже довольно прямолинейное - кэширующий docker registry 
из внешней сети образы лишь 1 раз скачиваются

а внутренняя сеть гораздо большими лимитами обладает

какие-то замедления могут быть лишь при первом запуске задачи - начиная со второго проблема исчезает
-->

---

### TLDR: Минимальная инфраструктура мечты

<img src="/materials/final_arch.png" class="w-[850px] mx-auto" />

<!--
ну и таким эволюционным путем мы получили текущую инфраструктуру
* кубер для изоляции и масштабирования
* даги для честных решений задач
* проксирование ллмок для балансировки нагрузки и повышения эффективной утилизации гпушек
* docker registry убрал проблемы с внешней сетью
* а мониторинг позволяет быстро отдеажить появляющиеся проблемы
-->

---

### Теперь о пайплайнах 

<img src="/materials/pipelinesv2.png" class="w-[850px] mx-auto" />

<!--
но инфраструктура нужна для чего-то

давайте поговорим про пайплайны, которые мы запускаем
* начну с пайплайнов для скраппинга гитхаба - запускается пачка контейнеров с токенами от гитзаба, чтобы они постоянно искали кандидатов для задач
* дальше надо подготовить образы для задач - у нас есть куча правил для разных языков + билд агент
* дальше эти задачи идут на 2 цели
* постоянно обновляющиеся внутренние бенчмарки под эгидой SWE-MERA - в дополнении к общепринятым бенчмаркам
* и на генерацию задач - в генерации задач мы как внешние задачи используем, так и собранные нами
-->

---

### Базовый элемент пайплайнов - задача

<img src="/materials/one_task.png" class="w-[850px] mx-auto" />

<!--
теперь рассмотрим повторяющийся элемент ряда пайплайнов
в DAGе 3 шага
* подготовка репозитория - извлечение необходимой информации из контейнера с задачей
* дальше запускается под с агентом, отрабатывает
* запускается под для проверки решения

все необходимые артефакты сохраняются на S3 с каждого из подов
-->

---

### Базовый элемент решения задачи
#### Межконтейнерное взаимодействие агента и репозитория

<img src="/materials/agent_mcp3.png" class="w-[850px] mx-auto" />

<!--
межконтейнерное взаимодействие у агента мимикрирует под запуск агента как на обычной машине
* у агентов есть инструменты дял работы с файловой системой - через примонтированный диск он может спокойно модифицировать репозиторий
* агент может запускать команды в контейнере с репой через подложенный бинарник который работает как MCP сервер

те полноценный доступ у агента есть и к коду репозитрия, и к запуску команд башевских
-->

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
* Держим >100 агентов работающих в кластере постоянно
* Обмерили десятки чекпоинтов моделей на десятках тысяч задач
* Больше 100к задач прорешали агенты только за последнюю неделю

> всего за 4 месяца

<img src="/materials/heartbeat.png" class="w-[850px] mx-auto mt-4" />

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

> подсвечивает нагрузку на GPU сервера - здесь перегруз виден

---

### Время ответа LLM
<img src="/materials/vllm_proxy_monitoring_2.png" class="w-[850px] mx-auto" />

> подсвечивает нагрузку на GPU сервера - здесь все отлично

<!-- TODO новая схема работы  -->

---

### Повышение эффективной утилизации GPU


<img src="/materials/eff_util.png" class="w-[900px] mx-auto" />

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
