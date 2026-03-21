# Инструкция по выполнению лабораторной работы №3

## «Агентное моделирование: модель Daisyworld»

### Тема и цель

Лабораторная посвящена **агентному моделированию** (Agent-Based Modeling, ABM) на Julia.
Нужно реализовать модель Daisyworld с помощью пакета **Agents.jl**,
построить визуализации, затем для **каждого скрипта** пройти полный цикл литературного программирования:

```
исходный .jl → запуск → литературный .jl → генерация 3 форматов:
  ├── чистый .jl     (Literate.script)
  ├── .ipynb          (Literate.notebook)
  └── .qmd            (Literate.markdown)
→ выполнение .ipynb → интеграция .qmd в отчёт
```

### Полный список скриптов и цикл для каждого

В учебнике предложено **7 скриптов** (+ модель в `src/`). Для каждого --- свой цикл:

| # | Раздел | Исходный скрипт | Полный цикл? |
|---|--------|----------------|-------------|
| 1 | 3.2.3 | `src/daisyworld.jl` | Нет (модуль, подключается через `include`) |
| 2 | 3.2.4 | `scripts/daisyworld.jl` | **Да**: литературный → чистый + ipynb + qmd |
| 3 | 3.2.5 | `scripts/daisyworld-animate.jl` | Нет (только запуск + загрузка видео) |
| 4 | 3.2.6 | `scripts/daisyworld-count.jl` | **Да**: литературный → чистый + ipynb + qmd |
| 5 | 3.2.7 | `scripts/daisyworld-luminosity.jl` | **Да**: литературный → чистый + ipynb + qmd |
| 6 | 3.2.8 | `scripts/daisyworld__param.jl` | **Да**: литературный → чистый + ipynb + qmd |
| 7 | 3.2.9 | `scripts/daisyworld-count__param.jl` | **Да**: литературный → чистый + ipynb + qmd |
| 8 | 3.2.10 | `scripts/daisyworld-luminosity__param.jl` | **Да**: литературный → чистый + ipynb + qmd |

Итого: **6 скриптов** проходят полный цикл литературного программирования.
Для каждого из 6 генерируются 3 производных файла = **18 производных файлов**.

### Краткая сводка команд запуска всех скриптов

Рекомендуется запускать всё **из одной сессии Julia REPL** --- так быстрее,
потому что пакеты компилируются один раз. Запустите Julia из каталога `lab_03_daisyworld/`:

```bash
cd labs/lab03/lab_03_daisyworld
julia --project=.
```

Затем в Julia REPL:

```julia
using DrWatson
@quickactivate "lab_03_daisyworld"

## --- Запуск всех скриптов ---

# Скрипт 1: Базовая визуализация
include(scriptsdir("daisyworld.jl"))

# Скрипт 2: Анимация
include(scriptsdir("daisyworld-animate.jl"))

# Скрипт 3: Динамика числа маргариток
include(scriptsdir("daisyworld-count.jl"))

# Скрипт 4: Динамика модели (ramp)
include(scriptsdir("daisyworld-luminosity.jl"))

# Скрипт 5: Визуализация с параметрами
include(scriptsdir("daisyworld__param.jl"))

# Скрипт 6: Динамика с параметрами
include(scriptsdir("daisyworld-count__param.jl"))

# Скрипт 7: Полная динамика с параметрами
include(scriptsdir("daisyworld-luminosity__param.jl"))

## --- Генерация литературных форматов ---

using Literate

for (script, clean) in [
    ("daisyworld_literate.jl",                   "daisyworld_clean"),
    ("daisyworld-count_literate.jl",             "daisyworld-count_clean"),
    ("daisyworld-luminosity_literate.jl",        "daisyworld-luminosity_clean"),
    ("daisyworld__param_literate.jl",            "daisyworld__param_clean"),
    ("daisyworld-count__param_literate.jl",      "daisyworld-count__param_clean"),
    ("daisyworld-luminosity__param_literate.jl", "daisyworld-luminosity__param_clean"),
]
    src = scriptsdir(script)
    Literate.script(src, scriptsdir(); name=clean)   # → *_clean.jl
    Literate.notebook(src, "notebooks")               # → .ipynb (выполняется автоматически)
    Literate.markdown(src, "docs")                    # → .md
end
```

Ноутбуки выполняются автоматически при `Literate.notebook()` ---
отдельный `jupyter nbconvert --execute` **не нужен**.

Рендеринг отчёта и презентации (из терминала):

```bash
cd labs/lab03/report && quarto render simulation-modeling--lab03--report.qmd
cd ../presentation && quarto render simulation-modeling--lab03--presentation.qmd
```

---

## Шаг 0. Предварительные требования

Убедитесь, что установлены:

- **Julia** (версия 1.12+)
- **Jupyter** (с ядром `julia-1.12`, устанавливается через пакет `IJulia`)
- **Quarto** (для рендеринга отчёта и презентации)

> **Что говорить на видео:**
> «Для выполнения третьей лабораторной работы нам понадобятся:
> Julia версии 1.12 или выше, Jupyter для ноутбуков
> и Quarto для рендеринга отчёта.
> У меня всё установлено с прошлой лабораторной.
>
> Тема --- агентное моделирование.
> Мы реализуем модель Daisyworld с помощью пакета Agents.jl.»

---

## Шаг 1. Создание проекта DrWatson

```julia
using DrWatson
initialize_project("labs/lab03/lab_03_daisyworld"; authors="Ваше Имя")
```

Структура:

```
lab_03_daisyworld/
├── scripts/       # скрипты визуализации
├── src/           # модель Daisyworld
├── data/
├── plots/         # графики
├── notebooks/     # Jupyter notebooks
├── docs/          # Quarto-документация
├── test/
├── Project.toml
└── README.md
```

> **Что говорить на видео:**
> «Создаём проект DrWatson. Как и в прошлой лабораторной,
> DrWatson автоматически создаёт стандартную структуру папок.
> Проект называем `lab_03_daisyworld`.»

---

## Шаг 2. Установка зависимостей

```julia
using Pkg
Pkg.activate("labs/lab03/lab_03_daisyworld")
Pkg.add([
    "DrWatson", "Agents", "Random",
    "Plots", "CairoMakie",
    "DataFrames", "StatsBase",
    "Literate", "IJulia"
])
```

> **Что говорить на видео:**
> «Устанавливаем пакеты. Основные:
> `Agents` --- фреймворк для агентного моделирования в Julia,
> `CairoMakie` --- бэкенд визуализации для `abmplot`,
> `StatsBase` --- статистические функции для выборки позиций,
> `Literate` --- для литературного программирования.
> Все зависимости записываются в `Project.toml`.»

---

## Шаг 3. Создание модели (`src/daisyworld.jl`)

Это модуль модели --- он не проходит цикл литературного программирования,
а подключается через `include(srcdir("daisyworld.jl"))` из всех скриптов.

```julia
using Agents
import StatsBase
using Random

@agent struct Daisy(GridAgent{2})
    breed::Symbol
    age::Int
    albedo::Float64 # 0-1 fraction
end

function update_surface_temperature!(pos, model)
    absorbed_luminosity = if isempty(pos, model) # no daisy
        (1 - model.surface_albedo) * model.solar_luminosity
    else
        daisy = model[id_in_position(pos, model)]
        (1 - daisy.albedo) * model.solar_luminosity
    end
    local_heating = absorbed_luminosity > 0 ? 72 *
        log(absorbed_luminosity) + 80 : 80
    model.temperature[pos...] = (model.temperature[pos...] +
        local_heating) / 2
end

function diffuse_temperature!(pos, model)
    ratio = model.ratio
    npos = nearby_positions(pos, model)
    model.temperature[pos...] =
        (1 - ratio) * model.temperature[pos...] +
        sum(model.temperature[p...] for p in npos) * 0.125 * ratio
end

function propagate!(pos, model)
    isempty(pos, model) && return
    daisy = model[id_in_position(pos, model)]
    temperature = model.temperature[pos...]
    seed_threshold = (0.1457 * temperature - 0.0032 * temperature^2) - 0.6443
    if rand(abmrng(model)) < seed_threshold
        empty_near_pos = random_nearby_position(pos, model, 1,
            npos -> isempty(npos, model))
        if !isnothing(empty_near_pos)
            add_agent!(empty_near_pos, model, daisy.breed, 0, daisy.albedo)
        end
    end
end

function daisy_step!(agent::Daisy, model)
    agent.age += 1
    agent.age >= model.max_age && remove_agent!(agent, model)
end

function daisyworld_step!(model)
    for p in positions(model)
        update_surface_temperature!(p, model)
        diffuse_temperature!(p, model)
        propagate!(p, model)
    end
    model.tick = model.tick + 1
    solar_activity!(model)
end

function solar_activity!(model)
    if model.scenario == :ramp
        if model.tick > 200 && model.tick <= 400
            model.solar_luminosity += model.solar_change
        end
        if model.tick > 500 && model.tick <= 750
            model.solar_luminosity -= model.solar_change / 2
        end
    elseif model.scenario == :change
        model.solar_luminosity += model.solar_change
    end
end

function daisyworld(;
    griddims = (30, 30),
    max_age = 25,
    init_white = 0.2,
    init_black = 0.2,
    albedo_white = 0.75,
    albedo_black = 0.25,
    surface_albedo = 0.4,
    solar_change = 0.005,
    solar_luminosity = 1.0,
    scenario = :default,
    seed = 165,
)
    rng = MersenneTwister(seed)
    space = GridSpaceSingle(griddims)
    properties = (;max_age, surface_albedo, solar_luminosity,
        solar_change, scenario,
        tick = 0, ratio = 0.5, temperature = zeros(griddims))
    properties = Dict(k=>v for (k,v) in pairs(properties))

    model = StandardABM(Daisy, space; properties, rng,
        agent_step! = daisy_step!, model_step! = daisyworld_step!)

    grid = collect(positions(model))
    num_positions = prod(griddims)
    white_positions = StatsBase.sample(grid,
        Int(init_white * num_positions); replace = false)
    for wp in white_positions
        add_agent!(wp, Daisy, model, :white,
            rand(abmrng(model), 0:max_age), albedo_white)
    end
    allowed = setdiff(grid, white_positions)
    black_positions = StatsBase.sample(allowed,
        Int(init_black * num_positions); replace = false)
    for bp in black_positions
        add_agent!(bp, Daisy, model, :black,
            rand(abmrng(model), 0:max_age), albedo_black)
    end

    for p in positions(model)
        update_surface_temperature!(p, model)
    end

    return model
end
```

> **Что говорить на видео:**
> «Создаём файл `src/daisyworld.jl` --- это ядро модели.
> Определяем тип агента `Daisy` с атрибутами: `breed` --- вид маргаритки,
> `age` --- возраст, `albedo` --- альбедо.
>
> Четыре ключевые функции: расчёт температуры клетки,
> диффузия температуры между соседями,
> размножение маргариток при благоприятной температуре,
> и изменение солнечной светимости по сценарию.
>
> Функция `daisyworld()` создаёт модель --- инициализирует сетку 30 на 30,
> размещает белые и чёрные маргаритки и вычисляет начальную температуру.
> Этот файл подключается из всех скриптов через `include`.»

---

## Шаг 4. Скрипт 1: Базовая визуализация (`scripts/daisyworld.jl`)

### 4.1. Создание и запуск

Тепловая карта температуры с маргаритками (раздел 3.2.4 учебника).

**Запуск из терминала** (находясь в `lab_03_daisyworld/`):

```bash
cd labs/lab03/lab_03_daisyworld
julia --project=. scripts/daisyworld.jl
```

**Или из Julia REPL:**

```julia
using DrWatson
@quickactivate "lab_03_daisyworld"
include(scriptsdir("daisyworld.jl"))
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots

include(srcdir("daisyworld.jl"))

using CairoMakie
model = daisyworld()

daisycolor(a::Daisy) = a.breed

plotkwargs = (
    agent_color=daisycolor, agent_size = 20, agent_marker = '*',
    heatarray = :temperature,
    heatkwargs = (colorrange = (-20, 60),),
)
plt1, _ = abmplot(model; plotkwargs...)

step!(model, 5)
plt2, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

step!(model, 40)
plt3, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

save(plotsdir("daisy_step001.png"), plt1)
save(plotsdir("daisy_step005.png"), plt2)
save(plotsdir("daisy_step040.png"), plt3)
```

> **Что говорить на видео:**
> «Создаём первый скрипт --- `scripts/daisyworld.jl`.
> Он строит тепловую карту температуры поверхности,
> маргаритки отображаются звёздочками.
> Строим три снимка: начальный, после 5 и после 40 шагов.
> Видно, как температурное поле эволюционирует под влиянием маргариток.»

### 4.2. Преобразование в литературный стиль

Создаём `scripts/daisyworld_literate.jl` --- добавляем разметку Literate.jl:

```julia
# # Базовая визуализация модели Daisyworld
#
# ## Описание
#
# Построим тепловую карту температуры поверхности модели.
# Маргаритки будут отображаться черно-белыми в соответствии с их видом.
#
# ## Теория
#
# Формула локального нагрева:
# $$T_{\text{local}} = 72 \ln((1 - \alpha) \cdot L) + 80$$
# где $\alpha$ --- альбедо клетки, $L$ --- солнечная светимость.

# ## Подключение пакетов

using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots

# ## Загрузка модели

include(srcdir("daisyworld.jl"))

using CairoMakie
model = daisyworld()

# ## Настройка визуализации
#
# Цвет агента определяется его видом (`breed`).

daisycolor(a::Daisy) = a.breed

plotkwargs = (
    agent_color=daisycolor, agent_size = 20, agent_marker = '*',
    heatarray = :temperature,
    heatkwargs = (colorrange = (-20, 60),),
)

# ## Начальное состояние

plt1, _ = abmplot(model; plotkwargs...)

# ## Состояние после 5 шагов

step!(model, 5)
plt2, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

# ## Состояние после 40 шагов

step!(model, 40)
plt3, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

# ## Сохранение графиков

#jl save(plotsdir("daisy_step001.png"), plt1)
#jl save(plotsdir("daisy_step005.png"), plt2)
#jl save(plotsdir("daisy_step040.png"), plt3)
```

> **Что говорить на видео:**
> «Преобразуем скрипт в литературный стиль.
> Добавляем текстовые пояснения с формулами в комментарии.
> Код остаётся тем же, но теперь каждый блок предваряется описанием.
> Директива `#jl` используется для `save` --- в ноутбуке графики
> отрисовываются inline, сохранять не нужно.»

### 4.3. Генерация трёх форматов

```julia
using Literate

# Чистый .jl (без комментариев-разметки) — в отдельную папку src/
Literate.script(scriptsdir("daisyworld_literate.jl"), scriptsdir(); name="daisyworld_clean")

# Jupyter notebook
Literate.notebook(scriptsdir("daisyworld_literate.jl"), "notebooks")

# Quarto-документация (.qmd / .md)
Literate.markdown(scriptsdir("daisyworld_literate.jl"), "docs")
```

> **Важно:** Указываем `name="daisyworld_clean"`, чтобы выходной файл
> не совпал с входным (иначе `Literate.script` выдаст ошибку).

Результат:
- `scripts/daisyworld_clean.jl` (чистый скрипт без разметки)
- `notebooks/daisyworld_literate.ipynb`
- `docs/daisyworld_literate.md`

### 4.4. Интеграция .qmd в отчёт

> **Примечание:** `Literate.notebook` по умолчанию **выполняет** ноутбук
> при генерации (видно по строке `executing notebook` в консоли).
> Отдельный шаг с `jupyter nbconvert --execute` не нужен ---
> ноутбук уже содержит результаты и графики.

```markdown
![Тепловая карта Daisyworld (шаг 40)](../lab_03_daisyworld/plots/daisy_step040.png){#fig-heatmap width=100%}
```

> **Что говорить на видео:**
> «Теперь проходим полный цикл литературного программирования.
> Из литературного скрипта генерируем три формата:
> `Literate.script` --- чистый Julia-скрипт без разметки,
> `Literate.notebook` --- Jupyter notebook,
> `Literate.markdown` --- Quarto-документация.
>
> Затем выполняем ноутбук через `jupyter nbconvert` ---
> теперь в нём есть результаты и графики.
> И интегрируем `.qmd` в отчёт.
>
> Этот цикл мы повторим для каждого скрипта.»

---

## Шаг 5. Скрипт 2: Анимация (`scripts/daisyworld-animate.jl`)

**Только создание и запуск** --- для анимации полный цикл Literate не требуется.
Результат --- видео, которое нужно загрузить на видеохостинг.

**Запуск:**

```bash
julia --project=. scripts/daisyworld-animate.jl
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots

include(srcdir("daisyworld.jl"))

using CairoMakie
model = daisyworld()

daisycolor(a::Daisy) = a.breed

plotkwargs = (
    agent_color=daisycolor, agent_size = 20, agent_marker = '*',
    heatarray = :temperature,
    heatkwargs = (colorrange = (-20, 60),),
)

abmvideo(
    plotsdir("simulation.mp4"),
    model;
    title = "Daisy World",
    frames = 60,
    plotkwargs...,
)
```

> **Что говорить на видео:**
> «Скрипт анимации --- `daisyworld-animate.jl`.
> Функция `abmvideo` делает 60 кадров симуляции и сохраняет MP4.
> Видно, как маргаритки распространяются по сетке
> и температурное поле меняется. Загрузим видео на видеохостинг.»

---

## Шаг 6. Скрипт 3: Динамика числа маргариток (`scripts/daisyworld-count.jl`)

### 6.1. Создание и запуск

**Запуск:**

```bash
julia --project=. scripts/daisyworld-count.jl
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots

include(srcdir("daisyworld.jl"))

using CairoMakie

black(a) = a.breed == :black
white(a) = a.breed == :white
adata = [(black, count), (white, count)]

model = daisyworld(; solar_luminosity = 1.0)

agent_df, model_df = run!(model, 1000; adata)
figure = Figure(size = (600, 400));

ax = figure[1, 1] = Axis(figure, xlabel = "tick", ylabel = "daisy count")
blackl = lines!(ax, agent_df[!, :time], agent_df[!, :count_black], color = :black)
whitel = lines!(ax, agent_df[!, :time], agent_df[!, :count_white], color = :orange)
Legend(figure[1, 2], [blackl, whitel], ["black", "white"], labelsize = 12)

save(plotsdir("daisy_count.png"), figure)
```

> **Что говорить на видео:**
> «Скрипт `daisyworld-count.jl` строит график динамики числа маргариток
> за 1000 шагов. Видно колебания --- чёрные и белые конкурируют за пространство.»

### 6.2. Полный цикл Literate

1. Создаём `scripts/daisyworld-count_literate.jl` (добавляем разметку `#`)
2. Генерируем три формата:

```julia
Literate.script(scriptsdir("daisyworld-count_literate.jl"), scriptsdir(); name="daisyworld-count_clean")
Literate.notebook(scriptsdir("daisyworld-count_literate.jl"), "notebooks")
Literate.markdown(scriptsdir("daisyworld-count_literate.jl"), "docs")
```

3. Интегрируем `.qmd` в отчёт (ноутбук уже выполнен при генерации).

> **Что говорить на видео:**
> «Проходим полный цикл: литературный скрипт → чистый код + ноутбук + документация.
> Выполняем ноутбук, интегрируем документацию в отчёт.»

---

## Шаг 7. Скрипт 4: Динамика модели (`scripts/daisyworld-luminosity.jl`)

### 7.1. Создание и запуск

**Запуск:**

```bash
julia --project=. scripts/daisyworld-luminosity.jl
```

Комплексный график: число маргариток + температура + солнечная светимость.
Сценарий `:ramp` --- светимость растёт, потом снижается.

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots

include(srcdir("daisyworld.jl"))

using CairoMakie

black(a) = a.breed == :black
white(a) = a.breed == :white
adata = [(black, count), (white, count)]

model = daisyworld(solar_luminosity = 1.0, scenario = :ramp)

temperature(model) = StatsBase.mean(model.temperature)
mdata = [temperature, :solar_luminosity]

agent_df, model_df = run!(model, 1000; adata = adata, mdata = mdata)

figure = CairoMakie.Figure(size = (600, 600));
ax1 = figure[1, 1] = Axis(figure, ylabel = "daisy count")
blackl = lines!(ax1, agent_df[!, :time], agent_df[!, :count_black], color = :red)
whitel = lines!(ax1, agent_df[!, :time], agent_df[!, :count_white], color = :blue)
figure[1, 2] = Legend(figure, [blackl, whitel], ["black", "white"])

ax2 = figure[2, 1] = Axis(figure, ylabel = "temperature")
ax3 = figure[3, 1] = Axis(figure, xlabel = "tick", ylabel = "luminosity")

lines!(ax2, model_df[!, :time], model_df[!, :temperature], color = :red)
lines!(ax3, model_df[!, :time], model_df[!, :solar_luminosity], color = :red)
for ax in (ax1, ax2); ax.xticklabelsvisible = false; end
figure

save(plotsdir("daisy_luminosity.png"), figure)
```

> **Что говорить на видео:**
> «Скрипт `daisyworld-luminosity.jl` строит три панели:
> число маргариток, температура, светимость.
> Сценарий `ramp` --- светимость растёт с шага 200 до 400,
> снижается с 500 до 750. Видна саморегуляция: маргаритки
> компенсируют изменение светимости до определённого предела.»

### 7.2. Полный цикл Literate

1. Создаём `scripts/daisyworld-luminosity_literate.jl`
2. Генерируем:

```julia
Literate.script(scriptsdir("daisyworld-luminosity_literate.jl"), scriptsdir(); name="daisyworld-luminosity_clean")
Literate.notebook(scriptsdir("daisyworld-luminosity_literate.jl"), "notebooks")
Literate.markdown(scriptsdir("daisyworld-luminosity_literate.jl"), "docs")
```

3. Интегрируем `.qmd` в отчёт (ноутбук уже выполнен при генерации).

---

## Шаг 8. Скрипт 5: Базовая визуализация с параметрами (`scripts/daisyworld__param.jl`)

### 8.1. Создание и запуск

**Запуск:**

```bash
julia --project=. scripts/daisyworld__param.jl
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots
using CairoMakie

include(srcdir("daisyworld.jl"))

## Параметры эксперимента
param_dict = Dict(
    :griddims => (30, 30),
    :max_age => [25, 40],
    :init_white => [0.2, 0.8],
    :init_black => 0.2,
    :albedo_white => 0.75,
    :albedo_black => 0.25,
    :surface_albedo => 0.4,
    :solar_change => 0.005,
    :solar_luminosity => 1.0,
    :scenario => :default,
    :seed => 165,
)

## Создаём список всех комбинаций
params_list = dict_list(param_dict)

for params in params_list

    model = daisyworld(;params...)

    daisycolor(a::Daisy) = a.breed

    plotkwargs = (
        agent_color=daisycolor, agent_size = 20, agent_marker = '*',
        heatarray = :temperature,
        heatkwargs = (colorrange = (-20, 60),),
    )
    plt1, _ = abmplot(model; plotkwargs...)

    step!(model, 5)
    plt2, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

    step!(model, 40)
    plt3, _ = abmplot(model; heatarray = model.temperature, plotkwargs...)

    plt1_name = savename("daisyworld",params) * "_step01" * ".png"
    plt2_name = savename("daisyworld",params) * "_step04" * ".png"
    plt3_name = savename("daisyworld",params) * "_step40" * ".png"

    save(plotsdir(plt1_name), plt1)
    save(plotsdir(plt2_name), plt2)
    save(plotsdir(plt3_name), plt3)
end
```

> **Что говорить на видео:**
> «Теперь добавляем параметры. Используем `dict_list` из DrWatson ---
> она генерирует все комбинации. Варьируем `max_age` (25, 40)
> и `init_white` (0.2, 0.8) --- получается 4 комбинации.
> Для каждой строим тепловые карты. DrWatson автоматически
> генерирует уникальные имена файлов через `savename`.»

### 8.2. Полный цикл Literate

1. Создаём `scripts/daisyworld__param_literate.jl`
2. Генерируем:

```julia
Literate.script(scriptsdir("daisyworld__param_literate.jl"), scriptsdir(); name="daisyworld__param_clean")
Literate.notebook(scriptsdir("daisyworld__param_literate.jl"), "notebooks")
Literate.markdown(scriptsdir("daisyworld__param_literate.jl"), "docs")
```

3. Интегрируем `.qmd` в отчёт (ноутбук уже выполнен при генерации).

---

## Шаг 9. Скрипт 6: Динамика числа маргариток с параметрами (`scripts/daisyworld-count__param.jl`)

### 9.1. Создание и запуск

**Запуск:**

```bash
julia --project=. scripts/daisyworld-count__param.jl
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots
using CairoMakie

include(srcdir("daisyworld.jl"))

black(a) = a.breed == :black
white(a) = a.breed == :white
adata = [(black, count), (white, count)]

## Параметры эксперимента
param_dict = Dict(
    :griddims => (30, 30),
    :max_age => [25, 40],
    :init_white => [0.2, 0.8],
    :init_black => 0.2,
    :albedo_white => 0.75,
    :albedo_black => 0.25,
    :surface_albedo => 0.4,
    :solar_change => 0.005,
    :solar_luminosity => 1.0,
    :scenario => :default,
    :seed => 165,
)

params_list = dict_list(param_dict)

for params in params_list

    model = daisyworld(;params...)

    agent_df, model_df = run!(model, 1000; adata)
    figure = Figure(size = (600, 400));
    ax = figure[1, 1] = Axis(figure, xlabel = "tick", ylabel = "daisy count")
    blackl = lines!(ax, agent_df[!, :time], agent_df[!, :count_black], color = :black)
    whitel = lines!(ax, agent_df[!, :time], agent_df[!, :count_white], color = :orange)
    Legend(figure[1, 2], [blackl, whitel], ["black", "white"], labelsize = 12)

    plt_name = savename("daisy-count",params) * ".png"

    save(plotsdir(plt_name), figure)
end
```

> **Что говорить на видео:**
> «Скрипт `daisyworld-count__param.jl` --- динамика числа маргариток
> для всех 4 комбинаций параметров. Для каждой комбинации
> прогоняем 1000 шагов и строим график.
> Видно, как разные параметры влияют на колебания.»

### 9.2. Полный цикл Literate

1. Создаём `scripts/daisyworld-count__param_literate.jl`
2. Генерируем:

```julia
Literate.script(scriptsdir("daisyworld-count__param_literate.jl"), scriptsdir(); name="daisyworld-count__param_clean")
Literate.notebook(scriptsdir("daisyworld-count__param_literate.jl"), "notebooks")
Literate.markdown(scriptsdir("daisyworld-count__param_literate.jl"), "docs")
```

3. Интегрируем `.qmd` в отчёт (ноутбук уже выполнен при генерации).

---

## Шаг 10. Скрипт 7: Динамика модели с параметрами (`scripts/daisyworld-luminosity__param.jl`)

### 10.1. Создание и запуск

**Запуск:**

```bash
julia --project=. scripts/daisyworld-luminosity__param.jl
```

```julia
using DrWatson
@quickactivate "project"
using Agents
using DataFrames
using Plots
using CairoMakie

include(srcdir("daisyworld.jl"))

black(a) = a.breed == :black
white(a) = a.breed == :white
adata = [(black, count), (white, count)]

## Параметры эксперимента
param_dict = Dict(
    :griddims => (30, 30),
    :max_age => [25, 40],
    :init_white => [0.2, 0.8],
    :init_black => 0.2,
    :albedo_white => 0.75,
    :albedo_black => 0.25,
    :surface_albedo => 0.4,
    :solar_change => 0.005,
    :solar_luminosity => 1.0,
    :scenario => :ramp,
    :seed => 165,
)

params_list = dict_list(param_dict)

for params in params_list

    model = daisyworld(;params...)

    temperature(model) = StatsBase.mean(model.temperature)
    mdata = [temperature, :solar_luminosity]

    agent_df, model_df = run!(model, 1000; adata = adata, mdata = mdata)

    figure = CairoMakie.Figure(size = (600, 600));
    ax1 = figure[1, 1] = Axis(figure, ylabel = "daisy count")
    blackl = lines!(ax1, agent_df[!, :time], agent_df[!, :count_black], color = :red)
    whitel = lines!(ax1, agent_df[!, :time], agent_df[!, :count_white], color = :blue)
    figure[1, 2] = Legend(figure, [blackl, whitel], ["black", "white"])

    ax2 = figure[2, 1] = Axis(figure, ylabel = "temperature")
    ax3 = figure[3, 1] = Axis(figure, xlabel = "tick", ylabel = "luminosity")
    lines!(ax2, model_df[!, :time], model_df[!, :temperature], color = :red)
    lines!(ax3, model_df[!, :time], model_df[!, :solar_luminosity], color = :red)
    for ax in (ax1, ax2); ax.xticklabelsvisible = false; end

    plt_name = savename("daisy-luminosity",params) * ".png"
    save(plotsdir(plt_name), figure)
end
```

> **Что говорить на видео:**
> «Последний скрипт --- `daisyworld-luminosity__param.jl`.
> Комплексная динамика со сценарием `ramp` для всех комбинаций.
> Видно, как при разных параметрах саморегуляция работает по-разному.»

### 10.2. Полный цикл Literate

1. Создаём `scripts/daisyworld-luminosity__param_literate.jl`
2. Генерируем:

```julia
Literate.script(scriptsdir("daisyworld-luminosity__param_literate.jl"), scriptsdir(); name="daisyworld-luminosity__param_clean")
Literate.notebook(scriptsdir("daisyworld-luminosity__param_literate.jl"), "notebooks")
Literate.markdown(scriptsdir("daisyworld-luminosity__param_literate.jl"), "docs")
```

3. Интегрируем `.qmd` в отчёт (ноутбук уже выполнен при генерации).

> **Что говорить на видео:**
> «Итого для каждого из шести скриптов мы прошли полный цикл:
> исходный скрипт, литературная версия, генерация трёх форматов ---
> чистый код, Jupyter notebook, Quarto-документация ---
> выполнение ноутбука, интеграция в отчёт.»

---

## Шаг 11. Сводная генерация всех форматов (удобный скрипт)

Для удобства можно создать `scripts/generate.jl`:

```julia
using Literate

literate_scripts = [
    "daisyworld_literate.jl",
    "daisyworld-count_literate.jl",
    "daisyworld-luminosity_literate.jl",
    "daisyworld__param_literate.jl",
    "daisyworld-count__param_literate.jl",
    "daisyworld-luminosity__param_literate.jl",
]

for script in literate_scripts
    src = scriptsdir(script)
    clean_name = replace(script, "_literate.jl" => "_clean")
    Literate.script(src, scriptsdir(); name=clean_name)  # → *_clean.jl
    Literate.notebook(src, "notebooks")                   # → .ipynb
    Literate.markdown(src, "docs")                        # → .qmd / .md
end
```

> **Что говорить на видео:**
> «Для удобства создаём скрипт `generate.jl`,
> который одним запуском генерирует все 18 производных файлов
> из 6 литературных скриптов.»

---

## Шаг 12. Проверка ноутбуков

`Literate.notebook` по умолчанию **выполняет** ноутбук при генерации.
Отдельный шаг с `jupyter nbconvert --execute` не нужен ---
все 6 ноутбуков уже содержат результаты и графики.

Можно открыть любой ноутбук в Jupyter для проверки:

```bash
jupyter notebook notebooks/daisyworld_literate.ipynb
```

> **Что говорить на видео:**
> «Все ноутбуки уже выполнены --- `Literate.notebook` делает это
> автоматически при генерации. Откроем один для проверки ---
> видим код, текст и графики.»

---

## Шаг 13. Оформление и рендеринг отчёта

### 13.1. Структура отчёта (`report/simulation-modeling--lab03--report.qmd`)

1. **Цель работы** --- изучение агентного моделирования
2. **Задание** --- по пунктам из учебника
3. **Теоретическое введение**:
   - Агентный подход (ABM): агенты, среда, взаимодействия
   - Принципы: эмерджентность, автономия, гетерогенность, локальность
   - Модель Daisyworld: гипотеза Геи, параметры, динамика
4. **Выполнение** (для каждого скрипта):
   - Код (фрагменты)
   - Результаты (графики из `plots/`)
   - Скриншоты генерации литературных форматов
5. **Анализ результатов с параметрами**
6. **Выводы**
7. **Список литературы**

### 13.2. Рендеринг

```bash
cd labs/lab03/report
quarto render simulation-modeling--lab03--report.qmd
```

> **Что говорить на видео:**
> «Рендерим отчёт. Quarto формирует PDF с содержанием,
> нумерацией рисунков, формулами и библиографией по ГОСТу.»

---

## Шаг 14. Оформление презентации

Слайды:

1. Информация о докладчике
2. Цель работы
3. Агентное моделирование --- определение и принципы
4. Модель Daisyworld --- описание
5. Создание проекта DrWatson
6. Тепловая карта (скриншот)
7. Анимация (ссылка на видео)
8. Динамика числа маргариток (скриншот)
9. Полная динамика с ramp-сценарием (скриншот)
10. Литературное программирование --- генерация форматов
11. Результаты с параметрами
12. Выводы

```bash
cd labs/lab03/presentation
quarto render simulation-modeling--lab03--presentation.qmd
```

> **Что говорить на видео:**
> «Презентация: теория ABM, Daisyworld, ключевые визуализации, выводы.»

---

## Шаг 15. Фиксация в git

```bash
cd labs/lab03
git add .
git commit -m "feat(lab03): добавить лабораторную работу №3"
git push
```

> **Что говорить на видео:**
> «Фиксируем результаты в git. Согласно заданию,
> результирующие файлы не удаляем. Коммитим и пушим.
> Лабораторная выполнена.»

---

## Сводка: итоговая структура проекта

```
labs/lab03/
├── lab_03_daisyworld/                             # проект DrWatson
│   ├── Project.toml
│   ├── Manifest.toml
│   ├── src/
│   │   └── daisyworld.jl                         # МОДЕЛЬ (подключается из всех скриптов)
│   ├── scripts/
│   │   ├── daisyworld.jl                          # (1) базовая визуализация
│   │   ├── daisyworld_literate.jl                 #     → литературная версия
│   │   ├── daisyworld-animate.jl                  # (2) анимация (без Literate-цикла)
│   │   ├── daisyworld-count.jl                    # (3) динамика числа
│   │   ├── daisyworld-count_literate.jl           #     → литературная версия
│   │   ├── daisyworld-luminosity.jl               # (4) полная динамика
│   │   ├── daisyworld-luminosity_literate.jl      #     → литературная версия
│   │   ├── daisyworld__param.jl                   # (5) визуализация с параметрами
│   │   ├── daisyworld__param_literate.jl          #     → литературная версия
│   │   ├── daisyworld-count__param.jl             # (6) динамика с параметрами
│   │   ├── daisyworld-count__param_literate.jl    #     → литературная версия
│   │   ├── daisyworld-luminosity__param.jl        # (7) полная динамика с параметрами
│   │   ├── daisyworld-luminosity__param_literate.jl  # → литературная версия
│   │   └── generate.jl                            # скрипт генерации всех форматов
│   ├── notebooks/                                 # 6 сгенерированных .ipynb
│   │   ├── daisyworld_literate.ipynb
│   │   ├── daisyworld-count_literate.ipynb
│   │   ├── daisyworld-luminosity_literate.ipynb
│   │   ├── daisyworld__param_literate.ipynb
│   │   ├── daisyworld-count__param_literate.ipynb
│   │   └── daisyworld-luminosity__param_literate.ipynb
│   ├── docs/                                      # 6 сгенерированных .md/.qmd
│   │   ├── daisyworld_literate.md
│   │   ├── daisyworld-count_literate.md
│   │   ├── daisyworld-luminosity_literate.md
│   │   ├── daisyworld__param_literate.md
│   │   ├── daisyworld-count__param_literate.md
│   │   └── daisyworld-luminosity__param_literate.md
│   └── plots/                                     # все графики
│       ├── daisy_step001.png
│       ├── daisy_step005.png
│       ├── daisy_step040.png
│       ├── simulation.mp4
│       ├── daisy_count.png
│       ├── daisy_luminosity.png
│       └── daisyworld_*.png / daisy-count_*.png / daisy-luminosity_*.png  # с параметрами
├── report/
│   ├── _quarto.yml
│   ├── simulation-modeling--lab03--report.qmd
│   ├── bib/cite.bib
│   ├── image/
│   └── _output/
├── presentation/
│   ├── _quarto.yml
│   ├── simulation-modeling--lab03--presentation.qmd
│   ├── image/
│   └── _resources/
└── INSTRUCTIONS.md                                # <-- этот файл
```

---

## Сводная таблица: что генерируется из чего

| Исходный скрипт | Литературный | Чистый .jl | .ipynb | .qmd/.md |
|-----------------|-------------|-----------|--------|----------|
| `daisyworld.jl` | `daisyworld_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |
| `daisyworld-animate.jl` | --- | --- | --- | --- |
| `daisyworld-count.jl` | `daisyworld-count_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |
| `daisyworld-luminosity.jl` | `daisyworld-luminosity_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |
| `daisyworld__param.jl` | `daisyworld__param_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |
| `daisyworld-count__param.jl` | `daisyworld-count__param_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |
| `daisyworld-luminosity__param.jl` | `daisyworld-luminosity__param_literate.jl` | `Literate.script` | `Literate.notebook` | `Literate.markdown` |

**Итого: 7 исходных скриптов → 6 литературных → 18 производных файлов (6 чистых + 6 ipynb + 6 qmd)**

---

## Ключевой workflow

```
src/daisyworld.jl (модель) ──► include() из всех скриптов

Для каждого скрипта (кроме animate):
  scripts/X.jl (исходный)
       │
       ▼ запустить, проверить
       │
  scripts/X_literate.jl (добавить # разметку)
       │
       ├──► Literate.script()    → scripts/X_literate.jl  (чистый)
       ├──► Literate.notebook()  → notebooks/X_literate.ipynb
       └──► Literate.markdown()  → docs/X_literate.md
                                        │
                                        ▼
                              (ноутбук выполняется автоматически)
                                        │
                                        ▼
                              интеграция .qmd в отчёт
```

> **Что говорить на видео (заключение):**
> «Подведём итоги. В ходе лабораторной работы мы:
>
> Первое --- изучили агентный подход к имитационному моделированию.
> ABM позволяет исследовать сложные системы, где глобальное поведение
> возникает из взаимодействия множества автономных агентов.
>
> Второе --- реализовали модель Daisyworld на Julia с помощью Agents.jl.
> Чёрные и белые маргаритки на сетке 30 на 30 регулируют температуру
> через различное альбедо.
>
> Третье --- для каждого из шести скриптов прошли полный цикл
> литературного программирования: исходный скрипт, литературная версия,
> генерация трёх форматов --- чистый Julia-код, Jupyter notebook
> и Quarto-документация. Всего получили 18 производных файлов.
>
> Четвёртое --- выполнили все 6 ноутбуков
> и интегрировали документацию в отчёт.
>
> Пятое --- провели эксперименты с параметрами: варьировали
> максимальный возраст и начальную долю белых маргариток.
>
> Ключевое наблюдение: Daisyworld демонстрирует эмерджентную саморегуляцию ---
> простые локальные правила приводят к глобальной стабилизации климата.
>
> Спасибо за внимание!»
