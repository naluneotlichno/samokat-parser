🔥 **Архитектура CLI-парсера для Самоката** 🚀  
Давай сделаем чёткую структуру, чтобы быстро реализовать код и ничего не забыть! 😈

---

## **📌 Функционал CLI-приложения**

1. **Приветствие пользователя** 🎉

   - Выводим сообщение с краткой инструкцией и командами.

2. **Получение списка категорий товаров** 🛍️

   - Делаем запрос к API, получаем и отображаем список категорий.
   - Пользователь выбирает категорию (вводит номер).

3. **Проверка наличия подкатегорий** 🧐

   - Если у категории есть подкатегории → показываем их и просим выбрать.
   - Если подкатегорий нет → переходим к парсингу товаров.

4. **Получение списка товаров из выбранной категории** 📦

   - Делаем запрос к API и забираем товары (название, цена, ссылка).
   - Выводим товары в удобном формате.

5. **Сохранение данных в файл** 📂

   - Пользователь выбирает формат (`JSON` или `CSV`).
   - Парсер сохраняет данные в выбранный формат.

6. **Дополнительные улучшения** (по желанию)
   - **Фильтрация товаров** (например, по минимальной цене).
   - **Кеширование запросов**, чтобы не делать лишние запросы к API.
   - **Выбор города** (если API требует город для точности).

---

## **🛠 Архитектура кода (структура файлов)**

Разделим код на модули, чтобы всё было **чисто и понятно**.

```
/samokat-parser
│── main.go             # Точка входа в приложение
│── internal/
│   ├── cli.go          # Логика командного интерфейса (взаимодействие с пользователем)
│   ├── api.go          # Работа с API Самоката (запросы, обработка данных)
│   ├── parser.go       # Парсинг данных (категории, товары)
│   ├── output.go       # Сохранение данных в JSON/CSV
│── config/
│   ├── headers.json    # Храним поддельные заголовки User-Agent
│── README.md           # Документация
```

💡 **Зачем такая структура?**

- **Отделяем логику API, CLI и обработки данных**.
- **Легко поддерживать и расширять код** (например, добавить новый формат экспорта).

---

## **🔍 Детальное описание каждого модуля**

### **1️⃣ `main.go` – Точка входа**

```go
package main

import (
	"fmt"
	"samokat-parser/internal/cli"
)

func main() {
	fmt.Println("🚀 Добро пожаловать в Самокат-парсер!")
	fmt.Println("🔹 Доступные команды:")
	fmt.Println("  - categories  → Показать список категорий")
	fmt.Println("  - parse       → Выбрать категорию и спарсить товары")
	fmt.Println("  - exit        → Выйти из программы")

	cli.Start()
}
```

💡 **Что делает?**

- Показывает **приветственное сообщение**.
- Запускает обработку команд через `cli.go`.

---

### **2️⃣ `cli.go` – Обработчик команд**

```go
package cli

import (
	"fmt"
	"samokat-parser/internal/api"
	"samokat-parser/internal/parser"
	"samokat-parser/internal/output"
)

func Start() {
	for {
		fmt.Print("\nВведите команду: ")
		var command string
		fmt.Scanln(&command)

		switch command {
		case "categories":
			categories := api.GetCategories()
			for i, cat := range categories {
				fmt.Printf("%d. %s\n", i+1, cat.Name)
			}

		case "parse":
			fmt.Print("Введите номер категории: ")
			var choice int
			fmt.Scanln(&choice)

			category := api.GetCategoryByIndex(choice)
			products := parser.ParseProducts(category)

			fmt.Print("Выберите формат (json/csv): ")
			var format string
			fmt.Scanln(&format)

			output.SaveToFile(products, format)

		case "exit":
			fmt.Println("👋 Пока!")
			return

		default:
			fmt.Println("❌ Неизвестная команда")
		}
	}
}
```

💡 **Что делает?**

- Обрабатывает **команды пользователя** (категории, парсинг, выход).
- Взаимодействует с API и парсером.

---

### **3️⃣ `api.go` – Запросы к API Самоката**

```go
package api

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

const baseURL = "https://api-web.samokat.ru/v2/"

type Category struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

func GetCategories() []Category {
	url := baseURL + "showcases"
	req, _ := http.NewRequest("GET", url, nil)
	req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64)")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Println("❌ Ошибка запроса:", err)
		return nil
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	var categories []Category
	json.Unmarshal(body, &categories)

	return categories
}
```

💡 **Что делает?**

- Получает **список категорий** через API.
- Возвращает **массив с ID и названием**.

---

### **4️⃣ `parser.go` – Парсинг товаров**

```go
package parser

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

type Product struct {
	Name  string `json:"name"`
	Price string `json:"price"`
	URL   string `json:"url"`
}

func ParseProducts(category string) []Product {
	url := fmt.Sprintf("https://api-web.samokat.ru/v2/products?category=%s", category)

	req, _ := http.NewRequest("GET", url, nil)
	req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64)")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Println("❌ Ошибка запроса:", err)
		return nil
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	var products []Product
	json.Unmarshal(body, &products)

	return products
}
```

💡 **Что делает?**

- Получает список товаров из категории.
- Возвращает массив с **названием, ценой и ссылкой**.

---

### **5️⃣ `output.go` – Сохранение в JSON/CSV**

```go
package output

import (
	"encoding/csv"
	"encoding/json"
	"fmt"
	"os"
	"samokat-parser/internal/parser"
)

func SaveToFile(products []parser.Product, format string) {
	if format == "json" {
		file, _ := os.Create("products.json")
		json.NewEncoder(file).Encode(products)
		fmt.Println("✅ Данные сохранены в products.json")
	} else if format == "csv" {
		file, _ := os.Create("products.csv")
		writer := csv.NewWriter(file)
		defer writer.Flush()

		writer.Write([]string{"Название", "Цена", "Ссылка"})
		for _, p := range products {
			writer.Write([]string{p.Name, p.Price, p.URL})
		}
		fmt.Println("✅ Данные сохранены в products.csv")
	} else {
		fmt.Println("❌ Неизвестный формат")
	}
}
```

💡 **Что делает?**

- Сохраняет товары в **JSON** или **CSV**.

---

## **🚀 Готовы к разработке?**

Это полная структура CLI-парсера! 💥 Теперь можно быстро писать код и уже через пару часов **получить рабочее приложение**! 😈🔥

**Ты готов ворваться в код?** 😏💻
