# Up2you
Up2you – a smartphone detox service where you set your off-screen hours and place a stake on your willpower. If you exceed your allowed phone time, you lose the stake. If you succeed, you get it back. 

It’s simple: **your focus, your rules, your call.**




### Первая неделя (дни 0 – 7) ― **“Трекинг & База”**  
Цель — получить доказательство, что мы умеем точно и энергоэффективно считать экран‑тайм и держать код/инфру под контролем. Итоговый артефакт: `v0.1‑alpha`.

| День | Поток | Конкретные задачи | Владелец | Критерий «готово» |
|------|-------|-------------------|----------|-------------------|
| **День 0**<br>(пятница до старта) | PM / Все | • Kick‑off, согласование OKR<br>• Создать борд (Jira/Linear), расписать issue по дням | PM | Борд заполнен, задачи оценены (t‑shirt) |
| **День 1** | **Android** | 1. Создать репо + Android Studio project (minSdk 21).<br>2. Экран «Запрос Usage Access» + пояснение.<br>3. Добавить модуль `event-collector` (Service + WorkManager 60 с). | Lead Android | *Runtime‑разрешение запрошено; Service запускается автоматически* |
| | **DevOps** | 1. Подключить GitHub Actions (assembleDebug).<br>2. Настроить CI‑lint (detekt, ktlint). | DevOps | Pipeline “green” при merge → `main` |
| **День 2** | **Android** | 1. Реализовать выборку `queryEvents(begin,end)`; фильтровать `ACTIVITY_RESUMED/PAUSED`.<br>2. Создать Room‑таблицы `usage_events`, `checkpoint`. | Android A | *Лог рассчитанного screen‑time выводится в Logcat; ±5 с (ручная проверка)* |
| | **QA** | 1. Подготовить тест‑скрипт «открываю YouTube 15 мин». | QA | Скрипт загружен в TestRail |
| **День 3** | **Android** | 1. Добавить periodic Work (tag “export”) — выгрузка JSON‑файла в /Download.<br>2. Профайлить батарею (Android Profiler) на 2 часовом прогоне. | Android B | *CPU < 2 %, Battery < 0.5 % / ч (3 раза подряд)* |
| | **Backend (Go)** | 1. Инициализировать Go‑модуль `api-gateway` (gin).<br>2. Эндпоинт `POST /device/register` (returns deviceId). | Go dev | cURL показывает `201 Created` |
| **День 4** | **Android** | 1. Прототип «Strict mode»: AccessibilityService ловит `TYPE_WINDOW_STATE_CHANGED`.<br>2. Фича‑flag в SharedPreferences. | Android A | *При включённом флаге события приходят ≥ 1 / сек* |
| | **UX** | 1. Написать первый черновик privacy‑copy «почему нужны разрешения». | PM/UX | Текст утверждён Legal |
| **День 5** | **Android ↔ Go** | 1. Регистрация устройства в бекенде.<br>2. Отправка списка событий (`gzip json`) в `/events/batch` раз в 3 ч. | Android B + Go dev | *Логи видны в PostgreSQL `usage_events`* |
| | **DevOps** | 1. Helm chart для API; развернуть на kind‑cluster.<br>2. Метрики Prometheus (requests/sec,  p95). | DevOps | Grafana dashboard открывается; latency < 100 мс |
| **День 6** | **Dog‑food** | 1. Вся команда ставит `v0.1‑SNAPSHOT` на личный телефон (3 бренда).<br>2. Запускаем 12‑часовой «детокс‑окно» тест. | Все + QA | *На всех устройствах есть валидный лог; батарея < 6 %* |
| **День 7** | **Ревью & Tag** | 1. Code‑review outstanding PRs.<br>2. Retrospective «что пошло/что нет».<br>3. Создать Git‑tag `v0.1-alpha`, push в Firebase App Distribution (internal). | Lead Android + PM | Tag опубликован; релиз‑ноты; backlog обновлён |

---

#### Распределение времени внутри дня (пример)

* **08:30 – 09:00** Daily Stand‑up (15 мин) + «Battery health» отчёт (15 мин).  
* **09:00 – 12:30** Фокус‑работа (coding).  
* **13:30 – 16:00** Фокус‑работа + unit‑тесты.  
* **16:00 – 17:00** Pair‑review / QA sync / метрики.  
* **17:00 – 17:30** Документация (Confluence) & update Jira.

---

#### Риски первой недели и буферы

| Риск | Вероятность | Буфер | План «B» |
|------|-------------|-------|----------|
| Запрос Usage Access не показывается на некоторых OEM | средняя | +0.5 дня | Инструкция вручную через `Settings.ACTION_USAGE_ACCESS_SETTINGS` |
| Батарея > 0.5 % / ч | средняя | +0.5 дня | Увеличить интервал WorkManager до 90 с, включить JobScheduler |
| CI fail из‑за SigningConfig | низкая | +0.2 дня | Отложить release build; debug‑кейт до фикса |

---

#### Чек‑лист «Definition of Done» для `v0.1‑alpha`

* Экран‑тайм считается локально, точность ≤ ±5 с.  
* Разрешение **Usage Access** выдаётся через онбординг.  
* Battery‑impact ≤ 0.5 % / ч (2‑часовой сценарий YouTube).  
* События выгружаются на стенд‑бекенд; таблица `usage_events` заполнена.  
* CI / CD pipeline зелёный; tag раскатан на internal distribution.  

При выполнении всех пунктов команда переходит к спринту № 2 («Гибрид‑точность и депозиты»).



# 
#


### Первая неделя — **подробный чек‑лист задач для PM**

> Цель PM: обеспечить фокус команды на результате `v0.1‑alpha`, снять блокеры, держать прозрачность статуса и рисков.

| День | PM‑задачи | Конкретный результат / артефакт |
|------|-----------|---------------------------------|
| **День 0 (Kick‑off)** | 1. Провести встречу OKR (60 мин)<br>2. Создать доску (Jira/Linear): эпик *Android‑MVP*, спринт “Week 1”<br>3. Разбить backlog MVP на user‑story → task (granулярность ≤ 1 дня)<br>4. Подготовить «Working‑Agreement» (daily time, code‑review SLA, definition‑of‑done)<br>5. Занести ключевые риски в Risk Register (Confluence) | • Протокол Kick‑off<br>• Jira‑спринт с оценкой story‑points<br>• Документ Working‑Agreement |
| **День 1** | 1. Модерировать Daily Stand‑up (15 мин) + **battery‑metric check‑in** (5 мин)<br>2. Финализировать PRD‑секцию «Tracking Logic» (описание бизнес‑правила ≥ 20 мин ⇒ штраф, допущения, доп. буфер 30 сек)<br>3. Согласовать макет экрана разрешения (с UX) — текст, скрин<br>4. Обновить Roadmap MVP в Miro (видимость для стейкхолдеров) | • Версия 1 PRD загружена в Confluence<br>• Approved copy разрешения |
| **День 2** | 1. Daily Stand‑up ➜ фиксировать блокеры в Jira label `impediment`<br>2. Собрать первые цифры «точность + battery» от Android‑dev, занести в sheet *Metrics‑Week1*<br>3. Проверить статус CI‑pipeline (pull‑requests pending?) — эскалировать владельцам<br>4. Обновить Risk Register: «OEM UsageAccess dialog не показывается» → mitigation plan | • Таблица *Metrics‑Week1* (точность, CPU, батарея)<br>• Risk Register v0.2 |
| **День 3** | 1. Facilitate mid‑week Sync (30 мин): Dev + QA + Backend (хэнд‑офф batch‑endpoint)<br>2. Создать шаблон Release Notes `v0.1‑alpha`<br>3. Уточнить у Legal статус текста Privacy Policy (rus/eng)<br>4. Дополнить PRD разделом «Логи & экспорт» (JSON‑формат) | • Minutes mid‑week Sync<br>• Draft Release Notes |
| **День 4** | 1. Daily Stand‑up → фокус на подготовке dog‑food<br>2. В Miro‑UX flow отметить место для “Strict mode” переключателя; проверить, нет ли конфликтов с Accessibility policy.<br>3. Подготовить FAQ (5 Q/A) для внутреннего теста (как выдать Usage Access и т.д.) | • FAQ.md в репо<br>• UX flow обновлён |
| **День 5** | 1. Организовать Dog‑food test («12‑часовое окно»): расписка участников, чек‑лист действий, форма обратной связи Google Form.<br>2. Сформировать метрики успеха dog‑food (краши = 0, батарея < 6 %)<br>3. Координировать DevOps: убедиться, что staging API работает и логирует события | • Test plan Dog‑food<br>• Google Form live |
| **День 6** | 1. Мониторить Slack‑канал dog‑food: собирать баг‑репорты, добавлять в Jira label `dogfood`.<br>2. Проверить с Backend, что данные приходят (SQL‑dump на 3 устройства).<br>3. Подготовить черновик Retrospective canvas (Start/Stop/Continue). | • SQL‑снимок с usage_events<br>• Retro‑canvas.docx |
| **День 7** | 1. Провести Sprint‑Review (демо профайла + Room лог).<br>2. Провести Sprint‑Retro (45 мин) и сформировать Action items.<br>3. Согласовать Release Notes + загрузка `v0.1‑alpha` в Firebase App Distribution (internal testers).<br>4. Создать Sprint 2 backlog; расставить приоритеты (Strict mode, Stripe). | • Demo видео‑ссылка<br>• Retro Action items в Jira<br>• Tag `v0.1‑alpha` + Release notes<br>• Sprint 2 дескриптор |

---

#### Непрерывные обязанности PM в течение недели

| Категория | Задачи |
|-----------|--------|
| **Коммуникации** | • Вести дневной *changelog* в Slack<br>• Держать стейкхолдеров (фандер/CTO) в курсе: короткий daily e‑mail «Progress + Risks» |
| **Процесс** | • Следить за WIP‑лимитом (≤ 5 одновременных tasks/engineer)<br>• Гарантировать ≤ 24 ч cycle‑time pull‑requests |
| **Метрики** | • Заполнять d/metrics: точность, battery, краши (Crashlytics).<br>• Обновлять burn‑down chart спринта |
| **Риски** | • Ежедневно пересматривать Risk Register: probability, impact, mitigation owner |
| **Документация** | • Обновлять Confluence: PRD, API‑контракт, UX‑flows |

При выполнении всех пунктов PM закрывает спринт 1 и готовит команду к спринту 2 («Гибрид‑точность и платежи»).
