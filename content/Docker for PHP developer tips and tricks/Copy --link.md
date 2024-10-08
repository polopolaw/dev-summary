Документация: https://docs.docker.com/reference/dockerfile/#copy---link

**Кратко** `COPY --link <source> <destination>` означает, что Docker создаёт специальную ссылку, указывающую на исходные файлы. Это экономит место и ускоряет процесс сборки Docker-образа.

**Примеры использования:** Когда вы используете один и тот же файл в нескольких местах и хотите его скопировать, но при этом уменьшить размер образа, можно использовать метод жесткой ссылки с помощью команды `COPY --link`.

**Преимущества:**

- **Снижение использования дискового пространства:** Уменьшает объём используемого пространства во время сборки благодаря использованию жесткой ссылки.
- **Ускорение сборки:** `COPY --link` ускоряет сборку, избегая реального копирования файлов, особенно для крупных файлов или проектов с множеством зависимостей.
- **Оптимизация размера слоёв:** `COPY --link` помогает держать слои Docker компактными, избегая дублирования данных, что ведёт к уменьшению размера образов.
- **Минимизация избыточности данных:** Обеспечивает хранение каждого файла только один раз, даже если он используется на нескольких этапах сборки.
- **Согласованность между этапами:** Все этапы сборки, использующие один и тот же файл, будут работать с одной и той же версией данных.
- **Экологичность:** Снижение объёма хранения и мощности процессора в сборках Docker может уменьшить энергопотребление в дата-центрах.
- **Упрощение управления образами:** Меньшие образы легче управлять, передавать и хранить, что особенно полезно в средах с ограниченной пропускной способностью или ресурсами хранения.
- **Экономичность в облачных средах:** В облачных средах с оплатой за использование ресурсов оптимизация пространства и времени сборки может привести к снижению затрат.
- **Увеличение производительности CI/CD:** Быстрая сборка и меньшие образы могут улучшить производительность CI/CD, ускоряя деплой и тестирование.
- **Резервное копирование обычным копированием:** Если создание жесткой ссылки невозможно, Docker автоматически вернётся к обычному копированию, обеспечивая непрерывность процесса сборки.