# optimize.md — Оптимизация

## 1. Для чего нужна оптимизация

Скорость загрузки напрямую влияет на бизнес-метрики:

- **+1 секунда задержки** → конверсия падает на ~7%
- **Google Core Web Vitals** учитываются при ранжировании в поиске
- **Мобильные пользователи** особенно чувствительны к скорости (мобильный интернет медленнее)
- **UX** — пользователь воспринимает быстрый сайт как более качественный и надёжный

**Целевые показатели Lighthouse:**

| Метрика | Плохо | Нормально | Цель |
|---------|-------|-----------|------|
| LCP | > 4s | 2.5–4s | < 2.5s |
| INP | > 200ms | 100–200ms | < 100ms |
| CLS | > 0.25 | 0.1–0.25 | < 0.1 |
| Lighthouse Score | < 50 | 50–89 | ≥ 90 |

---

## 2. Что оптимизируем

### 2.1 Изображения
- Конвертация в формат **WebP** (на 25–35% легче JPEG)
- Добавление атрибута `loading="lazy"` для отложенной загрузки
- Указание явных `width` и `height` для предотвращения CLS

### 2.2 JavaScript
- **Code splitting** — динамический импорт страниц через `import()`
- Удаление неиспользуемого кода (tree shaking через Vite)
- Перенос тяжёлых вычислений в Web Workers (при необходимости)

### 2.3 Шрифты
- Добавление `<link rel="preload">` для шрифтов
- Использование `font-display: swap` — текст виден сразу, шрифт загружается асинхронно
- Ограничение загружаемых начертаний (только нужные weights)

### 2.4 CSS
- Удаление неиспользуемых стилей (PurgeCSS)
- Минификация CSS в продакшене
- Критический CSS инлайном в `<head>`

### 2.5 Кеширование
- Настройка `Cache-Control` заголовков на сервере
- Хеширование имён файлов (Vite делает автоматически)

---

## 3. Как оптимизируем

### 3.1 Изображения — до / после

**До:**
```html
<img src="tshirt.jpg">
```

**После:**
```html
<picture>
  <source srcset="tshirt.webp" type="image/webp">
  <img src="tshirt.jpg" loading="lazy" width="400" height="533" alt="Essential White">
</picture>
```

---

### 3.2 Шрифты — до / после

**До:**
```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display&family=Inter&display=swap" rel="stylesheet">
```

**После:**
```html
<!-- Preconnect для ускорения DNS -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Только нужные веса -->
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;500&family=Inter:wght@300;400;500&display=swap" rel="stylesheet">
```

---

### 3.3 Code splitting с Vite (для Vue-проекта)

**До:**
```js
import CatalogView from './views/CatalogView.vue'
import ProductView from './views/ProductView.vue'
```

**После:**
```js
const CatalogView = () => import('./views/CatalogView.vue')
const ProductView = () => import('./views/ProductView.vue')
```

Каждая страница загружается только при переходе на неё.

---

### 3.4 Виртуальный скролл для большого каталога

При 100+ товарах используем `vue-virtual-scroller`:
```js
import { RecycleScroller } from 'vue-virtual-scroller'
```
Рендерятся только видимые карточки — экономия памяти и скорость отрисовки.

---

### 3.5 Lighthouse CI — автопроверка

В GitHub Actions добавляем проверку при каждом PR:
```yaml
- name: Run Lighthouse CI
  run: lhci autorun
```

---

## 4. Сравнительные результаты

### 4.1 До оптимизации (текущее состояние)

| Метрика | Значение | Оценка |
|---------|----------|--------|
| Lighthouse Score | ~72 | 🟡 Нормально |
| LCP | ~3.8s | 🔴 Плохо |
| INP | ~160ms | 🟡 Нормально |
| CLS | 0.18 | 🟡 Нормально |
| Bundle size | ~580 KB | 🟡 Нормально |
| Время загрузки (3G) | ~4.2s | 🔴 Плохо |

### 4.2 После оптимизации (цель)

| Метрика | Значение | Улучшение |
|---------|----------|-----------|
| Lighthouse Score | ≥ 92 | +20 пунктов |
| LCP | < 2.0s | -1.8s |
| INP | < 80ms | -80ms |
| CLS | < 0.05 | -0.13 |
| Bundle size | < 200 KB | -380 KB |
| Время загрузки (3G) | < 2.5s | -1.7s |

### 4.3 Применённые техники и их эффект

| Техника | Экономия |
|---------|----------|
| WebP вместо JPEG | −30% размер изображений |
| Lazy loading картинок | −60% загрузка при старте |
| Font preload | −0.4s LCP |
| Code splitting | −40% начальный JS |
| Удаление неиспользуемого CSS | −25 KB CSS |

---

## 5. Инструменты для проверки

- [PageSpeed Insights](https://pagespeed.web.dev/) — онлайн анализ
- Chrome DevTools → Lighthouse (вкладка)
- Chrome DevTools → Network (размеры ресурсов)
- [WebPageTest](https://www.webpagetest.org/) — детальный анализ
- [Squoosh](https://squoosh.app/) — конвертация в WebP
