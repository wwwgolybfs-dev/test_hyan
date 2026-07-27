# Инструкция по настройке системы Petzker Cookware с мини-играми

## 📋 Содержание

1. **Структура файлов**
2. **Настройка GitHub Pages**
3. **Обновление базы данных Supabase**
4. **Подключение игр в главный файл**
5. **Развертывание приложения**

---

## 1️⃣ Структура файлов

Вам понадобятся следующие файлы:

```
petzker-cookware/
├── index.html              # Главный файл (календарь + дневник + меню игр)
├── dice.html               # Игра с кубиками
├── minesweeper.html        # Классический минёр
├── doodle.html             # Дудл Джамп
├── throne.html             # Терновый трон
├── README.md               # Этот файл
└── .gitignore
```

---

## 2️⃣ Настройка GitHub Pages

### Шаг 1: Создайте репозиторий на GitHub

1. Перейдите на https://github.com/new
2. Назовите репозиторий `petzker-cookware` (или любое другое имя)
3. Выберите "Public"
4. Создайте репозиторий

### Шаг 2: Загрузите файлы

```bash
# Клонируйте репозиторий
git clone https://github.com/YOUR_USERNAME/petzker-cookware.git
cd petzker-cookware

# Скопируйте все HTML файлы в папку проекта

# Добавьте файлы
git add .
git commit -m "Initial commit: Add Petzker Cookware with mini-games"
git push origin main
```

### Шаг 3: Включите GitHub Pages

1. Перейдите в **Settings** вашего репозитория
2. Прокрутите вниз до **Pages**
3. В разделе "Source" выберите **main branch**
4. Сохраните

**Ваше приложение будет доступно по адресу:**
```
https://YOUR_USERNAME.github.io/petzker-cookware/
```

---

## 3️⃣ Обновление базы данных Supabase

### Создание таблиц для игр

Откройте **SQL Editor** в Supabase и выполните следующие команды:

#### Таблица для рейтинга Минёра

```sql
CREATE TABLE minesweeper_scores (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  telegram_user_id TEXT NOT NULL,
  name TEXT NOT NULL,
  difficulty TEXT NOT NULL,
  time INT NOT NULL,
  completed_at TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_minesweeper_difficulty ON minesweeper_scores(difficulty);
CREATE INDEX idx_minesweeper_user ON minesweeper_scores(telegram_user_id);
```

#### Таблица для игры с кубиками

```sql
CREATE TABLE dice_game_scores (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  telegram_user_id TEXT NOT NULL,
  name TEXT NOT NULL,
  score INT NOT NULL,
  completed_at TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_dice_user ON dice_game_scores(telegram_user_id);
```

#### Обновление таблицы vacations (добавьте если её нет)

```sql
CREATE TABLE IF NOT EXISTS vacations (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  day TEXT NOT NULL,
  name TEXT NOT NULL,
  telegram_user_id TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_vacations_day ON vacations(day);
```

#### Таблица для дневника

```sql
CREATE TABLE IF NOT EXISTS diary_entries (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  text TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Настройка Row Level Security (RLS)

Для обеспечения безопасности включите RLS:

```sql
-- Включите RLS для всех таблиц
ALTER TABLE vacations ENABLE ROW LEVEL SECURITY;
ALTER TABLE diary_entries ENABLE ROW LEVEL SECURITY;
ALTER TABLE minesweeper_scores ENABLE ROW LEVEL SECURITY;
ALTER TABLE dice_game_scores ENABLE ROW LEVEL SECURITY;

-- Разрешите всем читать и писать
CREATE POLICY "Enable all operations for authenticated users" ON vacations
  FOR ALL USING (true) WITH CHECK (true);

CREATE POLICY "Enable all operations for diary" ON diary_entries
  FOR ALL USING (true) WITH CHECK (true);

CREATE POLICY "Enable all operations for minesweeper" ON minesweeper_scores
  FOR ALL USING (true) WITH CHECK (true);

CREATE POLICY "Enable all operations for dice" ON dice_game_scores
  FOR ALL USING (true) WITH CHECK (true);
```

---

## 4️⃣ Подключение игр в главный файл

### Обновите переменную GAMES_BASE_URL

В файле **index.html** найдите строку:

```javascript
const GAMES_BASE_URL = 'https://wwwgolybfs-dev.github.io/petzker-games/';
```

Замените на ваш URL:

```javascript
const GAMES_BASE_URL = 'https://YOUR_USERNAME.github.io/petzker-cookware/';
```

### Проверьте Supabase конфигурацию

Убедитесь, что в обоих файлах используются одинаковые учетные данные:

```javascript
const SUPABASE_URL = 'https://heubrattlnikielnfheg.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## 5️⃣ Развертывание приложения

### Через Telegram Bot

1. **Откройте BotFather в Telegram**: `@BotFather`
2. **Создайте новый бот** (если его еще нет)
3. **Получите токен и Chat ID**
4. **В настройках бота включите Web App**:
   - Используйте URL: `https://YOUR_USERNAME.github.io/petzker-cookware/`

### Через Direct Link

Просто поделитесь ссылкой:
```
https://YOUR_USERNAME.github.io/petzker-cookware/
```

---

## 🎮 Описание игр

### 🎲 Кубики
- **Механика**: Размещайте блоки на доске
- **Улучшение**: Скрипт анализирует доступные клетки и показывает только те фигурки, которые можно разместить
- **Цель**: Набрать максимальное количество очков

### 💣 Минёр
- **Сложности**: Easy (8×8, 10 мин), Medium (12×12, 30 мин), Hard (16×16, 40 мин)
- **Механика**: Классический Windows Minesweeper
- **Рейтинг**: Сохраняет лучшие времена в Supabase
- **Управление**: ЛКМ - открыть, ПКМ - флаг

### 🧑 Дудл Джамп
- **Механика**: Прыгайте выше, избегайте врагов
- **Персонаж**: Толстый китаец в шортах и майке (подкатана)
- **Управление**: Стрелки или A/D для движения
- **Типы платформ**: Обычные и пружинные (дают бонус)

### 👑 Терновый Трон
- **Механика**: Сбор золота, избегание врагов
- **Управление**: Виртуальный джостик в нижнем левом углу
- **Волны**: Враги становятся быстрее с каждой волной
- **Цель**: Собрать максимум золота до поражения

---

## 🔧 Возможные проблемы

### Игры не загружаются

**Проблема**: CORS ошибка в консоли браузера

**Решение**:
1. Убедитесь, что все файлы находятся в одной папке GitHub Pages
2. Проверьте, что URL в `GAMES_BASE_URL` доступен
3. Попробуйте открыть файл напрямую в браузере

### Supabase не сохраняет данные

**Проблема**: Таблицы не существуют или RLS не настроен

**Решение**:
1. Откройте SQL Editor в Supabase
2. Выполните команды из раздела "Обновление базы данных"
3. Проверьте, что таблицы видны в разделе "Tables"

### Игры работают медленно

**Проблема**: Браузер на мобильном устройстве

**Решение**:
1. Закройте другие вкладки
2. Очистите кеш браузера
3. Используйте последнюю версию браузера

---

## 📱 Интеграция с Telegram Mini App

Чтобы использовать приложение как Telegram Mini App:

1. **Создайте бота** через @BotFather
2. **Получите Bot Token**
3. **В командах бота добавьте Web App**:
   ```
   /webapp - https://YOUR_USERNAME.github.io/petzker-cookware/
   ```
4. **Откройте бота и нажмите на Web App**

Приложение автоматически:
- Получит информацию о пользователе из Telegram
- Включит гаптическую обратную связь
- Покажет интеграцию с системой

---

## 🚀 Дополнительные настройки

### Изменение цветовой схемы

В файле `index.html` найдите `:root` CSS переменные и измените цвета:

```css
:root {
  --blue: #2E7BEE;        /* Основной синий */
  --orange: #F0921A;      /* Акцент оранжевый */
  --green: #22C55E;       /* Зеленый */
  --red: #EF4444;         /* Красный */
}
```

### Добавление новой игры

1. Создайте новый файл `game-name.html`
2. Используйте тот же template структуру
3. Добавьте в меню игр в `index.html`:
   ```html
   <div class="game-card" onclick="selectGame('game-name')">
     <div class="game-card-icon">🎮</div>
     <div class="game-card-title">Название</div>
     <div class="game-card-desc">Описание</div>
   </div>
   ```
4. Добавьте в `selectGame()` функцию URL игры

---

## 📞 Поддержка

Если у вас возникли проблемы:

1. **Проверьте консоль браузера** (F12 → Console)
2. **Посмотрите Supabase логи** в разделе Realtime
3. **Убедитесь в интернет соединении**
4. **Очистите кеш** (Ctrl+Shift+Delete)

---

**Готово! Ваше приложение Petzker Cookware с мини-играми готово к использованию! 🎉**
