# 📘 Интеграция Genesys Cloud Embeddable Framework с Dynamics 365 (v1) через iframe

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

### ✅ Варианты хранения

| Метод                       | Персистентность | Использует только CIFramework | Подходит? |
| --------------------------- | --------------- | ----------------------------- | --------- |
| `getEnvironment()`          | ❌               | ✅                             | ✅         |
| `setSession` / `getSession` | ❌               | ✅                             | ✅         |
| `localStorage` в iframe UI  | ✅               | ✅                             | ✅         |
| `navigateTo + sharedVar`    | ✅               | ✅                             | ✅\*       |


### 🔹 Пример 1: сохранение в сессии

```ts
Microsoft.CIFramework.setSession("genesys_settings", JSON.stringify({
  region: "prod-euw1",
  clientId: "abc123"
}));

Microsoft.CIFramework.getSession("genesys_settings").then((value) => {
  const settings = JSON.parse(value);
});
```

### 🔹 Пример 2: использование `localStorage` в iframe

```ts
localStorage.setItem("genesys_settings", JSON.stringify({ region: "prod", clientId: "123" }));
const settings = JSON.parse(localStorage.getItem("genesys_settings") || "{}");
```

---

## 🔷 3. Синхронизация статусов агентов

### ✅ Получение и установка статуса в **Genesys**

Работает через Genesys Cloud SDK или Embeddable Framework Events.

### ❌ Прямая установка статуса в Dynamics **невозможна** через `CIFramework`

### 🧩 Как это реализовать:

#### Чтобы статус отразился в Dynamics:

   * **нельзя напрямую менять статус Omni-Channel агента** через JS или API
   * нужно использовать **Power Automate Flow**, который по webhook может вызывать внутреннюю команду
   * или Plugin в Dynamics, который выполнит нужное действие от имени пользователя

### ⚠️ Примерной архитектуры:

```text
Genesys SDK <---> iframe UI <---> CIFramework <--> HTTP-запрос --> Flow/Plugin --> Смена статуса в Dynamics
```

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
    <h2>Настройки Genesys Интеграции</h2>
    <form>
      <label>Регион: <input type="text" id="region" /></label><br />
      <label>Client ID: <input type="text" id="clientId" /></label><br />
      <label>Сопоставление статусов:</label>
      <table>
        <tr><th>Genesys</th><th>Dynamics</th></tr>
        <tr><td><input value="Available" /></td><td><input value="Ready" /></td></tr>
      </table>
      <button type="submit">Сохранить</button>
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
* Смена статуса в Dynamics требует backend-интеграции (Flow или Plugin)
* CIFramework v1 подходит под текущую архитектуру

---

Если нужно, могу собрать тебе базовый `settings.html` + React шаблон + схемы вызова Flow для смены статуса.
