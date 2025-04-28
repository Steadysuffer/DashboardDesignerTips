# 1. Таблицы

## 1.1. Выравнивание по центру заголовка для произвольного количества столбцов
```javascript
// Переменные для удобства обращения и сокращения кода
const tableId = 'table-' + w.general.renderTo;
const $table = $(document.getElementById(tableId));

// Массив ТОЧНЫХ названий заголовков столбцов для центрирования
const headersToCenter = [
    "Время на согласование, дней",
    "Среднее время согласования данного документа, дней",
    "Дата создания документа",          
    "Дата назначения на согласованта",
    // Добавьте еще столбцы при необходимости
];

// Создаем карту (Map) для быстрого поиска индекса по названию заголовка в формате:
// headerIndexMap = {
//      headerText1: index1,
//      headerText2: index2,
// }
// ВАЖНО: Заполняем карту ДО возможных удалений столбцов
const headerIndexMap = new Map();
$table.find('thead th').each(function (index, th) {
    const headerText = $(th).text().trim();
    if (headerText) {
        headerIndexMap.set(headerText, index);
}
});
// Пробегаемся по массиву столбцов и для каждого столбца применяем css стили для выравнивания
headersToCenter.forEach(headerText => {
    if (headerIndexMap.has(headerText)) { // Проверяем наличие текста в заголовке
        const index = headerIndexMap.get(headerText); // Получаем индекс столбца по заголовку
        const nthChildIndex = index + 1; // Индексы начинаются с нуля
        const thSelector = `#${tableId} thead th:nth-child(${nthChildIndex})`;
        const tdSelector = `#${tableId} tbody td:nth-child(${nthChildIndex})`;
        $(thSelector).css('text-align', 'center');  // Для заголовка
        $(tdSelector).css('text-align', 'center');  // Для значений в столбце
    }
});
```
## 1.2. Форматирование даты для произвольного количества столбцов
```javascript
// Переменные для удобства обращения и сокращения кода
const tableId = 'table-' + w.general.renderTo;
const $table = $(document.getElementById(tableId));

// Массив ТОЧНЫХ названий заголовков столбцов с датами для форматирования
const headersToDateFormat = [
    "Дата создания документа",
    "Дата назначения на согласованта",
    // Добавьте еще столбцы при необходимости
    
];

// Создаем карту (Map) для быстрого поиска индекса по названию заголовка в формате:
// headerIndexMap = {
//      headerText1: index1,
//      headerText2: index2,
// }
// ВАЖНО: Заполняем карту ДО возможных удалений столбцов
const headerIndexMap = new Map();
$table.find('thead th').each(function (index, th) {
    const headerText = $(th).text().trim();
    if (headerText) {
        headerIndexMap.set(headerText, index);
    }
});

headersToDateFormat.forEach(headerText => {
    if (headerIndexMap.has(headerText)) {
        const index = headerIndexMap.get(headerText);
        const nthChildIndex = index + 1;
        const dateCellsSelector = `#${tableId} tbody td:nth-child(${nthChildIndex})`;
        $(dateCellsSelector).each(function () {
            const originalDateString = $(this).text().trim();
            if (originalDateString && originalDateString.includes('T') && originalDateString.includes('Z')) {
                const dateObject = new Date(originalDateString);
                if (!isNaN(dateObject.getTime())) {
                    const day = String(dateObject.getDate()).padStart(2, '0');
                    const month = String(dateObject.getMonth() + 1).padStart(2, '0');
                    const year = dateObject.getFullYear();
                    const hours = String(dateObject.getHours()).padStart(2, '0');
                    const minutes = String(dateObject.getMinutes()).padStart(2, '0');
                    const seconds = String(dateObject.getSeconds()).padStart(2, '0');
                    const formattedDate = `${day}.${month}.${year} ${hours}:${minutes}:${seconds}`;
                    $(this).text(formattedDate);
                }
            }
        });
    }
});
```

## 1.3 Окрашивание ячеек в зависимости от значения в другом столбце (сравнение двух чисел)
```javascript
// Названия заголовков, для которых делаем окрашивание
const actualTimeHeaderText = "Время на согласование, дней";
const averageTimeHeaderText = "Среднее время согласования данного документа, дней";

const tableId = 'table-' + w.general.renderTo;
const table = document.getElementById(tableId);
const $table = $(table);

// Создаем карту (Map) для быстрого поиска индекса по названию заголовка в формате:
// headerIndexMap = {
//      headerText1: index1,
//      headerText2: index2,
// }
// ВАЖНО: Заполняем карту ДО возможных удалений столбцов
const headerIndexMap = new Map();
$table.find('thead th').each(function (index, th) {
    const headerText = $(th).text().trim();
    if (headerText) {
        headerIndexMap.set(headerText, index);
    }
});

const actualTimeColIndex = headerIndexMap.get(actualTimeHeaderText);
const averageTimeColIndex = headerIndexMap.get(averageTimeHeaderText);

// Проверяем, найдены ли оба столбца
if (actualTimeColIndex !== undefined && averageTimeColIndex !== undefined) {
    $table.find('tbody tr').each(function (rowIndex, tr) {
        const $row = $(tr);
        const actualTimeCell = $row.children('td').eq(actualTimeColIndex);
        const averageTimeCell = $row.children('td').eq(averageTimeColIndex);
        // Проверяем, что обе ячейки существуют в строке
        if (actualTimeCell.length > 0 && averageTimeCell.length > 0) {
            const actualTimeText = actualTimeCell.text().trim().replace(',', '.');
            const averageTimeText = averageTimeCell.text().trim().replace(',', '.');
            const actualValue = parseFloat(actualTimeText);
            const averageValue = parseFloat(averageTimeText);
            // Сбрасываем фон перед установкой нового
            actualTimeCell.css('background-color', '');
            if (!isNaN(actualValue) && !isNaN(averageValue)) {
                if (actualValue < averageValue) {
                    actualTimeCell.css('background-color', 'lightgreen'); // Зеленый
                } else if (actualValue > averageValue) {
                    actualTimeCell.css('background-color', 'lightcoral'); // Красный
                }
            }
        }
    });
}
```

## 1.4 Окрашивание ячеек в зависимости от значения в другом столбце (сравнение двух строк)
```javascript
const warningColor = "lightcoral";

$table.find('tbody tr').each(function (rowIndex, tr) {
    const $row = $(tr);
    const $cells = $row.children('td'); // Все ячейки строки
    const $lastCell = $cells.last();   // Последняя ячейка
    if ($lastCell.length > 0) {
        const fioText = $lastCell.text().trim();
        // Проверяем, что в последней ячейке есть текст
        if (fioText) {
            // Проходим по всем ячейкам КРОМЕ последней
            $cells.not($lastCell).each(function () {
                const $currentItem = $(this);
                const currentCellText = $currentItem.text().trim();
                // Сравниваем текст текущей ячейки с текстом последней
                if (currentCellText === fioText) {
                    $currentItem.css('background-color', warningColor);
                }
            });
        }
    }
});
```
## 1.5 Переименование заголовков столбцов
```javascript
const headerNameWithIndex = new Map([
    [1, 'Номер документа'],
    [2, 'Краткое содержание'],
    [3, 'ФИО согласованта']
    // Добавьте сюда другие пары [номер столбца, новое_имя]
]);
// Создаем переменную с таблицей для удобства работы
const table = document.getElementById('table-' + w.general.renderTo);
const $table = $(table);

for (const [key, value] of headerNameWithIndex) {
    const headerSelector = `thead tr th:nth-child(${key})`;
    const headerCell = table.querySelector(headerSelector); 
    if (headerCell) {
        headerCell.textContent = value;
    }
}
```