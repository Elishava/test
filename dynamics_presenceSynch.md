## 🔷 1. Встраивание кастомного интерфейса в Dynamics через iframe

### ✅ Цель

Сделать так, чтобы **UI для настроек или отображения статусов автоматически подгружался** в iframe, встраиваемом через файл `framework.js` из Genesys Embeddable Framework.

### 🧱 Пример:

```ts
const iframe = document.createElement('iframe');
iframe.src = 'https://yourdomain.com/settings.html';
iframe.style.width = '100%';
iframe.style.height = '400px';
iframe.style.border = 'none';
document.body.appendChild(iframe);
```

### 📝 Особенности:

* Такой iframe будет загружаться автоматически
* Пользователь Dynamics не должен ничего настраивать
* `settings.html` — это твой кастомный UI (например, React-страница)

---

## 🔷 2. Хранение настроек (region, clientId, statusMapping и т.д.)

### 📌 Требование: не использовать `Xrm.WebApi`, только `CIFramework`

### ✅ Варианты хранения

| Метод                       | Персистентность | Использует только CIFramework | Подходит? |
| --------------------------- | --------------- | ----------------------------- | --------- |
| `getEnvironment()`          | ❌               | ✅                             | ✅         |
| `setSession` / `getSession` | ❌               | ✅                             | ✅         |
| `localStorage` в iframe UI  | ✅               | ✅                             | ✅         |
| `navigateTo + sharedVar`    | ✅               | ✅                             | ✅\*       |

### 🔹 Пример 1: чтение из параметров URL

```ts
Microsoft.CIFramework.getEnvironment().then((env) => {
  const region = env["region"];
  const clientId = env["clientId"];
});
```

Параметры передаются в URL:

```
https://apps.usw2.pure.cloud/crm/index.html?region=prod-euw1&clientId=abc123
```

### 🔹 Пример 2: сохранение в сессии

```ts
Microsoft.CIFramework.setSession("genesys_settings", JSON.stringify({
  region: "prod-euw1",
  clientId: "abc123"
}));

Microsoft.CIFramework.getSession("genesys_settings").then((value) => {
  const settings = JSON.parse(value);
});
```

### 🔹 Пример 3: использование `localStorage` в iframe

```ts
// Внутри settings.html
localStorage.setItem("genesys_settings", JSON.stringify({ region: "prod", clientId: "123" }));
const settings = JSON.parse(localStorage.getItem("genesys_settings") || "{}");
```

---

## 🔷 3. Синхронизация и изменение статуса агента в Dynamics 365

### ✅ Цель:

* Считывать и изменять статус агента из внешнего интерфейса
* Использовать стандартные и кастомные статусы

### 📌 Возможности изменения статуса

Dynamics 365 использует систему Omni-Channel, в которой статусы управляются через сущности `msdyn_presence` и нестандартное действие `CCaaS_ModifyAgentPresence`.

### 🔹 Вариант 1: Изменение статуса через Web API

```http
POST https://<your_org>.crm.dynamics.com/api/data/v9.0/CCaaS_ModifyAgentPresence
Content-Type: application/json
Authorization: Bearer <access_token>

{
  "AgentId": "<agent_guid>",
  "PresenceId": "<presence_guid>"
}
```

### 🔹 Вариант 2: Использование Power Automate Flow

* Создать поток с триггером (например, HTTP-запрос)
* Добавить шаг **Perform an unbound action** с названием `CCaaS_ModifyAgentPresence`
* Передать параметры `AgentId` и `PresenceId`

### 🔹 Получение всех возможных статусов

```http
GET https://<your_org>.crm.dynamics.com/api/data/v9.0/msdyn_presences?$select=msdyn_presenceid,msdyn_name
```

### 🔹 Получение статуса конкретного агента

```http
GET https://<your_org>.crm.dynamics.com/api/data/v9.0/systemusers(<agent_guid>)?$expand=msdyn_userpresence($select=msdyn_presenceid,msdyn_name)
```

### 🔹 Создание кастомных статусов

1. Перейти в **Omnichannel Admin Center** → **Agent Experience** → **Presence**
2. Добавить новый статус с нужными параметрами
3. Назначить статус в нужные **Presence profiles**

**Важно:** кастомные статусы появятся в `msdyn_presence`, и будут доступны по API аналогично стандартным.

![image](https://github.com/user-attachments/assets/bc5cecce-29c1-4621-946b-3e63d264d387)# 📘 Интеграция Genesys Cloud Embeddable Framework с Dynamics 365 (v1) через iframe

---

## 🔷 4. CIFramework v1 vs v2: стоит ли переходить?

| Параметр               | CIFramework v1       | CIFramework v2                 |
| ---------------------- | -------------------- | ------------------------------ |
| Поддержка              | Есть                 | Новый стандарт                 |
| API                    | JS через `window`    | ES6-модули + строгая типизация |
| Работа с вкладками     | Через `navigateTo`   | Улучшено                       |
| Расширяемость UI       | iframe / WebResource | PCF, Custom Pages              |
| Интеграция с PowerApps | Нет                  | Да                             |

### 📌 Вывод

Если ты используешь Genesys через iframe и не планируешь глубокую миграцию — **v1 CIFramework вполне подходит**.

---

## 🎁 Пример структуры UI в iframe (settings.html)

```html
<html>
  <head><title>Genesys Settings</title></head>
  <body>
    <h2>Genesys Integration Settings</h2>
    <form>
      <label>Регион: <input type="text" id="region" /></label><br />
      <label>Client ID: <input type="text" id="clientId" /></label><br />
      <label>Status Mapping:</label>
      <table>
        <tr><th>Genesys</th><th>Dynamics</th></tr>
        <tr><td><input value="Available" /></td><td><input value="Ready" /></td></tr>
      </table>
      <button type="submit">Save</button>
    </form>
  </body>
</html>
```

Скрипт можно добавить в конце:

```html
<script>
  document.querySelector("form").addEventListener("submit", (e) => {
    e.preventDefault();
    const region = document.getElementById("region").value;
    const clientId = document.getElementById("clientId").value;
    const settings = { region, clientId };
    localStorage.setItem("genesys_settings", JSON.stringify(settings));
  });
</script>
```

---

## ✅ Резюме

* Ты можешь встроить UI полностью через Genesys, не требуя конфигурации от пользователя D365
* Настройки можно безопасно хранить через CIFramework (`getSession`, `getEnvironment`) или `localStorage`
* Смена статуса в Dynamics требует backend-интеграции (Flow или Plugin) с использованием `CCaaS_ModifyAgentPresence`
* Возможна работа с кастомными статусами
* CIFramework v1 подходит под текущую архитектуру

---

## 🔗 Полезные ссылки

* [Управление статусами в Omnichannel](https://learn.microsoft.com/ru-ru/dynamics365/customer-service/use/oc-manage-presence-status)
* [Расширение статусов присутствия](https://learn.microsoft.com/ru-ru/dynamics365/contact-center/extend/presence-status-sync)
