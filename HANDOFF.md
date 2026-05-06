# Mono Checkout Prototype — состояние проекта

Прототип чекаута/корзины для Mono Market на украинском. Single-file `index.html` + `assets/` папка, без бандлера. iPhone 13 mini (375×812) внутри фейковой рамки на тёмном фоне.

## Репозиторий и деплой
- GitHub: `https://github.com/OceanDeath/mono-checkout-prototype` (юзер `OceanDeath` с большой O)
- Live: `https://oceandeath.github.io/mono-checkout-prototype/` (нижний регистр в URL — особенность GitHub Pages)
- В корне есть `.nojekyll` чтобы GitHub Pages не запускал Jekyll и билд проходил быстро
- Локальная папка пользователя: `~/Documents/mono-checkout-prototype`
- Деплой стандартный: `cp ~/Downloads/checkout-repo/index.html ./index.html && git add -A && git commit -m "..." && git push origin main`

## Готовые экраны (6 штук, все wired)
Все живут как `<div class="screen ...">` внутри `.phone`. Активный получает `.active`. Переключение через `showScreen(name)` — поддерживаемые имена: `cart` | `checkout` | `pickup` | `np` | `success` | `details`.

### 1. Cart (`#cartScreen`) — стартовый, active по дефолту
- 3 товара: Logitech MX Master 3S (3999, Фокстрот), PS5 Slim (22999, KTC, -15% от 26999), Asus TUF F15 (31199, KTC, -15% от 44999)
- Все выбраны, общая сумма `58 197 ₴`, без мыши `54 198 ₴`
- Хедер с share, лимит-баннер, плавающий таб-бар внизу

### 2. Checkout (`#checkoutScreen`)
- Hero с синим градиентом, при скролле сжимается (top-chrome получает тень)
- Cart-card (синяя зона), способ оплаты (Частинами/Повністю — сегмент), карточка mono
- Город + сегмент Доставка/Самовивіз
- Опции доставки: до відділення (NP) / кур'єром, можна сменить на самовывоз
- Самовывоз: `state-pickup` модификатор скрывает мышь Logitech (Фокстрот не работает с самовывозом)
- При активном выборе адреса (NP-отделение или магазин) кнопка «Продовжити» становится активной → открывает payment modal

### 3. Pickup (`#pickupScreen`) — список магазинов KTC
Хедер 136px белый, статусбар `filter: invert(1)`, список магазинов с пинами

### 4. NP (`#npScreen`) — список отделений Нової Пошти
Хедер 136px с лого NP. Список с 6 элементами: иконка-квадратик 40×40 (icon_box / icon_post / icon_truck — каждый SVG имеет встроенный серый фон). Кнопка «Зберегти» обновляет адрес в чекауте.

### 5. Payment Modal (внутри `#checkoutScreen`)
Bottom-sheet 440px: drag-handle, заголовок «Підтвердіть оплату» + «Згорнути», 2 карты на выбор (`.pay-card-row` — НЕ `.payment-card`! последний используется в чекауте для другого блока), info-row, кнопка «Сплатити повністю: 58 197 ₴».
- Чорна картка — выбрана по дефолту, баланс 50 450.25
- Біла картка — баланс 29 191.87
- При клике на «Сплатити повністю» — закрывает модалку и через 200ms показывает success + конфетти

### 6. Success (`#successScreen`)
- Фиолетовый градиент `#4760e7 → #8469d5`
- Cat illustration `cat_success.svg` 331×268 с speech-bubble «Вітаю з покупкою!»
- Подзаголовок белый opacity 0.7
- Order-card 327×140: счётчик/сумма, миниатюры товаров (gap 8px, без теней), капсула «Переглянути замовлення»
- Чёрная кнопка «Повернутися на головну»
- **Confetti**: 120 частиц на canvas из 3 источников, гравитация + drag + sin-покачивание, ~3.5 сек жизни. Запускается `runConfetti()` при показе экрана.
- **Динамический контент** через `syncSuccessOrderCard()`: читает `#cartQty` и `#cartPrice` (которые обновляются в чекауте при `state-pickup`), скрывает 3-ю миниатюру (мышь) если pickup. Так получается:
  - Доставка НП: «3 товари / 58 197 ₴ / 3 миниатюры»
  - Самовивіз: «2 товари / 54 198 ₴ / 2 миниатюры»

### 7. Details (`#detailsScreen`)
- Светло-серый фон `#f4f4f4`, скроллящийся список из 4 карточек
- **Шапка collapse on scroll** (по figma 6345:8366):
  - Header 96px только со status-bar + back (большой title и дата теперь в потоке списка как `.details-headline`)
  - При `scrollTop > 8` шапка получает `.collapsed` → тень снизу + проявляется compact-title 16/700 центрированный с `letter-spacing: -0.3px`
  - При входе на экран `.collapsed` сбрасывается + scroll → 0
- Карточка статуса: status-block (icon + name + eta + chevron) → `.details-sep` → 2 товара (item-row 96px, item-name 14/600 max 2 строки, цена 18/700, миниатюра 64×64 справа)
- Карточка получения: 4 info-row через `.details-sep` — Продавець (КТС + лого) / Самовивіз з магазину (адрес + sub-caption «з 10:00 до 21:00» внутри того же блока) / Одержувач / Телефон
- Карточка оплаты: 1 info-row «Повна оплата / З чорної картки» + миниатюра карты
- Карточка действий: «Скасувати замовлення» (синий круг с ✕) + «Поставити запитання» (синий круг с message через CSS mask)
- info-row padding: `12px 16px` (поджато), value без margin-top (плотный стек caption + value)

## Архитектура CSS
- Всё absolute-позиционированием по координатам Figma (`left`/`top`/`width`/`height` в px)
- Карточки: `margin: 0 16px` для горизонтального центрирования внутри 375
- Сепараторы внутри карточек: `<div class="details-sep">` — 1px серая линия с `margin: 0 16px` (НЕ под обрез карточки)
- Шрифты: SF Pro Text (default), SF Pro Display для больших Bold-заголовков
- Letter-spacing для 24/700 заголовков: `-0.36px`
- Continue-button gradient: `linear-gradient(to right, #3083ff 0%, #306aff 100%)`
- Modal pattern: `.X-modal` оверлей с `.backdrop` 60% black + sheet с translateY(100%→0), transition 0.3s cubic-bezier(.2,.7,.2,1)

## Конвенции работы
- **Comments в коде на русском** для readability
- **Figma node-id в CSS-комментариях**: «figma 6343:8230», «figma 6345:8366»
- **Свежие assets** скачиваются из Figma (через `Figma:get_design_context` MCP), кладутся в `/home/claude/work/assets/`
- **Сборка outputs**:
  ```
  rm -rf /mnt/user-data/outputs/checkout-repo /mnt/user-data/outputs/checkout-repo.zip && \
  mkdir -p /mnt/user-data/outputs/checkout-repo && \
  cp -r /home/claude/work/* /mnt/user-data/outputs/checkout-repo/ && \
  cp /home/claude/work/.nojekyll /mnt/user-data/outputs/checkout-repo/ && \
  cd /mnt/user-data/outputs && \
  zip -qr checkout-repo.zip checkout-repo
  ```
  Затем `present_files` с `[index.html, checkout-repo.zip]`
- **Пользователь общается на ты, по-русски, отвечает кратко**
- **Скриншот после деплоя обычно с показом проблемы** → стиль работы: точечные правки по скриншотам

## Asset inventory
В `/home/claude/work/assets/`:
- product_cart_1.png (PS5), product_cart_2.png (laptop), product_cart_3.png (Logitech mouse)
- KTC_logo.png, Foxtrot_logo.png
- NP_Logo_Round_24x24.svg, NP_Badge.svg
- Viddilennya.png, Courier.png, Location_edit.svg
- icon_box.svg, icon_post.svg, icon_truck.svg (с встроенным серым фоном)
- back_icon_blue.svg, Chevron_Right_MD.svg
- battery.svg, wifi.svg, cellular.svg, home_indicator.svg
- black_card_mc_small.svg, white_card_visa_small.svg, info_small.svg
- cat_success.svg
- Status_waiting.svg
- Add.svg, Bazar_Icon.svg, Cabinet_icon.svg, Delete.svg, Lapka.svg, Share.svg, cards_icon.svg, market_icon.svg

## Полный пользовательский флоу
1. Корзина → выбрать товары → «Продовжити»
2. Чекаут → (опционально) переключить на самовывоз → выбрать адрес NP-отделения или KTC-магазина → кнопка «Продовжити» активна
3. Нажать «Продовжити» → выезжает модалка оплаты
4. Выбрать карту → «Сплатити повністю»
5. Success-экран с конфетти и order-card
6. «Переглянути замовлення» → details со схлопывающейся шапкой
7. Back → возврат на success
8. «Повернутися на головну» (с success) → возврат в корзину

## Что осталось / возможные следующие задачи
- Адаптивность под мобильный viewport (раньше пробовали через `transform: scale()`, откатили — нужна нормальная адаптивность через переделку фиксированных width/left/top, или scale + max-width гибрид)
- Theme-color для Safari address bar чтобы убрать шов между статусбаром iOS и нашим хедером
- Возможно дополнительные экраны и состояния
