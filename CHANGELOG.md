# Значимые обновления ROSBIOTECH Studies

Здесь собраны главные этапы развития ROSBIOTECH Studies: новые возможности, крупные архитектурные обновления и важные исправления.

Это человекопонятная история развития проекта, а не полный Git-журнал. Правила ведения истории определяются [`automation@v1/policy/ORGANIZATION_STANDARD.md`](https://github.com/rosbiotech-studies/automation/blob/v1/policy/ORGANIZATION_STANDARD.md).

Для самых крупных обновлений есть ссылка **«Подробнее об обновлении»** — она ведёт на историческую техническую заметку с контекстом, составом изменений, rollout и проверкой результата. Актуальные правила работы всегда находятся у соответствующих canonical owners в `automation@v1`; этот файл сохраняет историю того, как организация к ним пришла.

---

## 9 октября 2026

### Устаревшие CI-проверки теперь можно безопасно обновлять автоматически

В `automation@v1` внедрена **CI Revalidation** — отдельная GitHub-native maintenance capability для повторной проверки репозиториев после изменения stable delegated workflows или появления нового HEAD без актуального CI. Проверка запускается через `workflow_dispatch` на текущем HEAD, не создаёт фиктивных коммитов и не меняет учебное содержимое. Результат признаётся успешным только после локального CI с подтверждённой provenance стабильной версии `automation@v1` и повторного Organization Health.

Новая GitHub App **ROSBIOTECH Studies CI Revalidation** использует отдельную минимальную область доступа к выбранным предметным репозиториям, `subject-template` и `inbox`. Health Check App остаётся read-only, доступ к исходникам не расширяется до записи.

Правило onboarding новых дисциплин теперь учитывает оба допустимых представления области установки GitHub Apps — одиночный `repository_type` и множественный `repository_types`. При добавлении нового предметного репозитория Scheduled напомнит о ручном подключении **Source Mover** и **CI Revalidation**, если это требуется по актуальному реестру, а повторять уже обработанные напоминания не будет.

Проведены пилотный и полный production burn-in: повторные CI на всех 9 предметных репозиториях, шаблоне и inbox завершились успешно без контентных коммитов. Финальный fresh Organization Health: **85 PASS, 0 WARN, 0 FAIL — healthy**.

[Реализация — automation#54](https://github.com/rosbiotech-studies/automation/pull/54) · [Активация — automation#55](https://github.com/rosbiotech-studies/automation/pull/55)

---

## 4 октября 2026

### Конспекты получили единый мягкий presentation guide

В `automation@v1` появился `NOTES_PRESENTATION_GUIDE.md` — короткий рекомендательный owner для человекочитаемого представления `notes.md`.

Guide не задаёт универсальный шаблон лекции, практики или лабораторной: он направляет структуру по смыслу материала, убирает из пользовательского слоя лишнюю AI/process-метаинформацию, сохраняет полезную полноту и не разрешает scheduled reconciliation массово переформатировать уже удачные конспекты. Локальные правила дисциплин, evidence, homework и LaTeX по-прежнему принадлежат своим canonical owners.

Перед rollout правила прошли read-only burn-in на реальных сценариях: повреждённая транскрибация, семинарская дискуссия, incremental continuation практики, справочный `other`-контекст и расчётная лабораторная.

[Техническая история — automation#40](https://github.com/rosbiotech-studies/automation/pull/40)

### Расписание получило детерминированный GitHub transport

После live burn-in расписание больше не зависит от способности конкретной AI-сессии напрямую обратиться к API РОСБИОТЕХ.

В `automation` появился детерминированный resolver: он читает canonical название группы, получает актуальный `groupID` через `/api/Groups`, затем запрашивает `/api/Rasp` и fail-closed проверяет структуру и identity ответа. Реальный GitHub Actions smoke подтвердил цепочку `24о-090301-ИИ/1 → 18495 → 256 строк расписания`.

GitHub Actions теперь является основным transport-слоем расписания. Если свежий verified cache точно подходит запросу, используется materialized snapshot из `automation:schedule-cache`; для другой группы или диапазона используется постоянный Git-native on-demand path через служебную ветку `schedule-query`. SHA request-коммита однозначно связывает запрос с verified result. Direct HTTP текущего исполнителя сохранён только как менее надёжный fallback, ручной JSON — как последний аварийный вариант; web/browser search расписанием не считается. End-to-end rollout on-demand path независимо подтвердил `24о-090301-ИИ/2 → 18496 → 17 строк` на неделе с 5 октября 2026. Provider-specific ID по-прежнему не является canonical metadata и не сохраняется в `current.yml`.

Первоначальный cross-repository caller из публичного `.github` был отклонён burn-in тестом из-за visibility boundary GitHub Actions; production materialization и on-demand query поэтому полностью размещены внутри приватного `automation`.

Provider discipline теперь также имеет проверенный deterministic mapping основных типов занятий: `лек → lecture`, `пр → practice`, `лаб → laboratory`; subgroup suffix вида `, п/г N` удаляется только для subject matching. Live smoke подтвердил mapping на всех 256 строках текущего полного расписания. На этой основе semester-5 `class_types` reconciled в предметных metadata без догадок по цвету или структуре repository.

[Техническая история — automation#38](https://github.com/rosbiotech-studies/automation/pull/38) · [automation#39](https://github.com/rosbiotech-studies/automation/pull/39) · [automation#41](https://github.com/rosbiotech-studies/automation/pull/41) · automation#43

---

## 3 октября 2026

### Расписание перестало зависеть от сохранённого groupID

Выяснилось, что provider-specific ID учебной группы меняется: для `24о-090301-ИИ/1` ранее использовался `16675`, а позднее API списка групп уже возвращал `18495`. Поэтому group ID больше не считается частью устойчивого академического контекста.

`.github/current.yml` переведён на schema 2 и хранит только canonical название группы. Каждый независимый запрос расписания теперь сначала получает актуальный ID через `/api/Groups` по точному совпадению названия группы и только после этого обращается к `/api/Rasp`.

Старые, сохранённые или запомненные ID запрещено использовать как источник истины; актуальный ID является только временным transport value одного schedule-access.

[Техническая история — automation#37](https://github.com/rosbiotech-studies/automation/pull/37)

---

## 30 сентября 2026

### Inbox стал полностью идемпотентным

Исправлена самоподдерживающаяся петля `last_scanned_commit`: служебное продвижение cursor больше не заставляет следующую scheduled-сессию считать, что в Inbox появилось новое изменение.

Теперь полностью обработанный Inbox приходит в стабильное состояние. Если новых данных нет, повторные проверки не создают лишних commits и уведомлений.

[Техническая история — automation#31](https://github.com/rosbiotech-studies/automation/pull/31)

---

## 29 сентября 2026

### Организация научилась проверять саму себя

Запущена полноценная система **Federated Health**. Теперь ROSBIOTECH Studies может проверять не только отдельный repository, но и согласованность всей распределённой системы: актуальность delegated CI, metadata, state-файлов, производных README и других организационных инвариантов.

Для проверки создан отдельный read-only GitHub App **ROSBIOTECH Studies Health Check** с минимальными правами.

Одновременно прошёл большой integrity-рефакторинг:

- у правил стали ещё чётче разделены canonical owners;
- именование сущностей и доступ к расписанию вынесены в отдельные leaf standards;
- production prompts окончательно оставлены только оркестраторами;
- появился единый canonical root layout и проверяемый `other-NNN-<slug>`;
- добавлены authoritative `class_types` и проверки конфликтов target;
- централизованы executable metadata, naming и state-integrity checks;
- `.github/AGENTS.md` окончательно оставлен navigation-only.

Это одно из крупнейших архитектурных обновлений проекта: организация стала не только автоматизированной, но и способной **проверять собственную целостность**.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-29-federated-health-and-integrity.md)

---

## 28 сентября 2026

### GitHub automation получила собственные identities и прошла реальный burn-in

Появился canonical registry GitHub automation identities и правил хранения credentials. Read-only Health и write-capable Source Mover получили разные роли и границы доступа.

В тот же день реальные smoke/retry сценарии deterministic Source Mover нашли дефект с кириллическими путями Git. Проверка была исправлена на lossless NUL-delimited path handling, после чего Unicode payload успешно прошёл повторный transfer.

[Техническая история — automation#25](https://github.com/rosbiotech-studies/automation/pull/25) · [automation#28](https://github.com/rosbiotech-studies/automation/pull/28)

---

## 27 сентября 2026

### Появился настоящий академический контекст

У организации появился единый source of truth текущего учебного состояния — `.github/current.yml`: учебный год, семестр и группа.

Все предметные repositories были переведены на `.studyrepo.yml` schema 2. Теперь предмет хранит не копию «текущего семестра», а собственную историю `academic.terms`, поэтому одна дисциплина может корректно продолжаться несколько семестров.

Главная страница стала semester-aware: отдельно показывает текущий семестр, навигацию по периодам и общий список дисциплин.

Одновременно формализован доступ к расписанию: direct GET, проверка группы и безопасный pasted-JSON fallback.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-27-academic-context.md)

---

## 24 сентября 2026

### Inbox перестал зависеть от способности AI переносить большие файлы

Для больших UTF-8 payload появился lossless **AI-readable read-cache**: исходный Git blob остаётся неизменным, а AI получает проверяемое chunked представление для чтения.

Следом появился **deterministic Source Mover**. AI теперь отвечает за смысл и выбор target, а bytes source переносит проверяемая GitHub automation по canonical transfer request.

Так semantic reasoning и byte transport были разделены на два независимых слоя.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-24-deterministic-inbox-pipeline.md)

---

## 23 сентября 2026

### Telegram Inbox Bot стал официальным trusted transport

Реальная GitHub App identity бота была зарегистрирована в canonical trusted transport registry.

С этого момента service-originated provenance перестала основываться на доверии к одним только commit trailers: организация получила независимую проверяемую trust boundary для Telegram intake.

[Подробнее об эволюции Inbox Bot и identity →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-14-inbox-bot-and-trusted-identity.md)

---

## 21 сентября 2026

### Домашние работы получили собственный lifecycle

Появился canonical `HOMEWORK_STANDARD.md`, отдельный `homework.md`, execution state и scheduled homework executor.

Домашнее задание при этом не стало новым типом учебной сущности: assignment и solution остаются отношениями к существующему занятию. Для расчётных работ закреплены дата первой строкой и естественный «тетрадный» формат.

### Выбор учебной сущности стал формальным алгоритмом

Появился `ENTITY_RESOLUTION_STANDARD.md`. Относительные формулировки вроде «сегодняшняя пара» и «эта лабораторная» теперь разрешаются через единый алгоритм, а расписание используется только как evidence и не становится учебным источником.

В тот же день появился GitHub Math compatibility registry для реально наблюдавшихся renderer-проблем.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-21-homework-and-entity-resolution.md)

---

## 18 сентября 2026

### Правила превратились в настоящий single-source-of-truth graph

После перехода на canonical AGENTS pointer организация дочистила дублирующиеся правила из README, policy и production prompts.

У изменяемого contract теперь должен быть один owner, а остальные документы только ведут к нему. Root README `automation` стал индексом, prompts — orchestration layers, а `README_COMMON.md` был явно признан производным human-facing представлением.

Это стало фундаментом для всех последующих крупных миграций.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-18-single-source-of-truth.md)

---

## 17 сентября 2026

### Предметные AGENTS перестали хранить копию общих правил

Вместо синхронизируемого общего блока каждый предметный `AGENTS.md` получил короткий canonical pointer на `automation@v1/policy/AGENTS_COMMON.md`.

В этот же период были формализованы duplicate/collision semantics и единое значение состояния `processed` для interactive и scheduled обработки.

[Подробнее о переходе →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-18-single-source-of-truth.md)

---

## 16 сентября 2026

### У организации появилась собственная главная страница

Создан специальный repository `.github` с AI-навигацией и human-facing profile README.

Появился автоматический catalog sync дисциплин, поэтому список предметов больше не нужно было вручную поддерживать в нескольких местах.

[Техническая история — automation#4](https://github.com/rosbiotech-studies/automation/pull/4)

---

## 15 сентября 2026

### Организация научилась учитывать качество источника

Появился `SOURCE_EVIDENCE_STANDARD.md`.

Для source стали отдельно рассматриваться **авторитетность содержания** и **надёжность извлечения**. Нативные материалы преподавателя, фотографии, рукописи, live-транскрибации и AI-производные представления получили разные рекомендации по обработке.

Это помогло перестать путать «хорошо читается» с «является первичным и надёжным источником».

---

## 14 сентября 2026

### Contributor identity была отделена от технического transport

Появились transport-independent contributor provenance, stable contributor tags и canonical trusted transport model.

Inbox Bot начал переход от owner/PAT-схемы к GitHub App + Device Flow: пользователь, Telegram transport и GitHub machine identity стали разными сущностями с разными ролями.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-14-inbox-bot-and-trusted-identity.md)

---

## 13 сентября 2026

### Появился Telegram Inbox Bot

За первый день бот прошёл путь от минимального Telegram intake до crash-safe сервиса с SQLite state, ACL, invites, subject sync, recovery после неоднозначных GitHub операций и deployment-моделью для Orange Pi.

Появилась возможность быстро отправлять учебные материалы в организацию прямо из Telegram, не открывая GitHub.

[Подробнее об эволюции Inbox Bot →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-14-inbox-bot-and-trusted-identity.md)

---

## 10 сентября 2026

### Формулы получили единый GitHub-safe стандарт

Введён обязательный `MARKDOWN_LATEX_STANDARD.md`: inline-формулы через `$...$`, блочные через `$$...$$`, без нестабильно отображавшихся GitHub delimiters.

### Inbox впервые прошёл реальный end-to-end маршрут

Материалы лабораторной №1 по ПиАПП были приняты через Inbox, перенесены в предметный repository, объединены в общий `notes.md`, подтверждены state/receipt и только после проверки удалены из актуального дерева Inbox.

Новая knowledge-base architecture впервые отработала на реальных учебных данных.

[Подробнее о формировании архитектуры →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-09-organization-foundation.md)

---

## 9 сентября 2026

### ROSBIOTECH Studies стала полноценной системой

Появились единый `STUDY_REPOSITORY_STANDARD.md`, `.studyrepo.yml`, `subject-template`, общий `automation@v1`, Inbox и предметные repositories текущего семестра.

Были формализованы `notes.md`, `sources/`, notes reconciliation, entity model, ingestion security, digitization и общий scheduled knowledge-sync.

Это основной день перехода от хорошего прототипа одного предмета к общей AI-powered учебной базе.

[Подробнее об обновлении →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-09-organization-foundation.md)

---

## 8 сентября 2026

### Общая automation отделилась от предмета

Локальная validation БЖД была вынесена в отдельный `rosbiotech-studies/automation`, а предметный repository переключился на reusable workflow.

Именно тогда стало ясно, что правила и CI должны жить на уровне организации, а не копироваться по дисциплинам.

[Подробнее о формировании архитектуры →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-09-organization-foundation.md)

---

## 7 сентября 2026

### Всё началось с БЖД

Создан первый предметный repository — `bzhd`.

Уже в первом прототипе появились идеи, которые пережили все последующие перестройки: отдельные сущности занятий, единый `notes.md`, первичные материалы в `sources/`, contributor metadata и автоматическая проверка структуры.

Через два дня эта модель выросла в архитектуру всей организации.

[Подробнее о первых днях →](https://github.com/rosbiotech-studies/automation/blob/v1/docs/change-notes/2026-09-09-organization-foundation.md)
