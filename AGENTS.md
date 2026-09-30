# rosbiotech-studies

Учебная база знаний РОСБИОТЕХ. Этот файл нужен только для навигации по организации; рабочие правила находятся в соответствующих репозиториях.

## Быстрый маршрут

- Если задача относится к явно названной дисциплине — найди её в каталоге ниже, открой предметный репозиторий и прочитай его `.studyrepo.yml`, `README.md` и `AGENTS.md`.
- Если дисциплина/занятие не указаны явно и target может зависеть от текущего семестра, времени или расписания — сначала прочитай [`current.yml`](current.yml), затем применяй canonical organization/entity-resolution policy из `automation@v1`. Актуальные year/semester/group берутся из `current.yml`, а не из предметного repository.
- Не исследуй остальные репозитории без необходимости.
- Если задача относится к устройству организации, общим правилам или фоновой синхронизации — начни с `automation`.
- Если нужна история значимых обновлений самой организации — смотри [`CHANGELOG.md`](CHANGELOG.md); правила ведения этой истории принадлежат `automation@v1/policy/ORGANIZATION_STANDARD.md`.

`current.yml` содержит только актуальный academic context и не является набором правил. Служебные repository roles, metadata semantics и правила current-context resolution определяются canonical policy в [`automation@v1`](https://github.com/rosbiotech-studies/automation/tree/v1/policy).

Если задача относится к устройству организации или служебному repository, начни с `automation` и следуй canonical architecture/policy оттуда.

## Дисциплины

<!-- SUBJECTS:BEGIN -->
- **Безопасность жизнедеятельности** (`БЖД`) → [`bzhd`](https://github.com/rosbiotech-studies/bzhd) — `2026-2027 / семестр 5`
- **Бизнес-планирование** → [`business-planning`](https://github.com/rosbiotech-studies/business-planning) — `2026-2027 / семестр 5`
- **Защита информации** → [`information-security`](https://github.com/rosbiotech-studies/information-security) — `2026-2027 / семестр 5`
- **Имитационное моделирование** → [`simulation-modeling`](https://github.com/rosbiotech-studies/simulation-modeling) — `2026-2027 / семестр 5`
- **Компьютерное моделирование технологических процессов** → [`technological-process-modeling`](https://github.com/rosbiotech-studies/technological-process-modeling) — `2026-2027 / семестр 5`
- **Методы оптимизации и моделирование систем** (`МОиМС`) → [`optimization-and-system-modeling`](https://github.com/rosbiotech-studies/optimization-and-system-modeling) — `2026-2027 / семестр 5`; `2026-2027 / семестр 6`
- **Операционные системы** (`ОС`) → [`operating-systems`](https://github.com/rosbiotech-studies/operating-systems) — `2026-2027 / семестр 5`
- **Процессы и аппараты пищевых производств** (`ПиАПП`) → [`food-processes-and-apparatus`](https://github.com/rosbiotech-studies/food-processes-and-apparatus) — `2026-2027 / семестр 5`
- **Численные методы** → [`numerical-methods`](https://github.com/rosbiotech-studies/numerical-methods) — `2026-2027 / семестр 5`
<!-- SUBJECTS:END -->

