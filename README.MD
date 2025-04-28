# 1. Таблицы
Во многих случаях используются одни и те же сокращения, они будут опущены.
```javascript
// Сокращенное обращение к таблице
const tableId = 'table-' + w.general.renderTo;
const table = document.getElementById(tableId);
const $table = $(table);

// Цвета для выделения ячеек
const warningColor = "lightcoral"; 
const positiveColor = "lightgreen";

```

## 1.1. Выравнивание по центру заголовка для произвольного количества столбцов
```javascript
// Массив ТОЧНЫХ названий заголовков столбцов для центрирования
const headersToCenter = [
    "Имя сотрудника",
    "Название объекта",
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
// Массив ТОЧНЫХ названий заголовков столбцов с датами для форматирования
const headersToDateFormat = [
    "Дата увольнения сотрудника",
    "Дата начала строительства объекта",
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
const actualTimeHeaderText = "Время исполнения";
const averageTimeHeaderText = "Среднее время исполнения задания";

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
                    actualTimeCell.css('background-color', positiveColor); // Зеленый
                } else if (actualValue > averageValue) {
                    actualTimeCell.css('background-color', warningColor); // Красный
                }
            }
        }
    });
}
```

## 1.4 Окрашивание ячеек в зависимости от значения в другом столбце (сравнение двух строк)
```javascript
$table.find('tbody tr').each(function (rowIndex, tr) {
    const $row = $(tr);
    const $cells = $row.children('td'); // Все ячейки строки
    const $lastCell = $cells.last();   // Последняя ячейка
    if ($lastCell.length > 0) {
        const drugoiStolbecText = $lastCell.text().trim();
        // Проверяем, что в последней ячейке есть текст
        if (drugoiStolbecText) {
            // Проходим по всем ячейкам КРОМЕ последней
            $cells.not($lastCell).each(function () {
                const $currentItem = $(this);
                const currentCellText = $currentItem.text().trim();
                if (currentCellText === drugoiStolbecText) {
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
    [2, 'Содержание'],
    [3, 'ФИО сотрудника']
    // Добавьте сюда другие пары [номер столбца, новое_имя]
]);

for (const [key, value] of headerNameWithIndex) {
    const headerSelector = `thead tr th:nth-child(${key})`;
    const headerCell = table.querySelector(headerSelector); 
    if (headerCell) {
        headerCell.textContent = value;
    }
}
```
## 1.6 Объединение повторяющихся ячеек в одну ячейку
```javascript
const columnHeadersToMerge = [
    "Дата начала кампании",
    // Другие названия заголовков
];

const targetColumnIndices = [];
columnHeadersToMerge.forEach(header => {
    if (headerIndexMap.has(header)) {
        targetColumnIndices.push(headerIndexMap.get(header));
    }
});

if (targetColumnIndices.length > 0) {
    const mergeStateMap = new Map();
    targetColumnIndices.forEach(colIndex => {
        mergeStateMap.set(colIndex, {
            previousValue: null,
            anchorCell: null,
            rowspanCount: 1
        });
    });

    const $rows = $table.find('tbody tr');
    $rows.each(function (rowIndex) {
        const $currentRow = $(this);
        targetColumnIndices.forEach(colIndex => {
            const state = mergeStateMap.get(colIndex);
            if (!state) return;
            const $currentCell = $currentRow.children('td').eq(colIndex);
            if ($currentCell.length === 0) {
                state.previousValue = `__CELL_NOT_FOUND_${rowIndex}_${colIndex}__`;
                state.anchorCell = null;
                state.rowspanCount = 1;
                return;
            }
            const currentValue = $currentCell.text().trim();
            if (currentValue === state.previousValue && currentValue !== "") {
                state.rowspanCount++;
                $currentCell.hide();
                if (state.anchorCell) {
                    state.anchorCell.attr('rowspan', state.rowspanCount);
                    if (state.rowspanCount === 2) { // Стилизуем при первом объединении
                        state.anchorCell.css({'vertical-align': 'middle'});
                    }
                }
            } else {
                state.rowspanCount = 1;
                state.anchorCell = $currentCell;
                state.previousValue = currentValue;
                $currentCell.show().attr('rowspan', 1).css({'vertical-align': ''});
            }
        });
    });
} 
```
## 1.7 Добавление стрелки, указывающей положительную или отрицательную динамику в ячейку, в зависимости от значения
```javascript
const headerIndexMap = new Map();
$(table).find('thead th').each(function(index, th) {
    const headerText = $(th).text().trim();
    headerIndexMap.set(headerText, index);
});

const actualTimeHeaderText = "Время выполнение задачи";
const averageTimeHeaderText = "Среднее время выполнения задачи";

const actualTimeColIndex = headerIndexMap.get(actualTimeHeaderText);
const averageTimeColIndex = headerIndexMap.get(averageTimeHeaderText);

if (actualTimeColIndex !== undefined && averageTimeColIndex !== undefined) {
    $table.find('tbody tr').each(function (rowIndex, tr) {
        const $row = $(tr);
        const actualTimeCell = $row.children('td').eq(actualTimeColIndex);
        const averageTimeCell = $row.children('td').eq(averageTimeColIndex);
        if (actualTimeCell.length > 0 && averageTimeCell.length > 0) {
            actualTimeCell.find('span.time-indicator').remove();
            const actualTimeText = actualTimeCell.text().trim().replace(',', '.');
            const averageTimeText = averageTimeCell.text().trim().replace(',', '.');
            const actualValue = parseFloat(actualTimeText);
            const averageValue = parseFloat(averageTimeText);
            actualTimeCell.css('background-color', '');
            if (!isNaN(actualValue) && !isNaN(averageValue)) {
                let arrowChar = '';
                let arrowColor = '';
                let indicatorClass = '';
                if (actualValue <= averageValue) {
                    arrowChar = '▼'; // Стрелка вниз (лучше)
                    arrowColor = positiveColor;
                    indicatorClass = 'time-down';
                } else if (actualValue > averageValue) {
                    arrowChar = '▲'; // Стрелка вверх (хуже)
                    arrowColor = warningColor;
                    indicatorClass = 'time-up';
                }
                if (arrowChar) {
                    const $indicator = $('<span></span>')
                        .addClass('time-indicator') // Общий класс для всех индикаторов
                        .addClass(indicatorClass)   // Класс для конкретного типа (вверх/вниз)
                        .css('color', arrowColor)   // Задаем цвет стрелки
                        .css({ // Дополнительные стили для отступа
                            'display': 'inline-block', // Чтобы margin работал
                            'margin-left': '5px',      // Отступ слева от числа
                            'font-weight': 'bold'    // Сделать стрелку жирной (опционально)
                        })
                        .text(arrowChar);           // Устанавливаем символ стрелки
                    actualTimeCell.append($indicator);
                }
            }
        }
    });
}
```