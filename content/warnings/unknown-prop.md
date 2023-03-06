---
title: "Попередження: невідомий проп"
layout: single
permalink: warnings/unknown-prop.html
---
Попередження про наявність невідомого пропа з'являються, коли ви намагаєтесь відобразити DOM-елемент з пропом, що не може бути визначений React, як валідний атрибут або властивість. Ви повинні переконатись у тому, що DOM-елемент помилково не отримав пропси.

Існує декілька можливих причин, через які це попередження може з'явитись:

1. Можливо ви використовуєте `{...this.props}` або `cloneElement(element, this.props)`? Ваш компонент передає власні пропси безпосередньо до дочірнього компонента (див. [Компоненти і пропси](/docs/transferring-props.html)). У цьому випадку потрібно переконатись у тому, що пропси, призначенні лише для батьківського компонента, не потрапляють до дочірнього.

2. Ви використовуєте нестандартний DOM-атрибут на нативному DOM-вузлі, наприклад, для представлення власних даних. Якщо ви намагаєтесь додати власні дані до стандартного елементу DOM, розгляньте можливість використання спеціального data-атрибуту, як описано [на MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes).

3. React не може розпізнати зазначений вами атрибут. Ймовірно, це буде виправлено у наступних версіях React. Однак наразі React ігнорує усі невідомі атрибути й тому їх визначення у вашому додатку не призведе до їх відображення.

4. Ім'я вашого React-компонента починається з малої літери. У цьому випадку React інтерпретує його у якості DOM-тегу тому, що [для трансформації JSX використовується верхній та нижній регістри](/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized), щоб розрізняти React-компоненти та нативні елементи DOM.

---

Для того, щоби розв'язати цю проблему, складові компоненти мають отримувати всі пропси, призначені лише для них, та не передавати їх дочірнім компонентам. Наприклад:

**Погано:** Неочікуваний проп `layout` був переданий до тегу `div`.

```js
function MyDiv(props) {
  if (props.layout === 'horizontal') {
    // Погано! Тому що ви впевнені, що "layout" не є пропом, який <div> зрозуміє
    return <div {...props} style={getHorizontalStyle()} />
  } else {
    // Погано! Тому що ви впевнені, що "layout" не є пропом, який <div> зрозуміє
    return <div {...props} style={getVerticalStyle()} />
  }
}
```

<<<<<<< HEAD
**Добре:** Spread-оператор може бути використаний для деструктурізації поля об'єкту та визначення його, як окремої змінної.
=======
**Good:** The spread syntax can be used to pull variables off props, and put the remaining props into a variable.
>>>>>>> ba290ad4e432f47a2a2f88d067dacaaa161b5200

```js
function MyDiv(props) {
  const { layout, ...rest } = props
  if (layout === 'horizontal') {
    return <div {...rest} style={getHorizontalStyle()} />
  } else {
    return <div {...rest} style={getVerticalStyle()} />
  }
}
```

**Добре:** Також ви можете скопіювати усі пропси до нового об'єкту та видалити зайві ключі. Майте на увазі, що видаляти ключі потрібно лише у новому об'єкті, оскільки об'єкт `this.props` відповідає концепції іммутабельності, тобто має бути незмінним.

```js
function MyDiv(props) {

  const divProps = Object.assign({}, props);
  delete divProps.layout;

  if (props.layout === 'horizontal') {
    return <div {...divProps} style={getHorizontalStyle()} />
  } else {
    return <div {...divProps} style={getVerticalStyle()} />
  }
}
```
