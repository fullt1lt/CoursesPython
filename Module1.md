# Модуль1. Основы Python и Git

## Темы: переменные, списки, циклы, условия, файлы, функции, сортировка, Git.

### Задание:

Вам нужно создайть консольное приложение для управления контактами.Каждый контакт содержит имя, телефон, email.
Контакты хранятся в файле `contacts.txt`.

Функции программы:

- Добавление контакта
- Поиск контактов по имени или телефону
- Удаление контакта
- Обновление данных контакта
- Просмотр всех контактов

## Как взаимодействовать с пользователем?

При запуске приложения дать пользователю выбор:

1. Добавить контакт
2. Найти контакт
3. Удалить контакт
4. Обновить контакт
5. Просмотреть контакты
6. Выйти

Если пользователь вводит что-либо другое, сообщить "❌ Неверный выбор. Попробуйте снова.".

При выборе:

1. Добавить контакт.
   По очереди предложить ввести:
   - имя,
   - телефон(проверять введено ли число и колличество символов(должно быть 12 цифр)),
   - email(проверять есть ли знаки (@.)),

Проверять все данные на пустую строку. Если все данные введены корретно, сохранить их в файл `contacts.txt`.
При успехе вывести сообщение "✅ Контакт успешно добавлен!".
Если данные не валидны заново попросить их ввести.

2. Найти контакт
   Предложить пользователю ввести имя или номер телефона. Осуществить поиск контакт с таким именем или телефоном из файла `contacts.txt`.
   Если такой контакт существует в контактной книге то вывести его, если их несколько то вывести все. Если контакт не найден, то вывести сообщение ("❌ Контакт не найден.")
3. Удалить контакт
   Предложить пользователю ввести имя или номер телефона. Осуществить поиск контакт с таким именем или телефоном из файла `contacts.txt`.
   Если такой контакт существует в контактной книге то удалить его("✅ Контакт удалён!"). Если контакт не найден, то вывести сообщение ("❌ Контакт не найден.")
4. Обновить контакт
   Предложить пользователю ввести имя или номер телефона. Осуществить поиск контакт с таким именем или телефоном из файла `contacts.txt`.
   Если такой контакт существует в контактной книге то обновить его.
   Процес обновления контакта должен выглядеть так:
   По очереди предложить ввести:

   - имя,
   - телефон(проверять введено ли число и колличество символов(должно быть 12 цифр)),
   - email(проверять есть ли знаки (@.)),

  По завершению обновления вывести сообщение ("✅ Контакт обновлён!")
  Если контакт не найден, то вывести сообщение ("❌ Контакт не найден.")

5. Просмотреть контакты
   Сортировать по имени("по возрастанию(А-Я)). Вывести отсортированный список.
6. Выйти
   Вывести "👋 Программа завершена. До свидания!"

## Рекомендации по написанию программы

Я рекомендую декомпозировать задачу. Сначала написать код который не взаимодействует с пользователем и не использует файлы вообще.

- Продумать структуру в которой потенциально могут храниться наши данные
- Написать код, который позволит добавить/изменить/прочитать/удалить задачу просто что бы убедиться, что вы можете это сделать
- Перенести добавление/чтение/удаление/изменение в отдельные функции
- Добавить проверки, что операции возможны
- Добавить сортировки для чтения
- Добавить чтение/запись в файл для каждого действия
- Добавить взаимодействие с пользователем

## Сдача модуля

Задание предполагает, что у вас на него 2 недели. И результат вам нужно в виде `Pull Request`.
