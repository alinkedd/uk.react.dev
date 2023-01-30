---
id: hooks-reference
title: API-довідник хуків
permalink: docs/hooks-reference.html
prev: hooks-custom.html
next: hooks-faq.html
---

*Хуки* — це новинка в React 16.8. Вони дозволяють вам використовувати стан та інші можливості React без написання класу.

Ця сторінка описує API для вбудованих у React хуків.

Якщо ви новачок у хуках, ви можете спочатку переглянути [огляд](/docs/hooks-overview.html). Також ви можете знайти корисну інформацію у розділі [FAQ](/docs/hooks-faq.html).

- [Базові Хуки](#basic-hooks)
  - [`useState`](#usestate)
  - [`useEffect`](#useeffect)
  - [`useContext`](#usecontext)
- [Додаткові Хуки](#additional-hooks)
  - [`useReducer`](#usereducer)
  - [`useCallback`](#usecallback)
  - [`useMemo`](#usememo)
  - [`useRef`](#useref)
  - [`useImperativeHandle`](#useimperativehandle)
  - [`useLayoutEffect`](#uselayouteffect)
  - [`useDebugValue`](#usedebugvalue)
  - [`useDeferredValue`](#usedeferredvalue)
  - [`useTransition`](#usetransition)
  - [`useId`](#useid)
- [Library Hooks](#library-hooks)
  - [`useSyncExternalStore`](#usesyncexternalstore)
  - [`useInsertionEffect`](#useinsertioneffect)

## Базові хуки {#basic-hooks}

### `useState` {#usestate}

> Try the new React documentation for [`useState`](https://beta.reactjs.org/reference/react/useState).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)

```js
const [state, setState] = useState(initialState);
```

Повертає значення стану та функцію, що оновлює його.

Під час початкового рендеру повернутий стан (`state`) співпадає зі значенням, переданим у першому аргументі (`initialState`).

Функція `setState` використовується для оновлення стану. Вона приймає значення нового стану і ставить у чергу повторний рендер компонента.

```js
setState(newState);
```

Упродовж наступних повторних рендерів, перше значення, повернуте `useState`, завжди буде в актуальному стані при здійсненні оновлень.

>Примітка
>
>React гарантує, що функція `setState` зберігає ідентичність і не змінюється під час повторних рендерів. Саме тому, ви можете безпечно пропускати її включення до списків залежностей хуків `useEffect` чи `useCallback`.

#### Функціональні оновлення {#functional-updates}

Якщо наступний стан обчислюється з використанням попереднього, ви можете передати функцію до `setState`. Функція отримає попереднє значення і поверне оновлене. Ось приклад компонента лічильника, що використовує обидві форми `setState`:

```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Скинути</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

Кнопки "+" та "-" використовують функціональну форму, тому що оновлене значення базується на попередньому. В той же час кнопка "Скинути" використовує нормальну форму, тому що вона завжди скидає значення назад до початкового значення.

Якщо ваша функція для оновлення повертає таке саме значення, що і поточний стан, то наступний ререндер буде пропущено.

> Примітка
>
> На відміну від методу `setState` у класових компонентах, `useState` не об'єднує оновлювані об'єкти автоматично. Ви можете відтворити таку поведінку, комбінуючи функціональну форму оновлення і синтаксис розширення об'єктів:
>
> ```js
> const [state, setState] = useState({});
> setState(prevState => {
>   // Object.assign також спрацює
>   return {...prevState, ...updatedValues};
> });
> ```
>
> Іншим варіантом може бути хук `useReducer`, котрий більш підходить для керування об'єктами стану, що містять багато інших значень.

#### Лінива ініціалізація стану {#lazy-initial-state}

Аргумент `initialState` — це стан, що використовується протягом початкового рендеру. При наступних рендерах воно не враховується. Якщо початковий стан є результатом вартісних обчислень, ви можете замість нього надати функцію, що буде виконана лише під час початкового рендеру:

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

#### Припинення оновлення стану {#bailing-out-of-a-state-update}

Якщо ви оновите стан хука значенням, що дорівнює поточному, React вийде з хука без рендерингу дочірніх елементів чи запуску ефектів. (React використовує [алгоритм порівняння `Object.is`](https://developer.mozilla.org/uk/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).)

Зверніть увагу, що React-у може знадобитись відрендерити конкретний компонент перед припиненням оновлення. Це не повинно викликати занепокоєння, тому що React необов'язково опуститься "глибше" в дерево. Якщо ви здійснюєте вартісні обчислення під час рендерингу, ви можете оптимізувати їх, використавши `useMemo`.

#### Batching of state updates {#batching-of-state-updates}

React may group several state updates into a single re-render to improve performance. Normally, this improves performance and shouldn't affect your application's behavior.

Before React 18, only updates inside React event handlers were batched. Starting with React 18, [batching is enabled for all updates by default](/blog/2022/03/08/react-18-upgrade-guide.html#automatic-batching). Note that React makes sure that updates from several *different* user-initiated events -- for example, clicking a button twice -- are always processed separately and do not get batched. This prevents logical mistakes.

In the rare case that you need to force the DOM update to be applied synchronously, you may wrap it in [`flushSync`](/docs/react-dom.html#flushsync). However, this can hurt performance so do this only where needed.

### `useEffect` {#useeffect}

> Try the new React documentation for [`useEffect`](https://beta.reactjs.org/reference/react/useEffect).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)

```js
useEffect(didUpdate);
```

Приймає функцію, що містить імперативний, можливо з ефектами, код.

Зміни, підписки, таймери, логування та інші побічні ефекти не дозволяються всередині основного тіла функціонального компонента (яке ми називаємо _етап рендеру_). Це призводить до заплутаних помилок та невідповідностям у користувацькому інтерфейсі.

Натомість застосовуйте `useEffect`. Функція, передана в `useEffect`, буде запущена після того, як вивід рендеру з'явиться на екрані. Думайте про ефекти як про засіб втечі з чисто функціонального світу React до світу імперативів.

За замовчуванням ефекти запускаються після кожного завершеного рендеру, але ви можете запускати їх, наприклад, коли [змінились тільки певні значення](#conditionally-firing-an-effect).

#### Очищення ефектів {#cleaning-up-an-effect}

Ефекти часто створюють ресурси, пам'ять після використання яких, має бути звільнена перед тим, як компонент зникне з екрану, наприклад, підписка або ідентифікатор таймера. Щоб це зробити, функція, передана у `useEffect`, може повернути функцію очищення. Наприклад, щоб створити підписку:

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Очистити підписку
    subscription.unsubscribe();
  };
});
```

Функція очищення буде запущена перед видаленням компонента з інтерфейсу користувача, щоб запобігти витокам пам'яті. Крім цього, якщо компонент рендериться багато разів (що доволі типово), **попередній ефект буде очищено до виконання наступного**. У нашому прикладі це означає, що нова підписка створюється на кожному оновленні. Зверніться до наступного розділу, щоб дізнатися, як цього можна уникнути.

#### Порядок спрацювання ефектів {#timing-of-effects}

На відміну від `componentDidMount` і `componentDidUpdate`, функція передана в `useEffect` запускається **після** розмітки та рендеру, протягом відкладеної події. Це робить хук підходящим для багатьох поширених побічних ефектів, таких як налаштування підписок та обробників подій, оскільки більшість типів роботи не повинні блокувати оновлення екрану браузером.

Проте не всі ефекти можуть бути відкладені. Наприклад, зміна DOM, що видима користувачу, має запуститись синхронно перед наступним рендером, щоб користувач не помічав візуальної невідповідності. (Ця відмінність концептуально подібна до відмінності між пасивними та активними слухачами подій.) Для таких різновидів ефектів React надає додатковий хук, що зветься [`useLayoutEffect`](#uselayouteffect). Він має таку ж сигнатуру, як і `useEffect`, але відрізняється умовою запуску.

<<<<<<< HEAD
Незважаючи на те, що `useEffect` відкладається допоки браузер не виконає відображення, він гарантовано спрацює перед кожним новим рендером. React завжди застосовує ефекти попереднього рендеру перед початком нового оновлення.
=======
Additionally, starting in React 18, the function passed to `useEffect` will fire synchronously **before** layout and paint when it's the result of a discrete user input such as a click, or when it's the result of an update wrapped in [`flushSync`](/docs/react-dom.html#flushsync). This behavior allows the result of the effect to be observed by the event system, or by the caller of [`flushSync`](/docs/react-dom.html#flushsync).

> Note
> 
> This only affects the timing of when the function passed to `useEffect` is called - updates scheduled inside these effects are still deferred. This is different than [`useLayoutEffect`](#uselayouteffect), which fires the function and processes the updates inside of it immediately.

Even in cases where `useEffect` is deferred until after the browser has painted, it's guaranteed to fire before any new renders. React will always flush a previous render's effects before starting a new update.
>>>>>>> 5647a9485db3426d62b5a8203f4499c01bcd789b

#### Умовне спрацювання ефекту {#conditionally-firing-an-effect}

За замовчуванням ефекти спрацьовуються після кожного завершеного рендеру. Таким чином, ефект завжди створюється повторно при зміні одної з його залежностей.

Проте, у деяких випадках, це може бути надлишковим, так само як у прикладі з підпискою у попередньому розділі. Нам не потрібно створювати нову підписку для кожного оновлення, а лише тоді, коли змінився проп `source`.

Щоб реалізувати це, передайте другим аргументом до `useEffect` масив значень, від яких залежить ефект. Наш оновлений ефект тепер виглядає так:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

Зараз підписка буде створена повторно лише при зміні `props.source`.

>Примітка
>
>Якщо ви використовуєте цю оптимізацію, впевніться, що масив включає **всі значення з області видимості компонента (такі як пропси чи стан), що можуть змінюватись протягом часу і використовуються ефектом**. Інакше, ваш код буде посилатись на застарілі значення з попередніх рендерів. Дізнайтеся більше про те, [як мати справу з функціями](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) та що робити, коли [значення масиву змінюються надто часто](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>Якщо ви хочете запустити ефект і очистити його лише раз (при монтуванні і розмонтуванні), ви можете передати другим аргументом порожній масив (`[]`). React буде вважати, що ваш ефект не залежить від *жодного* із значень пропсів чи стану, а тому не потребує повторного запуску. Це не оброблюється як особливий випадок, а напряму випливає з роботи масиву залежностей.
>
>Якщо ви передаєте порожній масив (`[]`), пропси і стан усередині ефекту будуть завжди мати їх початкові значення. Передача другим аргументом `[]`, нагадує модель роботи вже знайомих `componentDidMount` та `componentWillUnmount`, але зазвичай є [кращі](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) [рішення](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) для уникнення частих повторних викликів ефектів. Також не забудьте, що React відкладає виконання `useEffect` до моменту відображення вмісту браузером, а отже можливість виконання додаткової роботи не є істотною проблемою.
>
>
>Ми радимо використовувати правило [`exhaustive-deps`](https://github.com/facebook/react/issues/14920), як частину нашого пакунку [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Воно попереджує про те, що залежності вказані невірно і пропонує рішення.

Масив залежностей не передається у якості аргументів до функції ефекту. Концептуально, проте, це те, що вони представляють: кожне значення, на яке посилається функція ефекту, також має з'являтись у масиві залежностей. У майбутньому, достатньо просунутий компілятор зможе створити цей масив автоматично.

### `useContext` {#usecontext}

> Try the new React documentation for [`useContext`](https://beta.reactjs.org/reference/react/useContext).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const value = useContext(MyContext);
```

Приймає об'єкт контексту (значення, повернуте з `React.createContext`) і повертає поточне значення контексту для нього. Поточне значення контексту визначається пропом `value` найближчого `<MyContext.Provider>`, що знаходиться вище у дереві компонентів.

Коли найближчий `<MyContext.Provider>`, що знаходиться вище поточного компонента, оновлюється, цей хук викличе повторний рендер з актуальним `value` контексту, переданим до провайдера `MyContext`. Навіть якщо предок використовує [`React.memo`](/docs/react-api.html#reactmemo) або [`shouldComponentUpdate`](/docs/react-component.html#shouldcomponentupdate), ререндер все одно відбудеться, починаючи з поточного компонента та використовуючи `useContext`.

Не забудьте про те, що аргумент переданий у `useContext`, повинен бути *власне об'єктом контексту*:

 * **Правильно:** `useContext(MyContext)`
 * **Неправильно:** `useContext(MyContext.Consumer)`
 * **Неправильно:** `useContext(MyContext.Provider)`

Компонент, що викликає `useContext`, завжди повторно відрендериться при зміні значення контексту. Якщо повторний рендер є вартісною операцією, ви можете [оптимізувати його, використавши мемоізацію](https://github.com/facebook/react/issues/15156#issuecomment-474590693).

>Порада
>
>Якщо ви ознайомились з API контексту до хуків, `useContext(MyContext)` є еквівалентним `static contextType = MyContext` у класі чи `<MyContext.Consumer>`.
>
>`useContext(MyContext)` дозволяє лише *читати* контекст і підписуватись на його зміни. Вам і досі необхідно мати `<MyContext.Provider>` вище у дереві, щоб *надати* цьому контексту значення.

**Давайте використаємо Context.Provider**
```js{31-36}
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```
Цей приклад – версія прикладу з [документації контекста](/docs/context.html), адаптована під хуки. Там ви зможете знайти більше інформації про те, як і коли використовувати контекст.


## Додаткові хуки {#additional-hooks}

Наступні хуки є або варіантами базових із розділу вище, або потрібні у вкрай специфічних випадках. Не потрібно вивчати їх наперед.

### `useReducer` {#usereducer}

> Try the new React documentation for [`useReducer`](https://beta.reactjs.org/reference/react/useReducer).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

Є альтернативою [`useState`](#usestate). Приймає редюсер типу `(state, action) => newState` і повертає поточний стан у парі з методом `dispatch`. (Якщо ви знайомі з Redux, ви вже знаєте як це працює.)

Слід віддати перевагу `useReducer` над `useState`, коли ви маєте складну логіку стану, що включає багато значень чи ваш наступний стан залежить від попереднього. Крім того, `useReducer` дозволяє вам оптимізувати продуктивність для компонентів, що викликають глибокі оновлення, тому що [ви можете передати вниз `dispatch`, замість функцій повторного виклику](/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down).

Ось приклад лічильника із розділу [`useState`](#usestate), переписаний із використанням редюсера:

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

>Примітка
>
>React гарантує, що функція `dispatch` зберігає ідентичність і не змінюється під час повторних рендерів. Саме тому, ви можете безпечно пропускати її включення до списків залежностей хуків `useEffect` чи `useCallback`.

#### Визначення початкового стану {#specifying-the-initial-state}

Є два різних шляхи ініціалізації стану `useReducer`. Ви можете обрати будь-який, залежно від ситуації. Найпростішим варіантом є передача початкового стану другим аргументом:

```js{3}
  const [state, dispatch] = useReducer(
    reducer,
    {count: initialCount}
  );
```

>Примітка
>
>React не використовує `state = initialState` конвенцію про аргументи, популяризовану в Redux. Початкове значення часом залежить від пропсів, а тому вказується безпосередньо у виклиці хука. Якщо ви впевнені щодо цього, ви можете викликати `useReducer(reducer, undefined, reducer)`, щоб земулювати поведінку Redux, але робити так не рекомендується.

#### Лінива ініціалізація {#lazy-initialization}

Ви також можете ліниво створити початковий стан. Щоб зробити це, ви можете передати функцію `init` третім аргументом. Початковий стан буде встановлений у `init(initialArg)`.

Це дозволить винести логіку обчислення початкового стану з редюсера. Також це може бути корисним при скиданні стану пізніше у відповідь на дію:

```js{1-3,11-12,19,24}
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Лічильник: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Скинути
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

#### Припинення дії dispatch {#bailing-out-of-a-dispatch}

Якщо ви повернете з хука редюсера значення, що дорівнює поточному стану, React вийде з нього без рендерингу дочірніх елементів чи запуску ефектів. (React використовує [алгоритм порівняння `Object.is`](https://developer.mozilla.org/uk/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).)

Зверніть увагу, що React-у може знадобитись відрендерити конкретний компонент перед припиненням оновлення. Це не повинно викликати занепокоєння, тому що React необов'язково опуститься "глибше" в дерево. Якщо ви здійснюєте вартісні обчислення під час рендерингу, ви можете оптимізувати їх, використавши `useMemo`.

### `useCallback` {#usecallback}

> Try the new React documentation for [`useCallback`](https://beta.reactjs.org/reference/react/useCallback).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

Повертає [мемоізовану](https://uk.wikipedia.org/wiki/%D0%9C%D0%B5%D0%BC%D0%BE%D1%96%D0%B7%D0%B0%D1%86%D1%96%D1%8F) функцію зворотнього виклику.

Передайте вбудовану функцію зворотнього виклику і масив залежностей. `useCallback` поверне мемоізовану версію функції зворотнього виклику, котра змінюється лише тоді, коли одна з її залежностей змінюється. Це корисно при передачі функцій зворотнього виклику до оптимізованих дочірніх компонентів, що покладаються на рівність посилань задля уникнення непотрібних рендерів (наприклад, `shouldComponentUpdate`).

`useCallback(fn, deps)` є еквівалентом `useMemo(() => fn, deps)`.

> Примітка
>
> Масив залежностей не передається у якості аргументів до функції зворотнього виклику. Концептуально, проте, це те, що вони представляють: кожне значення, на яке посилається функція зворотнього виклику, також має з'являтись у масиві залежностей. У майбутньому, достатньо просунутий компілятор зможе створити цей масив автоматично.
>
> Ми радимо використовувати правило [`exhaustive-deps`](https://github.com/facebook/react/issues/14920), як частину нашого пакунку [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Воно попереджує про те, що залежності вказані невірно і пропонує рішення.

### `useMemo` {#usememo}

> Try the new React documentation for [`useMemo`](https://beta.reactjs.org/reference/react/useMemo).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Повертає [мемоізоване](https://en.wikipedia.org/wiki/Memoization) значення.

Передайте функцію "створення" та масив залежностей. `useMemo` повторно обчислить мемоізоване значення лише при зміні однієї з залежностей. Така оптимізація допомагає уникнути вартісних обчислень при кожному рендері.

Пам'ятайте, що функція, передана до `useMemo`, запускається під час рендерингу. Не робіть у ній нічого, що ви зазвичай не робите під час рендерингу. Наприклад, побічні ефекти мають бути в `useEffect`, а не `useMemo`.

Якщо масив не наданий, нове значення буде обчислене при кожному рендері.

**Ви можете покластись на `useMemo` як на оптимізацію продуктивності, а не на семантичу гарантію.** У майбутньому React може вирішити "забути" деякі попередньо мемоізовані значення і переобчислити їх при наступному рендері, наприклад, для звільнення пам'яті для компонентів поза областю видимості екрана. Напишіть ваш код так, щоб він працював без `useMemo`, а потім додайте його для оптимізації продуктивності.

> Примітка
>
> Масив залежностей не передається у якості аргументів до функції. Концептуально, проте, це те, що вони представляють: кожне значення, на яке посилається функція, також має з'являтись у масиві залежностей. У майбутньому, достатньо просунутий компілятор зможе створити цей масив автоматично.
>
> Ми радимо використовувати правило [`exhaustive-deps`](https://github.com/facebook/react/issues/14920), як частину нашого пакунку [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Воно попереджує про те, що залежності вказані невірно і пропонує рішення.

### `useRef` {#useref}

> Try the new React documentation for [`useRef`](https://beta.reactjs.org/reference/react/useRef).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const refContainer = useRef(initialValue);
```

`useRef` поверне змінний об'єкт рефу, властивість `.current` якого ініціалізується переданим аргументом (`initialValue`). Повернутий об'єкт буде зберігатись протягом всього часу життя компонента.

Поширеним випадком використання є імперативний доступ до потомків:

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` вказує на примонтований елемент поля вводу тексту
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Фокусуватись на полі вводу</button>
    </>
  );
}
```

По суті, `useRef` — це "коробка", що може містити змінне значення у власній властивості `.current`.

Рефи можуть бути вам знайомі перш за все як [засіб доступу до DOM](/docs/refs-and-the-dom.html). Якщо ви передасте об'єкт рефу у React як `<div ref={myRef} />`, React встановить його властивість `.current` рівною значенню відповідного DOM вузла при будь-якій зміні цього вузла.

Проте `useRef()` є корисним не тільки для простого атрибута `ref`. Він [згодиться для постійного збереження будь-якого змінного значення](/docs/hooks-faq.html#is-there-something-like-instance-variables) подібно до використання полей екземпляра класу.

Це можливо, тому що `useRef()` створює простий JavaScript-об'єкт. Єдина різниця між `useRef()` і створенням об'єкта `{current: ...}` власноруч полягає в тому, що `useRef` поверне один і той самий реф-об'єкт при кожному рендері.

Пам'ятайте, що `useRef` *не* повідомляє вас про зміну свого вмісту. Зміна властивості `.current` не спричинить повторний рендер. Якщо ви хочете запустити деякий код під час того, як React прикріплює чи від'єднує реф до вузла DOM, то вам краще використати [реф зворотнього виклику](/docs/hooks-faq.html#how-can-i-measure-a-dom-node).


### `useImperativeHandle` {#useimperativehandle}

> Try the new React documentation for [`useImperativeHandle`](https://beta.reactjs.org/reference/react/useImperativeHandle).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` налаштовує значення екземпляра, яке надається батьківським компонентам при використанні `ref`. Як і зазвичай, у більшості випадків ви маєте уникати імперативного коду з використанням рефів. `useImperativeHandle` має використовуватись разом з [`forwardRef`](/docs/react-api.html#reactforwardref):

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

У цьому прикладі батьківський компонент, що рендерить `<FancyInput ref={inputRef} />`, матиме змогу викликати `inputRef.current.focus()`.

### `useLayoutEffect` {#uselayouteffect}

<<<<<<< HEAD
Сигнатура хука є ідентичною `useEffect`, але він запускається синхронно після всіх змін DOM. Використовуйте це для читання розмітки з DOM і синхронних повторних рендерів. Оновлення, заплановані всередині `useLayoutEffect`, будуть виконані синхронно перед тим, як браузер відобразить їх.
=======
> Try the new React documentation for [`useLayoutEffect`](https://beta.reactjs.org/reference/react/useLayoutEffect).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


The signature is identical to `useEffect`, but it fires synchronously after all DOM mutations. Use this to read layout from the DOM and synchronously re-render. Updates scheduled inside `useLayoutEffect` will be flushed synchronously, before the browser has a chance to paint.
>>>>>>> 5647a9485db3426d62b5a8203f4499c01bcd789b

Віддайте перевагу використанню `useEffect`, коли це можливо, щоб уникнути блокування візуальних оновлень.

> Порада
>
> Якщо ви переробляєте код класового компонента, зверніть увагу, що `useLayoutEffect` запускається на тому ж етапі, що й `componentDidMount` та `componentDidUpdate`. Однак **ми рекомендуємо розпочати з використання `useEffect`** і спробувати `useLayoutEffect` тільки при виникненні проблем.
>
>Якщо ви використовуєте серверний рендеринг, пам'ятайте, що *ані* `useLayoutEffect`, ні `useEffect` не можуть бути запущені допоки JavaScript не завантажився. Саме тому React попереджує про те, що компонент, який рендериться на сервері, містить `useLayoutEffect`. Щоб виправити це, ви можете помістити цю логіку в `useEffect` (якщо вона не потрібна при першому рендері) або відкласти відображення цього компонента до моменту, коли відрендериться клієнт (якщо HTML виглядає невалідним до запуску `useLayoutEffect`).
>
>Щоб виключити компонент, що потребує ефектів розмітки, із відрендереного на сервері HTML, відрендеріть його умовно з допомогою `showChild && <Child />` і відкладіть його відображення, використавши `useEffect(() => { setShowChild(true); }, [])`. Таким чином, інтерфейс користувача не буде виглядати неправильно перед гідратацією.

### `useDebugValue` {#usedebugvalue}

> Try the new React documentation for [`useDebugValue`](https://beta.reactjs.org/reference/react/useDebugValue).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
useDebugValue(value)
```

`useDebugValue` може бути використаний для відображення мітки користувацьких хуків у інструментах розробника React.

Наприклад, ми маємо користувацький хук `useFriendStatus` описаний у розділі ["Створення користувацьких хуків"](/docs/hooks-custom.html):

```js{6-8}
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // Показує мітку у інструментах розробника поруч з цим хуком
  // наприклад, "FriendStatus: В мережі"
  useDebugValue(isOnline ? 'В мережі' : 'Не в мережі');

  return isOnline;
}
```

> Порада
>
> Ми не радимо додавати значення для налагодження до кожного користувацького хука. Вони найбільш цінні для користувацьких хуків, що є частиною загальнодоступних бібліотек.

#### Відкладення форматування значень для налагодження {#defer-formatting-debug-values}

У певних випадках, форматування значення для відображення може бути вартісною операцією. Також воно не потрібне у випадку, коли хук не перевіряється безпосередньо.

З цієї причини `useDebugValue` приймає функцію форматування у якості необов'язкового другого параметра. Ця функція викликається лише при перевірці хуків. Вона отримує налагоджувальне значення у якості параметра і повинна повертати форматоване значення для відображення.

Для прикладу, користувацький хук, що повертає значення `Date`, може уникнути викликання функції `toDateString`, передавши наступну функцію форматування:

```js
useDebugValue(date, date => date.toDateString());
```

### `useDeferredValue` {#usedeferredvalue}

> Try the new React documentation for [`useDeferredValue`](https://beta.reactjs.org/reference/react/useDeferredValue).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const deferredValue = useDeferredValue(value);
```

`useDeferredValue` accepts a value and returns a new copy of the value that will defer to more urgent updates. If the current render is the result of an urgent update, like user input, React will return the previous value and then render the new value after the urgent render has completed.

This hook is similar to user-space hooks which use debouncing or throttling to defer updates. The benefits to using `useDeferredValue` is that React will work on the update as soon as other work finishes (instead of waiting for an arbitrary amount of time), and like [`startTransition`](/docs/react-api.html#starttransition), deferred values can suspend without triggering an unexpected fallback for existing content.

#### Memoizing deferred children {#memoizing-deferred-children}
`useDeferredValue` only defers the value that you pass to it. If you want to prevent a child component from re-rendering during an urgent update, you must also memoize that component with [`React.memo`](/docs/react-api.html#reactmemo) or [`React.useMemo`](/docs/hooks-reference.html#usememo):

```js
function Typeahead() {
  const query = useSearchQuery('');
  const deferredQuery = useDeferredValue(query);

  // Memoizing tells React to only re-render when deferredQuery changes,
  // not when query changes.
  const suggestions = useMemo(() =>
    <SearchSuggestions query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <>
      <SearchInput query={query} />
      <Suspense fallback="Loading results...">
        {suggestions}
      </Suspense>
    </>
  );
}
```

Memoizing the children tells React that it only needs to re-render them when `deferredQuery` changes and not when `query` changes. This caveat is not unique to `useDeferredValue`, and it's the same pattern you would use with similar hooks that use debouncing or throttling.

### `useTransition` {#usetransition}

> Try the new React documentation for [`useTransition`](https://beta.reactjs.org/reference/react/useTransition).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const [isPending, startTransition] = useTransition();
```

Returns a stateful value for the pending state of the transition, and a function to start it.

`startTransition` lets you mark updates in the provided callback as transitions:

```js
startTransition(() => {
  setCount(count + 1);
});
```

`isPending` indicates when a transition is active to show a pending state:

```js
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);
  
  function handleClick() {
    startTransition(() => {
      setCount(c => c + 1);
    });
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```

> Note:
>
> Updates in a transition yield to more urgent updates such as clicks.
>
> Updates in a transition will not show a fallback for re-suspended content. This allows the user to continue interacting with the current content while rendering the update.

### `useId` {#useid}

> Try the new React documentation for [`useId`](https://beta.reactjs.org/reference/react/useId).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const id = useId();
```

`useId` is a hook for generating unique IDs that are stable across the server and client, while avoiding hydration mismatches.

> Note
>
> `useId` is **not** for generating [keys in a list](/docs/lists-and-keys.html#keys). Keys should be generated from your data.

For a basic example, pass the `id` directly to the elements that need it:

```js
function Checkbox() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Do you like React?</label>
      <input id={id} type="checkbox" name="react"/>
    </>
  );
};
```

For multiple IDs in the same component, append a suffix using the same `id`:

```js
function NameFields() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id + '-firstName'}>First Name</label>
      <div>
        <input id={id + '-firstName'} type="text" />
      </div>
      <label htmlFor={id + '-lastName'}>Last Name</label>
      <div>
        <input id={id + '-lastName'} type="text" />
      </div>
    </div>
  );
}
```

> Note:
> 
> `useId` generates a string that includes the `:` token. This helps ensure that the token is unique, but is not supported in CSS selectors or APIs like `querySelectorAll`.
> 
> `useId` supports an `identifierPrefix` to prevent collisions in multi-root apps. To configure, see the options for [`hydrateRoot`](/docs/react-dom-client.html#hydrateroot) and [`ReactDOMServer`](/docs/react-dom-server.html).

## Library Hooks {#library-hooks}

The following Hooks are provided for library authors to integrate libraries deeply into the React model, and are not typically used in application code.

### `useSyncExternalStore` {#usesyncexternalstore}

> Try the new React documentation for [`useSyncExternalStore`](https://beta.reactjs.org/reference/react/useSyncExternalStore).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)


```js
const state = useSyncExternalStore(subscribe, getSnapshot[, getServerSnapshot]);
```

`useSyncExternalStore` is a hook recommended for reading and subscribing from external data sources in a way that's compatible with concurrent rendering features like selective hydration and time slicing.

This method returns the value of the store and accepts three arguments:
- `subscribe`: function to register a callback that is called whenever the store changes.
- `getSnapshot`: function that returns the current value of the store.
- `getServerSnapshot`: function that returns the snapshot used during server rendering.

The most basic example simply subscribes to the entire store:

```js
const state = useSyncExternalStore(store.subscribe, store.getSnapshot);
```

However, you can also subscribe to a specific field:

```js
const selectedField = useSyncExternalStore(
  store.subscribe,
  () => store.getSnapshot().selectedField,
);
```

When server rendering, you must serialize the store value used on the server, and provide it to `useSyncExternalStore`. React will use this snapshot during hydration to prevent server mismatches:

```js
const selectedField = useSyncExternalStore(
  store.subscribe,
  () => store.getSnapshot().selectedField,
  () => INITIAL_SERVER_SNAPSHOT.selectedField,
);
```

> Note:
>
> `getSnapshot` must return a cached value. If getSnapshot is called multiple times in a row, it must return the same exact value unless there was a store update in between.
> 
> A shim is provided for supporting multiple React versions published as `use-sync-external-store/shim`. This shim will prefer `useSyncExternalStore` when available, and fallback to a user-space implementation when it's not.
> 
> As a convenience, we also provide a version of the API with automatic support for memoizing the result of getSnapshot published as `use-sync-external-store/with-selector`.

### `useInsertionEffect` {#useinsertioneffect}

> Try the new React documentation for [`useInsertionEffect`](https://beta.reactjs.org/reference/react/useInsertionEffect).
>
> The new docs will soon replace this site, which will be archived. [Provide feedback.](https://github.com/reactjs/reactjs.org/issues/3308)

```js
useInsertionEffect(didUpdate);
```

The signature is identical to `useEffect`, but it fires synchronously _before_ all DOM mutations. Use this to inject styles into the DOM before reading layout in [`useLayoutEffect`](#uselayouteffect). Since this hook is limited in scope, this hook does not have access to refs and cannot schedule updates.

> Note:
>
> `useInsertionEffect` should be limited to css-in-js library authors. Prefer [`useEffect`](#useeffect) or [`useLayoutEffect`](#uselayouteffect) instead.
