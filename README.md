<div align="center"><img src="https://i.imgur.com/LxEjWku.png"></img></div>
<div align="center"><h1>Roselect</h1></div>
<div align="center">
	<a href="https://www.npmjs.com/package/@rbxts/roselect">
		<img src="https://badge.fury.io/js/%40rbxts%2Froselect.svg"></img>
	</a>
</div>
<div align="center">
        Simple “selector” library for Rodux, ported from javascript's 
        <a href="https://www.npmjs.com/package/reselect">reselect</a>
        library.
</div>

## Usability
* Selectors can compute derived data, allowing Redux to store the minimal possible state.
* Selectors are efficient. A selector is not recomputed unless one of its arguments changes.
* Selectors are composable. They can be used as input to other selectors.

<details open><summary>luau</summary><p>

```lua
local roselect = require(path.to.roselect)
local createSelector = roselect.createSelector
local reduce = roselect.reduce

local function shopItemsSelector(state)
        return state.shop.items
end

local function taxPercentSelector(state)
        return state.shop.taxPercent
end

local subtotalSelector = createSelector(
	shopItemsSelector,
	function(items)
                return reduce(items, function(acc, item)
                        return acc + item.value
                end, 0)
        end
)

local taxSelector = createSelector(
        subtotalSelector,
        taxPercentSelector,
        function(subtotal, taxPercent)
                return subtotal * (taxPercent / 100)
        end
)

local totalSelector = createSelector(
        subtotalSelector,
	taxSelector,
        function(subtotal, tax)
                return { total = subtotal + tax }
        end
)

local exampleState = {
        shop = {
                taxPercent = 8,
                items = {
			{ name = "apple", value = 1.20 },
			{ name = "orange", value = 0.95 }
                }
        }
}

print(subtotalSelector(exampleState)) -- 2.15
print(taxSelector(exampleState))      -- 0.172
print(totalSelector(exampleState))    -- { total = 2.322 }
```
</p></details>
<details><summary>roblox-ts</summary><p>

```ts
import { createSelector } from "@rbxts/roselect";

interface State {
	shop: {
		taxPercent: number;
		items: { name: string; value: number; }[];
	}
}

const shopItemsSelector = (state: State) => state.shop.items;
const taxPercentSelector = (state: State) => state.shop.taxPercent;

const subtotalSelector = createSelector(
	shopItemsSelector,
	items => items.reduce((acc, item) => acc + item.value, 0)
);

const taxSelector = createSelector(
	subtotalSelector,
	taxPercentSelector,
	(subtotal, taxPercent) => subtotal * (taxPercent / 100)
);

export const totalSelector = createSelector(
	subtotalSelector,
	taxSelector,
	(subtotal, tax) => ({ total: subtotal + tax })
);

const exampleState = identity<State>({
	shop: {
		taxPercent: 8,
		items: [
			{ name: 'apple', value: 1.20 },
			{ name: 'orange', value: 0.95 },
		]
	}
});

print(subtotalSelector(exampleState)); // 2.15
print(taxSelector(exampleState));      // 0.172
print(totalSelector(exampleState));    // { total: 2.322 }
```
</p></details>

## Installation
### luau
```bash
git clone https://github.com/HylianBasement/roselect.git ./modules
```

### roblox-ts
The installation can be done via `npm i @rbxts/roselect`.

## Composing Selectors
It's important to mention that `createSelector` checks if the first parameter is an array of dependencies.
Since lua tables are both dictionary and list, it treats that parameter as an array of selectors that needs to be composed.
A memoized selector can itself be an input-selector to another memoized selector. Here is `getVisibleTodos` being used as an input-selector to a selector that further filters the todos by keyword:
<details open><summary>luau</summary><p>

```lua
local function getVisibilityFilter(state)
	return state.visibilityFilter
end

local function getTodos(state)
	return state.todos
end

local function getKeyword(state)
	return state.keyword
end

local getVisibleTodos = createSelector(
	{ getVisibilityFilter, getTodos },
	function(visibilityFilter, todos)
		if visibilityFilter == "SHOW_ALL" then
			return todos
		elseif visibilityFilter == "SHOW_COMPLETED" then
			local newTodos = {}

			for k, todo in pairs(todos) do
				if todo.completed then
					newTodos[k] = todo
				end
			end

			return newTodos
		elseif visibilityFilter == "SHOW_ACTIVE" then
			local newTodos = {}

			for k, todo in pairs(todos) do
				if not todo.completed then
					newTodos[k] = todo
				end
			end

			return newTodos
		end
	end
)

local getVisibleTodosFilteredByKeyword = createSelector(
	{ getVisibleTodos, getKeyword },
  	function(visibleTodos, keyword)
		local newTodos = {}

		for k, todo in pairs(visibleTodos) do
			if todo.text:find(keyword) then
				newTodos[k] = todo
			end
		end

		return newTodos
  	end
)

return {
	getVisibleTodos = getVisibleTodos
}
```

</p></details>

<details><summary>roblox-ts</summary><p>

```ts
const getVisibilityFilter = (state: State) => state.visibilityFilter;
const getTodos = (state: State) => state.todos;
const getKeyword = (state: State) => state.keyword;
 
export const getVisibleTodos = createSelector(
	[ getVisibilityFilter, getTodos ],
	(visibilityFilter, todos) => {
		switch (visibilityFilter) {
		case "SHOW_ALL":
			return todos;
		case "SHOW_COMPLETED":
			return todos.filter(t => t.completed);
		case "SHOW_ACTIVE":
			return todos.filter(t => !t.completed);
		}
	}
);

const getVisibleTodosFilteredByKeyword = createSelector(
	[ getVisibleTodos, getKeyword ],
	(visibleTodos, keyword) => visibleTodos.filter(
		todo => todo.text.includes(keyword)
	)
);
```

</p></details>

## Can Roselect be used without Rodux?
Definitely. Just like the JS library, Roselect has no dependencies on any other package,
so it can be used independently.

## License
This project is licensed under the [MIT License](LICENSE).