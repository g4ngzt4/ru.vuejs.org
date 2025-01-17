---
title: Безопасность
type: guide
order: 504
---

## Сообщение об уязвимости

При поступлении сообщения об уязвимости, её исправление сразу становится первоочередной задачей для нас. Сотрудник работающий на полную ставку, бросает всё, чтобы заняться ею. Чтобы сообщить об уязвимости, пожалуйста, отправьте сообщение на электронную почту [vuejs.org@gmail.com](mailto:vuejs.org@gmail.com).

Хотя новые уязвимости обнаруживаются редко, рекомендуем также всегда использовать последние версии Vue и его официальных библиотек для обеспечения максимальной безопасности вашего приложения.

## Что делает Vue для вашей защиты

### HTML-содержимое

При использовании шаблонов или render-функций содержимое экранируется автоматически. Это значит, что для шаблона:

```html
<h1>{{ userProvidedString }}</h1>
```

если `userProvidedString` содержит:

```js
'<script>alert("hi")</script>'
```

то он будет экранирован в следующий HTML:

```html
&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

таким образом предотвращая внедрение вредоносного скрипта. Экранирование осуществляется с помощью нативного API браузера, такого как `textContent`, поэтому уязвимость возможна только в случае, если сам браузер уязвим.

### Привязка атрибутов

Аналогичным образом, динамические привязки к атрибутам также автоматически экранируются. Это значит, что для шаблона:

```html
<h1 v-bind:title="userProvidedString">
  hello
</h1>
```

если `userProvidedString` содержит:

```js
'" onclick="alert(\'hi\')'
```

то он будет экранирован в следующий HTML:

```html
&quot; onclick=&quot;alert('hi')
```

тем самым предотвращая преждевременное закрытие атрибута `title` для добавления нового, произвольного HTML. Экранирование выполняется с помощью нативного API браузера, такого как `setAttribute`, поэтому уязвимость возможна только в случае, если сам браузер уязвим.

## Потенциальные опасности

Для любого веб-приложении возможность выполнения пользовательского контента без санитизации в формате HTML, CSS, или JavaScript является потенциально опасным, и этого следует избегать везде где только возможно. Однако бывают моменты, когда некоторый риск может быть приемлемым.

Например, сервисы такие как CodePen и JSFiddle позволяют выполнять пользовательский контент, но в таком контексте где это ожидается и изолируется внутри iframe. В тех случаях, когда важная функция по своей природе требует определённого уровня уязвимости, ваша команда должна взвесить необходимость этой функции с учётом наихудщих сценариев, которые привнесёт её использование.

### Внедрение HTML

Как сказано ранее, Vue автоматически экранирует HTML-содержимое, предотвращая случайное внедрение HTML в приложение. Однако в тех случаях, когда вы уверены в безопасности HTML, можно отображать HTML-содержимое в сыром виде:

- Используя шаблон:
  ```html
  <div v-html="userProvidedHtml"></div>
  ```

- Используя render-функцию:
  ```js
  h('div', {
    domProps: {
      innerHTML: this.userProvidedHtml
    }
  })
  ```

- Используя render-функцию с JSX:
  ```jsx
  <div domPropsInnerHTML={this.userProvidedHtml}></div>
  ```

<p class="tip">Запомните, что предоставленный пользователем HTML никогда не может считаться безопасным на 100%, если не находится в изолированном iframe или в той части приложения, где только пользователь, написавший этот HTML, может получить к нему доступ. Кроме того, разрешать пользователям писать свои собственные шаблоны Vue может привнести аналогичные опасности.</p>

### Внедрение URL

В таком URL-адресе:

```html
<a v-bind:href="userProvidedUrl">
  Нажми на меня
</a>
```

Существует потенциальная проблема безопасности, если URL не был «санитизирован» для предотвращения выполнения JavaScript через `javascript:`. Есть библиотеки, как [sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url), которые могут помочь с этим, но несмотря на это:

<p class="tip">Если вы когда-нибудь занимались санитизацией URL на фронтенде, то у вас уже есть проблема с безопасностью. URL-адреса, предоставляемые пользователем, всегда должны санитизироваться на бэкэнде, перед сохранением в базу данных. Тогда проблема будет решена для _каждого_ клиента, который подключается к вашему API, в том числе нативные мобильные приложения. Также запомните, что использование санитизированных URL-адресов не гарантирует, что они ведут на безопасные ресурсы.</p>

### Внедрение стилей

Посмотрим на этот пример:

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="userProvidedStyles"
>
  Нажми на меня
</a>
```

предположим что `sanitizedUrl` был санитизирован и это действительно настоящий URL, а не JavaScript. Но используя `userProvidedStyles`, злоумышленники всё еще могут предоставить CSS для «click jack», т.е. стилизовать ссылку в виде прозрачного блока поверх кнопки «Входа в систему». В таком случае, если `https://user-controlled-website.com/` куда ведёт ссылка, создан таким, чтобы визуально повторять на страницу авторизации вашего приложения, появляется возможность перехвата например логинов и паролей пользователей.

Аналогично можете представить себе, как разрешение пользователям определять содержимое элемента `<style>` создаст ещё большую уязвимость, предоставив полный контроль над стилями страницы. Поэтому не стоит отрисовывать теги стилей внутри шаблонов, например так:

```html
<style>{{ userProvidedStyles }}</style>
```

Чтобы полностью обезопасить пользователей от техники click jacking, рекомендуем разрешать полный контроль над CSS только внутри изолированного iframe. В качестве альтернативы, если всё-таки необходимо предоставить пользователю возможность настройки стилей рекомендуем использовать [объектный синтаксис](class-and-style.html#Использование-объектов-1) и позволять указывать только значения для конкретных свойств, которые безопасно изменять, например так:

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="{
    color: userProvidedColor,
    background: userProvidedBackground
  }"
>
  Нажми на меня
</a>
```

### Внедрение JavaScript

Мы настоятельно не рекомендуем отрисовку элементов `<script>` с помощью Vue, поскольку шаблоны и render-функции никогда не должны иметь в себе побочных эффектов (side effects). Но это не единственный способ для включения строк, которые будут расцениваться как JavaScript во время выполнения.

Каждый HTML-элемент может иметь атрибуты, значения которых принимают строки JavaScript, например `onclick`, `onfocus`, и `onmouseenter`. Привязка JavaScript предоставленного пользователем к любому из этих атрибутов является потенциальной угрозой безопасности, поэтому этого следует избегать.

<p class="tip">Запомните, что предоставленный пользователем JavaScript никогда не может считаться безопасным на 100%, если он не находится в изолированном iframe или в части приложения, где только написавший его пользователь может когда-либо получить доступ к нему.</p>

Иногда мы получаем сообщения об уязвимостях, что в шаблонах Vue возможно выполнение межсайтового скриптинга (XSS). В целом, мы не считаем такие случаи реальными уязвимостями, так как нет практического способа защиты разработчиков от двух сценариев, допускающих использование XSS:

1. Разработчик явно указывает Vue отрисовать предоставленный пользователем, не-санитизированный контент в шаблонах Vue. По своей природе это небезопасно и у Vue нет возможности отслеживать это.

2. Разработчик монтирует Vue на всю HTML-страницу, которая, как оказалось, содержит как контент отрисованный на сервере, так и предоставленный пользователем. Фундаментально эта проблема аналогична \#1, но иногда разработчики могут сделать так, не осознавая этого. Подобное может привести к возможным уязвимостям, когда атакующий предоставляет HTML, который безопасен как обычный HTML, но небезопасен в качестве шаблона Vue. Лучше всего никогда не монтировать Vue на узлах, которые могут содержать контент предоставленный как сервером так и пользователем.

## Лучшие практики

Главное правило заключается в том, что если разрешаете выполнение не-санитизированного пользовательского контента (как например HTML, JavaScript, или даже CSS), то открываетесь для атак. Этот совет остаётся действенным, независимо от того используется ли Vue, другой фреймворк или никакого вообще.

Кроме вышеизложенных рекомендаций из раздела [потенциальных опасностей](#Потенциальные-опасности), рекомендуем также ознакомиться со следующими ресурсами:

- [HTML5 Security Cheat Sheet](https://html5sec.org/)
- [OWASP's Cross Site Scripting (XSS) Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)

После изучения используйте полученные знания для проверки исходного кода зависимостей на наличие потенциально опасных мест, если они содержат сторонние компоненты или каким-либо иным образом влияют на то, что будет отрисовываться в DOM.

## Координация с бэкэндом

Уязвимости безопасности HTTP, такие как подделка межсайтовых запросов (CSRF/XSRF) или внедрение межсайтовых скриптов (XSSI), в основном нацелены на бэкэнд, поэтому Vue тут мало чем может помочь. Тем не менее, лучше скоординировать действия с командой разработчиков бэкэнда, чтобы лучше узнать как следует взаимодействовать с API, например отправляя CSRF-токены при отправке форм.

## Отрисовка на стороне сервера (SSR)

При использовании SSR могут возникнуть дополнительные проблемы с безопасностью, поэтому во избежание уязвимостей следуйте рекомендациям изложенным в [документации по SSR](https://ssr.vuejs.org/ru/).
