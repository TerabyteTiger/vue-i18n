# Синтаксис сообщений локализации

## Структура

Синтаксис сообщений локализации:

```typescript
// As Flowtype definition, Locale Messages syntax like BNF annotation
type LocaleMessages = { [key: Locale]: LocaleMessageObject }
type LocaleMessageObject = { [key: Path]: LocaleMessage }
type LocaleMessageArray = LocaleMessage[]
type LocaleMessage = string | LocaleMessageObject | LocaleMessageArray
type Locale = string
type Path = string
```

Используя синтаксис выше, можно создать следующую структуру сообщений локализации:

```json
{
  // локализация 'en'
  "en": {
    "key1": "это сообщение 1", // обычное использование
    "nested": {
      // вложенное
      "message1": "это вложенное сообщение 1"
    },
    "errors": [
      // массив
      "это сообщение кода ошибки 0",
      {
        // объект в массиве
        "internal1": "это внутреннее сообщение кода ошибки 1"
      },
      [
        // массив в массиве
        "это вложенный массив ошибки 1"
      ]
    ]
  },
  // локализация 'ru'
  "ru": {
    // ...
  }
}
```

Для такой структуры сообщений локализации, можно переводить сообщения используя ключи:

```html
<div id="app">
  <!-- обычное использование -->
  <p>{{ $t('key1') }}</p>
  <!-- вложенное -->
  <p>{{ $t('nested.message1') }}</p>
  <!-- массив -->
  <p>{{ $t('errors[0]') }}</p>
  <!-- объект в массиве -->
  <p>{{ $t('errors[1].internal1') }}</p>
  <!-- массив в массиве -->
  <p>{{ $t('errors[2][0]') }}</p>
</div>
```

Результат:

```html
<div id="app">
  <!-- обычное использование -->
  <p>это сообщение 1</p>
  <!-- вложенное -->
  <p>это вложенное сообщение 1</p>
  <!-- массив -->
  <p>это сообщение кода ошибки 0</p>
  <!-- объект в массиве -->
  <p>это внутреннее сообщение кода ошибки 1</p>
  <!-- массив в массиве -->
  <p>это вложенный массив ошибки 1</p>
</div>
```

## Связанные сообщения локализации

Когда есть ключ с сообщением перевода, которое в точности повторяется в сообщении по другому ключу, то вместо дублирования можно поставить ссылку на него. Для этого к его содержимому нужно добавить префикс `@:` после которого указать полное имя ключа к сообщению перевода, включая пространство имён, к которому делаем ссылку.

Сообщения локализации:

```js
const messages = {
  en: {
    message: {
      the_world: 'the world',
      dio: 'DIO:',
      linked: '@:message.dio @:message.the_world !!!!'
    }
  }
}
```

Шаблон:

```html
<p>{{ $t('message.linked') }}</p>
```

Результат:

```html
<p>DIO: the world !!!!</p>
```

### Форматирование связанных сообщений локализации

Если важен регистр символов в переводе, то можно управлять регистром связанного сообщения локализации. Связанные сообщения можно отформатировать используя модификатор `@.modifier:key`

Доступны следующие модификаторы:

- `upper`: Форматирование в верхний регистр всех символов в связанном сообщении.
- `lower`: Форматирование в нижний регистр всех символов в связанном сообщении.
- `capitalize`: Форматирование заглавной первой буквы в связанном сообщении.

Сообщения локализации:

```js
const messages = {
  en: {
    message: {
      homeAddress: 'Home address',
      missingHomeAddress: 'Please provide @.lower:message.homeAddress'
    }
  },
  ru: {
    message: {
      homeAddress: 'Домашний адрес',
      missingHomeAddress: 'Пожалуйста укажите @.lower:message.homeAddress'
    }
  }
}
```

```html
<label>{{ $t('message.homeAddress') }}</label>

<p class="error">{{ $t('message.missingHomeAddress') }}</p>
```

Результат:

```html
<label>Домашний адрес</label>

<p class="error">Пожалуйста укажите домашний адрес</p>
```

При необходимости можно добавлять новые модификаторы или перезаписывать существующие через опцию `modifiers` в конструкторе `VueI18n`.

```js
const i18n = new VueI18n({
  locale: 'ru',
  modifiers: {
    // Добавление нового модификатора
    snakeCase: str => str.split(' ').join('-')
  },
  messages: {
    // ...
  },
})
```

### Группировка с помощью скобок

Ключ связанного сообщения также можно указывать в виде `@:(message.foo.bar.baz)`, где ссылка на другой ключ перевода обрамляется в скобки `()`.

Подобное может потребоваться, если за ссылкой на другое сообщение `@:message.something` требуется поставить точку `.`, которая в противном случае считалась бы частью ссылки.

Сообщения локализации:

```js
const messages = {
  en: {
    message: {
      dio: 'DIO',
      linked: "There's a reason, you lost, @:(message.dio)."
    }
  },
  ru: {
    message: {
      dio: 'DIO',
      linked: "Есть причина по которой ты проиграл, @:(message.dio)."
    }
  }
}
```

Шаблон:

```html
<p>{{ $t('message.linked') }}</p>
```

Результат:

```html
<p>There's a reason, you lost, DIO.</p>
```