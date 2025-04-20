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
