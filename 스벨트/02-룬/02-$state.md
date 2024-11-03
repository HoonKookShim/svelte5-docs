---
title: $state
---

`$state` 룬을 사용해서 _반응성 상태_ 를 만들 수 있습니다. 그러면 그 값이 바뀔 때 마다 UI가 _반응_ 하게 됩니다.

```svelte
<script>
	let count = $state(0);
</script>

<button onclick={() => count++}>
	clicks: {count}
</button>
```

당신이 지금까지 접해본 다른 프레임워크들과는 다르게, 상태를 처리하는 API는 존재하지 않습니다. — `count`는 객체나 함수가 아닌, 그냥 숫자이기 때문에, 다른 변수들한테 하는 것 처럼 그 값을 변경할 수 있습니다.

### Deep state

`$state`를 배열이나 간단한 객체에 사용한다면,  is used with an array or a simple object, the result is a deeply reactive _state proxy_.
[Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)를 사용하면 프로퍼티 값을 읽거나 쓸 때 스벨트가 코드를 실행시킬 수 있습니다.스벨트allow Svelte to run code when you read or write properties, including via methods like `array.push(...)`, triggering granular updates.

> [!NOTE] `Set`와 `Map` 같은 객체들은 프록시되지 않지만, 스벨트는  provides reactive implementations for various built-ins like these that can be imported from [`svelte/reactivity`](./svelte-reactivity).

State is proxified recursively until Svelte finds something other than an array or simple object. In a case like this...


```js
let todos = $state([
	{
		done: false,
		text: 'add more todos'
	}
]);
```

...modifying an individual todo's property will trigger updates to anything in your UI that depends on that specific property:

```js
// @filename: ambient.d.ts
declare global {
	const todos: Array<{ done: boolean, text: string }>
}

// @filename: index.js
// ---cut---
todos[0].done = !todos[0].done;
```

If you push a new object to the array, it will also be proxified:

```js
// @filename: ambient.d.ts
declare global {
	const todos: Array<{ done: boolean, text: string }>
}

// @filename: index.js
// ---cut---
todos.push({
	done: false,
	text: 'eat lunch'
});
```

> [!NOTE] When you update properties of proxies, the original object is _not_ mutated.

### Classes

클래스 필드에도 `$state`를 사용할 수 있습니다. (public 이거나 private 모두 사용할 수 있습니다):

```js
// @errors: 7006 2554
class Todo {
	done = $state(false);
	text = $state();

	constructor(text) {
		this.text = text;
	}

	reset() {
		this.text = '';
		this.done = false;
	}
}
```

> [!NOTE] 컴파일러는 `done`과 `text`를 private field를 참조하는 `get`/`set`프로토타입 메소드로 변형시킵니다.

## `$state.raw`

객체와 배열의 깊은 반응성을 원하지 않을 때는 `$state.raw`를 사용하면 됩니다.

State declared with `$state.raw` cannot be mutated; it can only be _reassigned_. In other words, rather than assigning to a property of an object, or using an array method like `push`, replace the object or array altogether if you'd like to update it:

```js
let person = $state.raw({
	name: 'Heraclitus',
	age: 49
});

// this will have no effect
person.age += 1;

// this will work, because we're creating a new person
person = {
	name: 'Heraclitus',
	age: 50
};
```

This can improve performance with large arrays and objects that you weren't planning to mutate anyway, since it avoids the cost of making them reactive. Note that raw state can _contain_ reactive state (for example, a raw array of reactive objects).

## `$state.snapshot`

To take a static snapshot of a deeply reactive `$state` proxy, use `$state.snapshot`:

```svelte
<script>
	let counter = $state({ count: 0 });

	function onclick() {
		// Will log `{ count: ... }` rather than `Proxy { ... }`
		console.log($state.snapshot(counter));
	}
</script>
```

This is handy when you want to pass some state to an external library or API that doesn't expect a proxy, such as `structuredClone`.
