# 📘 Интеграция пользовательского UI и конфигурации в Dynamics 365

## 🧭 Цель документа

Подробно рассмотреть все способы встраивания пользовательского интерфейса (UI) в Dynamics 365, а также варианты хранения и извлечения конфигурации для таких UI.

---

## 🔷 1. Варианты интеграции пользовательского UI в Dynamics 365

### ✅ 1.1. Web Resource (HTML/JS/React)

**Описание:**

* Это наиболее прямой способ вставки UI. Web Resource — это HTML или JS-файл, загружаемый в систему как ресурс.
* Можно подключить к формам, панелям, встраивать в iframe.

**Варианты использования:**

* React/Vue интерфейсы
* Настройки интеграции
* Визуализация данных

**Где подключается iframe Web Resource:**

* Открой форму нужной сущности в **Form Designer**
* Нажми **Insert → Web Resource**
* Выбери загруженный HTML-файл (например `settings.html`)
* Установи флаг "Display as iframe" и задай размеры (width/height)
* Укажи имя и описание компонента

**Добавление на форму:**

1. Создайте Web Resource (`settings.html`, `settings.js`, `bundle.js`)
2. Добавьте на форму как `iframe` или `web resource control`

**Пример iframe:**

```html
<iframe src="/WebResources/your_custom_ui.html" width="100%" height="400px" frameborder="0"></iframe>
```

**🧱 Как взаимодействовать с системой:**

```ts
Microsoft.CIFramework.getEnvironment().then((env) => {
  console.log("Region:", env["region"]);
});

Microsoft.CIFramework.setSession("genesys_config", JSON.stringify({
  region: "prod", clientId: "abc"
}));
```

**Пример HTML-файла:**

```html
<html>
  <body>
    <h2>Genesys Settings</h2>
    <label>Region: <input id="region" /></label><br />
    <label>Client ID: <input id="clientId" /></label><br />
    <button onclick="save()">Save</button>
  </body>
  <script>
    function save() {
      const config = {
        region: document.getElementById('region').value,
        clientId: document.getElementById('clientId').value
      };
      localStorage.setItem("genesys_settings", JSON.stringify(config));
    }
  </script>
</html>
```

**Когда использовать:**

* Нужен React или кастомный HTML-интерфейс
* Нужна интеграция с внешними API
* Не используется Power FX

---

![image](https://github.com/user-attachments/assets/3ae9ee59-b1fa-4509-a2e8-9046ec955e1e)

---

### ✅ 1.2. Power Apps Custom Page

**Описание:**

* Low-code/no-code UI, создаётся прямо в Power Platform.
* Можно встраивать в навигацию, формы или вызывать через команды.

**Особенности:**

* Поддерживает PowerFX, интеграцию с Dataverse, responsive-дизайн.
* Можно передавать параметры и вызывать Flows.

**Когда использовать:**

* Для построения административных интерфейсов
* Таблицы, формы, визуальные редакторы

![image](https://github.com/user-attachments/assets/c0787776-fd8a-495c-986b-b0737642109f)

---

![image](https://github.com/user-attachments/assets/a9ad7a30-067c-4780-a630-c9944ca31fe4)

---

### ✅ 1.3. PCF (PowerApps Component Framework)

**Описание:**

* Компоненты на TypeScript + React, встраиваются как контролы в формы или представления.
* Идеальны для визуализации, таблиц, визуальных редакторов, кнопок и др.

**Преимущества:**

* Работают как "родные" компоненты
* Полная интеграция в форму

**Минусы:**

* Требует разработки и упаковки через CLI
* Требует knowledge в Webpack, CDS SDK

---

![image](https://github.com/user-attachments/assets/3383127a-9561-4709-b9bd-4cdc7be17310)

---

![image](https://github.com/user-attachments/assets/96799da3-2de4-45b1-91da-c6589a3a1b10)


---

## 🔷 2. Хранение конфигурации UI в Dynamics 365

### ✅ Вариант 1: Custom Entity

Создай сущность `GenesysSettings` или `CustomIntegrationConfig`.

**Поля:**

* `Name` (строка)
* `Region` (опция или строка)
* `ClientId` (строка)
* `StatusMapping` (многострочный текст, JSON)

**Преимущества:**

* Персистентное хранение
* Возможность использовать security roles
* Можно редактировать в UI и через API

**Получение данных:**

```ts
Microsoft.CIFramework.invokeProcess("RetrieveRecord", {
  entityLogicalName: "your_custom_entity",
  id: "<record-guid>"
}).then(response => {
  const config = JSON.parse(response.StatusMapping);
});
```

### ✅ Вариант 2: Environment Variables

* Создаются в Power Platform Admin Center
* Можно читать через `CIFramework` + `WebAPI`
* Поддерживает разные environments (dev/test/prod)

**Пример запроса:**

```http
GET /api/data/v9.0/environmentvariabledefinitions
```

### ✅ Вариант 3: Прямое хранение в локальном хранилище (если UI внешний)

```ts
localStorage.setItem("genesys_settings", JSON.stringify({ region: "prod-euw1", clientId: "abc123" }));
```

---

## 📦 Структура возможной конфигурации (JSON)

```json
{
  "region": "prod-euw1",
  "clientId": "xyz-123",
  "statusMapping": {
    "Available": "Ready",
    "Away": "Break",
    "Busy": "Not Ready"
  },
  "features": {
    "logging": true,
    "debug": false
  }
}
```

---

## 📦 Структура возможной конфигурации (JSON)

```json
{
  "region": "prod-euw1",
  "clientId": "xyz-123",
  "statusMapping": {
    "Available": "Ready",
    "Away": "Break",
    "Busy": "Not Ready"
  },
  "features": {
    "logging": true,
    "debug": false
  }
}
```

---

## ✅ Резюме

| Способ               | Подходит для                                | Комментарий                                |
| -------------------- | ------------------------------------------- | ------------------------------------------ |
| Web Resource         | React-интерфейс                             | Максимальная свобода, простота встраивания |
| Custom Page          | Простые визуальные редакторы                | Быстро и нативно для админов               |
| PCF                  | Визуализация и взаимодействие внутри формы  | Глубокая интеграция в D365                 |
| Custom Entity        | Конфигурация, управляемая из UI             | Можно связать с ролью, API, UI             |
| Environment Variable | Общие настройки, переключаемые по окружению | Хорошо для продакшена                      |
| localStorage         | Быстро и локально                           | Только если iframe внешний                 |

---
