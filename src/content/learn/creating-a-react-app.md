---
title: Створення React-застосунку
---

<Intro>

Якщо ви вирішили розробити новий застосунок або вебсайт за допомогою React, ми радимо вибрати один із фреймворків.

</Intro>

## Рекомендовані React-фреймворки {/*bleeding-edge-react-frameworks*/}

Ці фреймворки підтримують усі функції, що знадобляться для розгортання та масштабування застосунку в публічному середовищі (in production). Вони інтегрували останні React-функції й послуговуються перевагами архітектури React.

<Note>

#### React-фреймворки не потребують сервера. {/*react-frameworks-do-not-require-a-server*/}

Усі фреймворки на цій сторінці можуть створювати односторінкові застосунки (SPA). Їх можна розгортати через [CDN](https://developer.mozilla.org/en-US/docs/Glossary/CDN) або сервіс статичного хостингу та вони не потребують сервера. Якщо ви хочете використати функції, для яких потрібен сервер (наприклад, SSR — рендеринг із боку сервера), то можна ввімкнути окремі маршрути без переписування застосунку.

</Note>

### Next.js (App Router) {/*nextjs-app-router*/}

**[App Router від Next.js's](https://nextjs.org/docs) — це React-фреймворк, який повністю використовує переваги архітектури React, щоб уможливлювати React-застосунки повного стеку.**

<TerminalBlock>
npx create-next-app@latest
</TerminalBlock>

Next.js підтримується командою [Vercel](https://vercel.com/). [Розгорнути застосунок Next.js](https://nextjs.org/docs/app/building-your-application/deploying) можна на будь-якому Node.js- або безсерверному хостингу, а також на власному сервері. Також Next.js підтримує [статичне експортування](https://nextjs.org/docs/app/building-your-application/deploying/static-exports), якому не потрібен сервер. На додачу Vercel пропонує необов'язкові платні хмарні послуги.

### React Router (v7) {/*react-router-v7*/}

**[React Router](https://reactrouter.com/start/framework/installation) — це найпопулярніша бібліотека маршрутизації для React, яку можна поєднати з Vite для створення повноцінного React-фреймворку**. Він зосереджується на стандартних вебінтерфейсах і має кілька [готових до розгортання шаблонів](https://github.com/remix-run/react-router-templates) для різних середовищ і платформ JavaScript.

Щоб створити новий проєкт з допомогою фреймворку React Router, виконайте:

<TerminalBlock>
npx create-react-router@latest
</TerminalBlock>

React Router підтримується [компанією Shopify](https://www.shopify.com).

### Expo (для нативних застосунків) {/*expo*/}

**[Expo](https://expo.dev/) — це React-фреймворк, що дає змогу створювати універсальні застосунки для Android, iOS і вебу з по-справжньому нативним UI.** Він пропонує набір інструментів для [React Native](https://reactnative.dev/), завдяки якому цими нативними частинами легше користуватися. Щоб створити новий проєкт Expo, виконайте:

<TerminalBlock>
npx create-expo-app@latest
</TerminalBlock>

Якщо Expo — це для вас щось нове, перегляньте [підручник Expo](https://docs.expo.dev/tutorial/introduction/).

Expo підтримується [компанією Expo](https://expo.dev/about). Розробляти застосунки за допомогою Expo можна безкоштовно, і їх можна подавати до каталогів застосунків Google і Apple без жодних обмежень. На додачу Expo пропонує необов'язкові платні хмарні послуги.


## Інші варіанти {/*other-options*/}

Є й інші перспективні фреймворки, які працюють над нашим баченням повного стеку з React:

- [TanStack Start (бета-версія)](https://tanstack.com/): TanStack Start — це React-фреймворк повного стеку на основі TanStack Router. Він надає повнодокументний SSR, потокове передавання, серверні функції, збирання тощо за допомогою інструментів як Nitro та Vite.
- [RedwoodJS](https://redwoodjs.com/): Redwood — це React-фреймворк повного стеку із великою кількістю попередньо встановлених пакетів і налаштувань, що дає змогу легко створювати вебзастосунки повного стеку.

<DeepDive>

#### З яких функцій складається бачення архітектури повного стеку від команди React? {/*which-features-make-up-the-react-teams-full-stack-architecture-vision*/}

Бандлер App Router від Next.js повністю реалізує офіційну [специфікацію серверних компонентів React](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md). Це дає змогу поєднувати компоненти, що виконуються під час збирання, ті, що працюють суто на сервері, та інтерактивні компоненти в єдине дерево React.

Наприклад, можна написати суто серверний компонент React як `async` функцію, що зчитує дані з бази даних або з файлу. Потім можна передати дані до інтерактивних компонентів:

```js
// Цей компонент виконується *лише* на сервері (або під час збирання).
async function Talks({ confId }) {
  // 1. Ви на сервері, тож можете спілкуватися зі своїм шаром даних. Наявність API — необов'язкова.
  const talks = await db.Talks.findAll({ confId });

  // 2. Додавайте будь-яку кількість логіки рендерингу. Це не збільшить ваш бандл із JavaScript.
  const videos = talks.map(talk => talk.video);

  // 3. Передайте дані до компонентів, що виконаються в браузері.
  return <SearchableVideoList videos={videos} />;
}
```

App Router від Next.js також інтегрує [отримання даних із Suspense](/blog/2022/03/29/react-v18#suspense-in-data-frameworks). Це дає змогу задати стан завантаження (наприклад, каркасний елемент для заповнення (skeleton placeholder)) для різних частин інтерфейсу безпосередньо в дереві React:

```js
<Suspense fallback={<TalksLoading />}>
  <Talks confId={conf.id} />
</Suspense>
```

Серверні компоненти та Suspense — це радше функції React, аніж Next.js. Проте їхнє залучення на рівні фреймворку вимагає нетривіальних зусиль для реалізації та підтримування. Наразі App Router від Next.js — це найповніша імплементація. Команда React працює разом із розробниками бандлерів, щоб було легше реалізувати ці функції в наступному поколінні фреймворків.

</DeepDive>

<Note>


#### Чи рекомендуєте Vite? {/*do-you-recommend-vite*/}

Ми пропонуємо кілька рекомендацій для Vite.

React Router v7 — це фреймворк на основі Vite, який дає вам змогу використовувати швидкий сервер розробки Vite та налаштувати все за допомогою фреймворку, який забезпечує маршрутизацію та отримання даних. Як і в інших фреймворках, які ми рекомендуємо, ви можете створити SPA за допомогою React Router v7.

Ми також рекомендуємо використовувати Vite під час [додавання React до наявного проєкту](/learn/add-react-to-an-existing-project) або [побудови фреймворку](/learn/building-a-react-framework).

Подібно до того, як у Svelte є Sveltekit, у Vue — Nuxt, а у Solid — SolidStart, React рекомендує використовувати для нових проєктів фреймворк, який легко інтегрується з інструментами збирання, як-от Vite.

</Note>

-----

_Якщо ви автор фреймворку та бажаєте додати його на цю сторінку, [будь ласка, напишіть нам](https://github.com/reactjs/react.dev/issues/new?assignees=&labels=type%3A+framework&projects=&template=3-framework.yml&title=%5BFramework%5D%3A+)._