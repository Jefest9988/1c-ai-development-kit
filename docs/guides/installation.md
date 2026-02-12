# Руководство по установке

## Системные требования

### Обязательные

- **Cursor IDE** версия 0.40+ (с поддержкой MCP)
- **1С:Предприятие 8.3** (любая редакция: базовая, КОРП, УТ и т.д.)
- **PowerShell 5.1+** (для Windows) или **Bash** (для Linux/macOS)
- **Git** 2.20+

### Рекомендуемые

- **Docker** (для запуска MCP-серверов локально)
- **Python 3.10+** (для некоторых утилит)
- **Node.js 18+** (для некоторых MCP-серверов)

## Установка базового набора

### Шаг 1: Клонирование репозитория

```bash
# Клонируйте репозиторий
git clone https://gitea.yourdomain.com/yourname/1c-ai-development-kit.git
cd 1c-ai-development-kit
```

### Шаг 2: Интеграция в существующий проект

Если у вас уже есть проект 1С:

```bash
# Скопируйте агентов
cp -r .cursor/agents/* /path/to/your-project/.cursor/agents/

# Скопируйте правила
cp -r .cursor/rules/* /path/to/your-project/.cursor/rules/

# Скопируйте навыки
cp -r .cursor/skills/* /path/to/your-project/.cursor/skills/

# Скопируйте команды OpenSpec
cp -r .cursor/commands/* /path/to/your-project/.cursor/commands/
```

### Шаг 3: Создание нового проекта

Если вы начинаете с нуля, используйте навык `1c-project-init`:

```bash
# В Cursor IDE откройте новую папку и выполните:
# "Инициализируй проект 1С из базы Srvr='server'; Ref='database'"
```

Навык автоматически:
- Создаст структуру папок
- Скопирует все навыки из 1c-ai-development-kit
- Определит версию 1С
- Создаст .1c-devbase.bat для работы с базой
- Выгрузит конфигурацию и расширения в XML
- Настроит OpenSpec
- Инициализирует Git

## Настройка MCP-серверов

### Бесплатные MCP-серверы

#### BSL LSP Bridge

**Назначение:** Интеграция с BSL Language Server для анализа кода

**Автор:** [Vladimir Akimov (SteelMorgan)](https://github.com/SteelMorgan)

**Установка:**

1. Клонируйте репозиторий:
```bash
git clone https://github.com/SteelMorgan/mcp-bsl-lsp-bridge.git
cd mcp-bsl-lsp-bridge
```

2. Установите зависимости:
```bash
npm install
```

3. Скачайте BSL Language Server:
```bash
# Скачайте jar файл с https://github.com/1c-syntax/bsl-language-server/releases
# Положите в папку bsl-lsp-bridge/
```

4. Запустите сервер:
```bash
node server.js
```

5. Добавьте в `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "bsl-lsp-bridge": {
      "command": "curl",
      "args": [
        "-X", "POST",
        "http://localhost:5007/mcp",
        "-H", "Content-Type: application/json",
        "-d", "@-"
      ]
    }
  }
}
```

#### RLM Toolkit

**Назначение:** Память между чатами, сохранение контекста

**Установка:**

1. Клонируйте репозиторий:
```bash
git clone https://github.com/yourusername/rlm-toolkit.git
cd rlm-toolkit
```

2. Установите зависимости:
```bash
pip install -r requirements.txt
```

3. Запустите сервер:
```bash
python server.py
```

4. Добавьте в `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "rlm-toolkit": {
      "command": "curl",
      "args": [
        "-X", "POST",
        "http://localhost:8200/mcp",
        "-H", "Content-Type: application/json",
        "-d", "@-"
      ]
    }
  }
}
```

### Платные MCP-серверы

Платные MCP-серверы предоставляются по подписке. Они включают:

- `1c-help` — документация 1С
- `1c-ssl` — БСП (Библиотека Стандартных Подсистем)
- `1c-templates` — шаблоны кода
- `1c-syntax-checker` — проверка синтаксиса BSL
- `1c-code-checker` — проверка логики (1С:Напарник)
- `1c-forms` — схемы управляемых форм

**Получение доступа:**

1. Приобретите на [vibecoding1c.ru/mcp_server](https://vibecoding1c.ru/mcp_server)
2. Получите Docker-контейнеры и инструкции по развертыванию
3. Запустите контейнеры на своем сервере (Linux/Windows/macOS)
4. Настройте свои API-ключи:
   - OpenAI/Anthropic (для embeddings) — или используйте локальные модели
   - 1С:Напарник (для проверки кода) — ваш ключ
   - Neo4j (для графового поиска) — локальная установка
5. Добавьте серверы в `.cursor/mcp.json` (используйте шаблон из `templates/mcp.json`)

**Важно:**
- ✅ **Не SaaS** — вы разворачиваете у себя
- ✅ **Ваши данные** — ничего не уходит на сторонние серверы
- ✅ **Ваши ключи** — используете свои API-ключи
- ✅ **Локальные модели** — можно использовать LMStudio, Ollama, Qwen
- ✅ **Полный контроль** — настраиваете под свои нужды

**Автор MCP:** Олег Филиппов ([@comol_foa](https://t.me/comol_foa))  
**Документация:** [vibecoding1c.ru](https://vibecoding1c.ru/)  
**Сообщество:** [t.me/comol_it_does_matter](https://t.me/comol_it_does_matter)

### Проектные MCP-серверы

Для каждого проекта 1С можно создать специализированные MCP-серверы:

1. **Metadata & Code Search MCP** — семантический поиск по вашей конфигурации
2. **Graph MCP** — граф зависимостей объектов

См. [Создание проектных MCP](project-mcp-setup.md) для деталей.

## Проверка установки

### Проверка агентов

В Cursor IDE:

1. Откройте Command Palette (Ctrl+Shift+P)
2. Найдите "Agent: List Available Agents"
3. Убедитесь, что видите агентов: `onec-code-writer`, `onec-code-reviewer`, и т.д.

### Проверка правил

1. Откройте любой файл `.bsl`
2. Начните писать код
3. AI должен следовать стандартам из `.cursor/rules/1c-coding-standards.mdc`

### Проверка навыков

1. В чате Cursor напишите: "Покажи доступные навыки"
2. AI должен перечислить навыки из `.cursor/skills/`

### Проверка MCP-серверов

1. В чате Cursor напишите: "Найди документацию по методу СтрРазделить"
2. Если `1c-help` MCP настроен, AI найдет документацию
3. Если нет — предложит установить MCP

## Решение проблем

### Агенты не видны

**Проблема:** Cursor не видит агентов в `.cursor/agents/`

**Решение:**
1. Перезапустите Cursor (Ctrl+Shift+P → "Reload Window")
2. Проверьте формат файлов агентов (должны быть `.md` с frontmatter)
3. Проверьте права доступа к файлам

### MCP-серверы не работают

**Проблема:** Ошибка "MCP server not responding"

**Решение:**
1. Проверьте, что сервер запущен: `curl http://localhost:5007/health`
2. Проверьте порты в `.cursor/mcp.json`
3. Проверьте логи сервера

### Навыки не выполняются

**Проблема:** AI не использует навыки

**Решение:**
1. Проверьте, что файлы `SKILL.md` на месте
2. Проверьте права выполнения для скриптов (`.ps1`, `.bat`)
3. Попросите AI явно: "Используй навык 1c-forms/compile"

## Следующие шаги

- [Настройка MCP-серверов](mcp-setup.md)
- [Работа с агентами](agents.md)
- [Использование навыков](skills.md)
- [Первый проект](first-project.md)
