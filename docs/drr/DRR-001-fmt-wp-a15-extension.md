# DRR-001: Введение `WorkPlan` и A.15-цепочки в FMT (`bc / role / method_description`)

**Статус:** Proposed
**Дата:** 2026-04-20
**Автор:** Ruslan (Стратег)
**Область применения:** FMT-exocortex-template (Platform-space) и все downstream-воркспейсы (DS-strategy, DS-ava, DS-aicu и т.п.)
**Тип по E.9:** полный DRR (семантическое изменение Δ-2: переопределение сущности `WP` в шаблоне)
**Builds on:** FPF A.15 (Role–Method–Work Alignment), A.15.1 (`U.Work`), A.15.2 (`U.WorkPlan`), A.7 (Strict Distinction), E.9 (DRR Method), E.10 (LEX-BUNDLE — L-PROC / L-FUNC / L-SCHED / L-ACT)
**Supersedes:** конвенцию «`WP-N` = контейнер смешанных сущностей» из `WORKPLAN.md`, `WP-REGISTRY.md`, `wp-new/SKILL.md`, `day-plan.md` (v0.3.x)

---

## 1. Problem frame — «почему мы это обсуждаем»

В текущем состоянии FMT и моего DS-strategy сущность `WP-N` (`РП`) структурно перегружена. Под одним номером могут лежать:

- **`U.BoundedContext`** (направление): `WP-ava-blog` — не имеет DoD, живёт годами.
- **`U.Capability` / `U.MethodDescription`** (способ/рецепт): `WP-scanner-skills` — описание умения, а не работа.
- **`U.WorkPlan`** (план с DoD): «статья PT с анонсом» — то, что называют РП правильно.
- **`U.Work`** (факт исполнения): «отработал вечер 20.04» — дневная запись.
- **`WorkProduct`** (артефакт-результат): «опубликованный пост avareange.ru/blog/…».

Это именно **A.15:2** «Ambiguous Process/Activity» + «Capability-as-Work» — «туман коммуникации», описанный FPF как парализующий принятие решений.

Симптомы в текущей системе:

1. `WORKPLAN.md` содержит записи без DoD — их нельзя закрыть, они бесконечны.
2. `DayPlan` ссылается на `РП № …`, но за номером стоит неопределённая смесь — нельзя однозначно сказать «вот это сделано».
3. `wp-new/SKILL.md` спрашивает `budget`, `criteria`, `result-of-month` (поля `U.WorkPlan`), но именует сущность `Work Product` — конфликт типов.
4. `ONTOLOGY.md` §4 определяет `WP = Рабочий продукт (экземпляр)`, но в жизни `WP-N` = «тема/направление/поток».
5. Невозможно аудитно ответить на вопрос «кто, по какой методе, по какому плану это сделал» — цепочка Role → Method → WorkPlan → Work отсутствует.

FPF-цитата (A.15:1, A.15:2): распутать «role vs method vs schedule vs actual run» — прямая задача A.15.

---

## 2. Decision — «что мы делаем»

### 2.1 Лексическое решение (L-PROC / L-SCHED / L-ACT)

В FMT и всех downstream-репо вводится **строгое A.7-различение** между пятью сущностями:

| FPF-тип | Имя в FMT | Где живёт | Имеет DoD? | Имеет дату? |
|---------|-----------|-----------|:----------:|:-----------:|
| `U.BoundedContext` | **BC** — направление / смысловое поле | `BC-REGISTRY.md` | ✗ | ✗ |
| `U.MethodDescription` | **SKILL / метода** — рецепт | `.claude/skills/**/SKILL.md` | ✗ | ✗ |
| `U.WorkPlan` (A.15.2) | **`WorkPlan`** — план с DoD (пишется целиком, без аббревиатуры) | `inbox/WorkPlan-*.md` + `WorkPlan-REGISTRY.md` | ✓ | план (start/end) |
| `U.Work` (A.15.1) | **Work** — запись факта исполнения | `DayPlan` (строка), `journal/*.md` | — (обратная ссылка на `WorkPlan`) | ✓ факт |
| `WorkProduct` (A.15.1 outcome) | **`WP / РП`** — артефакт-результат | `avareange.ru/blog/…`, `docs/…`, Git-коммит | — | дата публикации |

**Лексический инвариант (важно):**

- **`WP / РП`** в FMT сохраняет смысл из `ONTOLOGY.md §4` и SPF — это **`WorkProduct`** (артефакт, результат работы). Аббревиатура не переопределяется.
- **`WorkPlan`** (план с DoD) — новая сущность в FMT, пишется **только целиком**, без сокращения. Это снимает риск коллизии «WP = что именно?».
- **`BC`** — единственная аббревиатура для направлений; никаких `WP-<направление>` больше нет.

### 2.2 Структурное решение — структура файла `WorkPlan`

Каждый файл `inbox/WorkPlan-*.md` **ДОЛЖЕН** содержать три новых обязательных поля в frontmatter, закрывающих A.15-цепочку:

```yaml
---
workplan: 15                              # номер плана (не путать с номером артефакта WP)
title: Статья PT «Гонка технологий и человек…»
status: pending                           # pending | in_progress | done | cancelled
created: 2026-04-18
source: WeekPlan W17 строка 8

# --- A.15 chain (обязательно) ---
bc: BC-ava                                # U.BoundedContext — из BC-REGISTRY.md
role: Ruslan#ContentStrategist:BC-ava@W17 # U.RoleAssignment (A.2.1: Holder#Role:Context)
method_description:                        # U.MethodDescription (список SKILL-файлов)
  - .claude/skills/content-strategist/SKILL.md
  - .claude/skills/seo-article/SKILL.md

# --- FMT fields (сохранены без изменений) ---
budget: 1.5-2h
priority: высокий
result_of_month: R1
verification_class: open-loop

# --- B.5.1 + A.15.3 (опционально, но рекомендуется) ---
lifecycle_stage: Evidence                 # Explore → Shape → Evidence → Operate
variance_trail: []                        # A.15.3 SlotFillingsPlanItem — лог отклонений
---
```

**Что сохраняется без изменений:**
- `title`, `status`, `created`, `source` — как в текущем `wp-new/SKILL.md`.
- `budget`, `priority`, `result_of_month`, `verification_class` — как в текущем FMT.
- Секции `## Описание`, `## Артефакт`, `## Критерий готовности` → переименовываются в `## Контекст`, `## WorkProduct`, `## DoD (A.15.2)` (см. §2.4).

**Что переименовывается:**
- Ключ `wp:` → `workplan:` (в frontmatter). Старое `wp:` в legacy-файлах остаётся до touch-to-migrate.
- Секция `## Критерий готовности` → `## DoD (U.WorkPlan.DoD, A.15.2)`.
- Секция `## Артефакт` → `## WorkProduct (WP / РП)` — именно здесь живёт финальный артефакт, к которому ведёт план.
- Новая секция `## Evidence (U.Work, A.15.1)` — заполняется по ходу исполнения, ссылки на строки `DayPlan`.

**Имена файлов:**
- Новые планы: `inbox/WorkPlan-15.md`, `inbox/WorkPlan-16.md` и т.д. (нумерация сквозная).
- Legacy: `inbox/WP-*.md` существуют до Phase 2; при touch-to-migrate файл либо переименовывается в `WorkPlan-N.md`, либо, если это было направление, превращается в запись `BC-REGISTRY.md` без создания файла-плана.

### 2.3 Расщепление регистра

Единый `WP-REGISTRY.md` заменяется на **два** регистра с чёткими колонками:

- **`BC-REGISTRY.md`** — все `U.BoundedContext`-ы (направления). Колонки: `BC`, `Область`, `Активные WorkPlan-ы`, `Владелец`.
- **`WorkPlan-REGISTRY.md`** — все планы с DoD. Колонки: `WorkPlan #`, `Title`, `BC`, `Status`, `DoD`, `Owner`, `Budget`, `Priority`, `Stage`, `→ WorkProduct`.

Нумерация `WorkPlan-N` сквозная и **независима** от нумерации артефактов `WP-N` (если последняя вообще нужна; обычно `WorkProduct` идентифицируется URL/путём, а не номером).

### 2.4 Изменение `day-plan.md` (таблица дня)

Старая таблица `| # | РП | Задача | Бюджет | Статус |` заменяется на:

```
| # | BC | WorkPlan (DoD) | Role | Бюджет | Stage | Статус |
```

Каждая строка `DayPlan` — это **одна `U.Work`-запись** (факт исполнения), обязательно ссылающаяся на `WorkPlan`. Day Close-ритуал **ДОЛЖЕН** проверять, что для каждой строки заполнено `WorkPlan` или явно помечен флаг `adhoc` (тогда создаётся lightweight WorkPlan задним числом, CC-DRR.5).

### 2.5 Hard Distinctions (обновление)

В `memory/hard-distinctions.md` добавляются различения уровня FPF A.15:

- `U.MethodDescription ≠ U.Work` (рецепт ≠ исполнение; «скилл существует» ≠ «я его применил»).
- `U.Capability ≠ U.Work` (умение ≠ использование умения).
- `U.WorkPlan ≠ U.Work` (план с DoD ≠ факт дня).
- `U.BoundedContext ≠ U.WorkPlan` (направление ≠ конкретное дело с концом).
- `WorkProduct (артефакт) ≠ U.WorkPlan` (пост ≠ план по написанию поста).
- Термин **`WP / РП`** в FMT означает **строго `WorkProduct`** (артефакт-результат, A.15.1 outcome) — как в `ONTOLOGY.md §4` и SPF. Планирующая сущность именуется **`WorkPlan`** целиком, без аббревиатуры.

### 2.6 Правила миграции (phase-план теперь явно прописан)

Порядок применения Decision:

1. **Phase 0 — метаданные (день 0, обратимо):**
   - создать `docs/drr/DRR-001-…md` (этот файл), `BC-REGISTRY.md` с первыми BC.
   - Никакие существующие файлы не трогаются.
2. **Phase 1 — новые файлы по новой схеме (неделя 1):**
   - `wp-new/SKILL.md` спрашивает 3 новых поля; создаёт файл с новой секцией `## DoD`.
   - `day-plan.md` template — новая таблица.
3. **Phase 2 — миграция legacy `WP-N` (по мере касания):**
   - Каждый старый `WP-N.md` при первом открытии раскладывается на: `BC` (если это направление) или `WorkPlan` (если это дело с DoD). Решение фиксируется в логе миграции.
   - Массовый переписыватель не запускается — принцип «touch-to-migrate».

---

## 3. Rationale — «почему это правильно»

### 3.1 Проверка альтернатив

| Альтернатива | Почему отвергнута |
|---|---|
| **A. Оставить как есть** (`WP-N` как мешок) | Консервирует A.15:2 «туман коммуникации»; DoD ненадёжны; аудит цепочки невозможен. |
| **B. Ввести 5 отдельных префиксов** (`BC-*`, `CAP-*`, `WP-*`, `WORK-*`, `ART-*`) | Максимально точно по FPF, но **непрактично для одного человека**: 5 регистров, 5 skill-ов создания. Нарушает P-1 (Cognitive Elegance). |
| **C. Три слоя** (`BC` / `WorkPlan` / `Work`), `MethodDescription` через ссылку, `WorkProduct` — секция внутри `WorkPlan` | **Принятая.** Три сущности с регистром + две связки полями. Минимальный шов для FMT, полное покрытие A.15-цепочки. |
| **D. Использовать Notion/Linear** | Требует внешнего инструмента, ломает принцип «plain-text exocortex». Отвергнуто архитектурно. |

### 3.2 Pillar check (CC-DRR.2 — требуется оценка по 11 pillars)

| Pillar | Эффект Decision |
|---|---|
| **P-1 Cognitive Elegance** | **+** Три типа вместо пяти; минимальное расширение frontmatter (3 поля). |
| **P-2 Didactic Primacy** | **+** `Hard Distinctions` явно фиксирует различия; пример в E.9 архетипов (`U.Episteme` — ревизия теории) применим. |
| **P-3 Evidence Primacy** | **+++** Впервые появляется явная цепочка `Role → Method → WorkPlan → Work`, аудит восстановим. |
| **P-4 Explicit Rationale** | **+++** Сам DRR — реализация P-4. |
| **P-5 Contextual Scoping** | **+** `bc:` поле делает контекст явным; `BC-REGISTRY` — первая гранитная граница. |
| **P-6 Composition / Mereology** | **0** Нейтрально; `BC ⊃ WorkPlan ⊃ Work` — корректная иерархия. |
| **P-7 Strict Distinction (A.7)** | **+++** Прямая реализация: пять типов, ни один не путается. |
| **P-8 Economy of Means** | **+** Расширение на 3 поля (не 30); Phase 2 по принципу touch-to-migrate. |
| **P-9 Cross-Scale Coherence** | **+** Тот же шаблон работает для «подарок жене к ДР» и «книга DS за квартал». |
| **P-10 Open-Ended Evolution** | **+** DRR сам — механизм эволюции; изменение обратимо (Phase 2 инкрементальна). |
| **P-11 Trustworthy Autonomy** | **+** `method_description` связывает исполнение с документированным рецептом — основа для делегирования (человек/агент/ассистент). |

Баланс: **8 плюсов, 3 нейтрально, 0 минусов.** Решение согласовано со всеми pillars.

### 3.3 Taxonomy-lens check (CC-DRR.2)

| Lens | Комментарий |
|---|---|
| **Gov** (governance) | DRR-001 сам — governance-артефакт; вводит явный процесс для всех будущих WP-изменений. |
| **Arch** (architecture) | Лёгкое расширение схемы (frontmatter + регистр); не затрагивает IPO-паттерн CLAUDE.md. |
| **Onto/Epist** (ontology/epistemology) | Ключевая выгода: `A.15` категориально корректен, `ONTOLOGY.md` §4 `WP` получает однозначное определение. |
| **Prag** (pragmatic) | 3 новых поля в `wp-new` — минимум фрикции; DayPlan-таблица чуть шире (7 колонок vs 5). |
| **Did** (didactic) | `Hard Distinctions` обновляется, онбординг-гайды обновятся в Phase 2. |

### 3.4 Связь с существующей SPF/DP-онтологией

- `ONTOLOGY.md §4` определяет `WP = Рабочий продукт (экземпляр)`. DRR-001 **уточняет**: «экземпляр» понимается как `U.WorkPlan` (планируемое дело с DoD), а не «артефакт-результат». Артефакт-результат именуется `WorkProduct` внутри секции плана.
- `DP.EXOCORTEX.001 «Файл контекста РП»` — сохраняется, теперь это файл `WorkPlan`.
- `DP.M.003 «Ритуал ОРЗ»` — DayPlan-таблица адаптируется (см. §2.4).

### 3.5 Bias-audit (E.5.4)

- **Не-инвентированность:** все поля (`bc`, `role`, `method_description`, `variance_trail`) прямо соответствуют FPF A.2.1 / A.15 / A.15.3. Новых терминов не вводится.
- **Notational independence:** YAML frontmatter — нейтральный носитель; та же схема одинаково читается человеком, Claude и скриптом.
- **Unidirectional dependency:** FMT зависит от FPF (через SPF-Pack), обратной зависимости нет.

---

## 4. Consequences — «что изменится»

### 4.1 Impact graph (CC-DRR.3 — затрагиваемые паттерны/файлы)

| Файл / артефакт | Характер изменения | Фаза |
|---|---|:---:|
| `FMT-exocortex-template/ONTOLOGY.md` §4 | уточнение определения `WP` + 2 новых строки (`BC`, `WorkPlan`) | 1 |
| `FMT-exocortex-template/memory/hard-distinctions.md` | +5 различений (см. §2.5) | 1 |
| `FMT-exocortex-template/.claude/skills/wp-new/SKILL.md` | **разделяется на два скилла:** `workplan-new/SKILL.md` (создание плана) и `wp-new/SKILL.md` остаётся за регистрацией артефакта-результата (если понадобится); alias `wp-new` на время migration ведёт в `workplan-new` | 1 |
| `FMT-exocortex-template/roles/strategist/prompts/day-plan.md` | новая таблица (§2.4) | 1 |
| `FMT-exocortex-template/roles/strategist/prompts/week-plan.md` | колонка `BC` добавлена | 1 |
| `DS-strategy/docs/WP-REGISTRY.md` | расщепляется на `BC-REGISTRY.md` + `WorkPlan-REGISTRY.md` | 2 |
| `DS-strategy/WORKPLAN.md` | переразметка существующих `WP-N` → `BC` или `WorkPlan` | 2 |
| `DS-strategy/inbox/WP-*.md` (≈12 файлов на 2026-04-20) | touch-to-migrate: legacy `WP-N.md` разбирается на `WorkPlan-N.md` (план) и/или запись в `BC-REGISTRY.md` (направление); frontmatter расширяется | 2 |
| `memory/MEMORY.md` | запись о принятии DRR-001 в раздел «Принятые решения» | 1 |
| `docs/onboarding/onboarding-guide.md` | глава про A.15-цепочку (1 страница) | 2 |

### 4.2 Benefits

- **Аудит-цепочка A.15 восстановима:** от строки DayPlan (`U.Work`) через `WorkPlan` → `MethodDescription` → `Role` → `BC`.
- **DoD-надёжность:** у каждого `WorkPlan` есть start/end и DoD — дело можно закрыть.
- **SKILL-реюз:** одна и та же метода (`content-strategist`) может обслуживать 10 разных WorkPlan-ов — без копипасты.
- **BC-стабильность:** `BC-ava`, `BC-aicu`, `BC-home` живут годами, не создают фейковых «закрытий».
- **Совместимость с FPF F.17 UTS:** можно публиковать «таблицу смены недели» с колонками A.15.

### 4.3 Trade-offs и их митигация

| Trade-off | Митигация |
|---|---|
| 3 новых обязательных поля в `wp-new` увеличивают время создания на ~30 сек. | `wp-new/SKILL.md` предлагает BC из списка (автокомплит), `role` собирается из `Ruslan#<роль>:<BC>@<week>` автоматически. |
| Нужно вести `BC-REGISTRY.md` как отдельный файл. | Стартовые 5-7 BC формируются за одну сессию (уже намечены: ava, aicu, home, self, work-dev, clients-mkt). |
| Legacy `WP-N` файлы в DS-strategy понадобится мигрировать. | Phase 2 «touch-to-migrate» — не блокирует работу; миграция идёт по мере касания. |
| Потенциальный конфликт терминологии SPF ↔ A.15 (оба претендовали на «WP»). | **Снят архитектурно:** `WP / РП` в FMT = `WorkProduct` (артефакт) — как в SPF. Планирующая сущность именуется `WorkPlan` целиком, без аббревиатуры. `ONTOLOGY.md §4` получает только новую строку `WorkPlan`, определение `WP` сохраняется. |
| ADR-001, ADR-002 используют термин `РП` в старом смысле. | Оставляем без изменений (provenance); в будущих ADR/DRR используем новую терминологию. |

### 4.4 Риски

- **R1: Over-formalisation.** Человек может начать создавать WorkPlan для тривиальных дел. **Митигация:** в `wp-new` остаётся эвристика «если задача < 15 мин — это `adhoc` строка DayPlan, не WorkPlan».
- **R2: BC-inflation.** Слишком много BC. **Митигация:** жёсткое правило — BC создаётся только через DRR-lite (CC-DRR.5), максимум 2 в квартал.
- **R3: Разрыв с reality.** Frontmatter заполнен, но `Evidence` секция пуста. **Митигация:** Day Close-ритуал проверяет обратные ссылки DayPlan → WorkPlan.Evidence.

### 4.5 LAT pointer (CC-DRR.6)

Не требуется: изменение относится к FMT (Template-слой), не к part F/G FPF-ядра.

---

## 5. Acceptance criteria (DoD самого DRR)

- [ ] Файл `DRR-001-…md` создан в `FMT-exocortex-template/docs/drr/`.
- [ ] `memory/hard-distinctions.md` обновлён новыми различениями из §2.5.
- [ ] `ONTOLOGY.md` §4 содержит уточнение для `WP` и новые строки `BC`, `WorkPlan`.
- [ ] `wp-new/SKILL.md` спрашивает 3 новых поля.
- [ ] `day-plan.md` использует новую таблицу (7 колонок).
- [ ] Хотя бы один новый WorkPlan создан по новой схеме (smoke-test).
- [ ] Запись о принятии DRR-001 добавлена в `MEMORY.md`.

---

## 6. Status log

| Дата | Событие |
|---|---|
| 2026-04-20 | DRR-001 создан со статусом **Proposed**. |

---

_DRR-001 следует методу **E.9 (Design-Rationale Record)** FPF. Четыре структурных компонента (Problem frame / Decision / Rationale / Consequences) сохранены; § 2.6 (phase-план) и §5 (DoD самого DRR) — дополнительные разделы для практического применения в FMT._
