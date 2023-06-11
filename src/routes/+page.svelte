<script lang="ts">
	let people = [
		{
			name: 'Person 1',
			id: crypto.randomUUID(),
			input: null as HTMLInputElement | null
		},
		{
			name: 'Person 2',
			id: crypto.randomUUID(),
			input: null as HTMLInputElement | null
		},
		{
			name: 'Person 3',
			id: crypto.randomUUID(),
			input: null as HTMLInputElement | null
		}
	];

	let personNumber = people.length;
	let subtotal = 100;

	// TODO: implement two way bindings on these

	let taxPercentage = 6.825;
	$: tax = subtotal * (taxPercentage / 100);

	let tipPercentage = 20;
	$: tip = subtotal * (tipPercentage / 100);

	let items = [
		{
			name: 'item 1',
			value: 10,
			payers: [] as string[],
			id: crypto.randomUUID(),
			nameInput: null as HTMLInputElement | null,
			priceInput: null as HTMLInputElement | null
		}
	];

	let itemNumber = items.length;

	$: validItems = items.filter((item) => item.payers.length > 0);
	$: itemSum = validItems.reduce((sum, item) => sum + item.value, 0);
	$: itemsMatchSubtotal = itemSum.toFixed(2) === subtotal.toFixed(2);
</script>

<h1>bill splitter 🧾 💵</h1>

<h2>people</h2>
{#each people as person (person.id)}
	<div style="display: flex; gap: 1rem; align-items: baseline;">
		<input style="flex-grow: 1" bind:this={person.input} bind:value={person.name} />
		<button on:click={() => person.input?.select()}>✏️</button>
		<button on:click={() => (people = people.filter((p) => p !== person))}> ❌ </button>
	</div>
{/each}
<button
	style="text-align: center"
	on:click={() =>
		(people = [
			...people,
			{ name: `Person ${++personNumber}`, id: crypto.randomUUID(), input: null }
		])}
>
	+ person
</button>

<h2>totals</h2>
<p>
	<label>
		subtotal $<input inputmode="decimal" type="number" bind:value={subtotal} />
	</label>
</p>

<div style="display: flex; gap: 1rem; align-items: baseline">
	<label>
		tax %<input type="number" inputmode="decimal" autocomplete="off" bind:value={taxPercentage} />
	</label>
	<span> = </span>
	<label>
		tax $<input type="number" inputmode="decimal" autocomplete="off" bind:value={tax} disabled />
	</label>
</div>

<div style="display: flex; gap: 1rem; align-items: baseline">
	<label>
		tip %<input type="number" inputmode="decimal" autocomplete="off" bind:value={tipPercentage} />
	</label>
	<span> = </span>
	<label>
		tip $<input type="number" inputmode="decimal" autocomplete="off" bind:value={tip} disabled />
	</label>
</div>

<p>
	total <strong>
		${(subtotal + tax + tip).toFixed(2)}
	</strong>
</p>

<h2>items</h2>

{#each items as item (item.id)}
	<fieldset style={item.payers.length === 0 ? 'border-color: red' : ''}>
		<div style="display: flex; gap: 1rem; align-items: baseline">
			<input style="flex-grow: 1" type="text" bind:value={item.name} />
			<div style="display: flex; align-items: baseline">
				<span>$</span>
				<input style="flex-grow: 1" inputmode="decimal" type="number" bind:value={item.value} />
			</div>
			<button on:click={() => (items = items.filter((i) => i.id != item.id))}> ❌ </button>
		</div>
		<label>
			paid by <br />
			<div style="display: flex; gap: 0rem 1rem; flex-wrap: wrap">
				{#each people as person}
					<label style="white-space: nowrap">
						<input type="checkbox" bind:group={item.payers} value={person.id} />
						{person.name}
					</label>
				{/each}
			</div>
		</label>
	</fieldset>
{/each}

<div>
	<button
		on:click={() => {
			itemNumber++;
			items = [
				...items,
				{
					name: `item ${itemNumber}`,
					value: 10,
					payers: [],
					id: crypto.randomUUID(),
					nameInput: null,
					priceInput: null
				}
			];
		}}
	>
		+ item
	</button>
</div>

<h2>results</h2>

{#if itemsMatchSubtotal}
	<p>✅ items match subtotal</p>
	<table>
		<thead>
			<tr>
				<th>name</th>
				<th>subtotal</th>
				<th>total</th>
			</tr>
		</thead>
		<tbody>
			{#each people as person}
				<tr>
					<td>{person.name}</td>
					<td
						>${items
							.filter((item) => item.payers.includes(person.id))
							.reduce((sum, item) => sum + item.value / item.payers.length, 0)
							.toFixed(2)}</td
					>
					<td>
						<strong>
							${(
								items
									.filter((item) => item.payers.includes(person.id))
									.reduce((sum, item) => sum + item.value / item.payers.length, 0) *
								(1 + taxPercentage / 100 + tipPercentage / 100)
							).toFixed(2)}
						</strong>
					</td>
				</tr>
			{/each}
		</tbody>
	</table>
{:else if subtotal > itemSum}
	<p>❌ items are less than subtotal</p>
	<p>
		there should be
		<strong>${(subtotal - itemSum).toFixed(2)}</strong> more of items
	</p>
{:else}
	<p>❌ items are more than subtotal</p>
	<p>
		there should be
		<strong>${(itemSum - subtotal).toFixed(2)}</strong> less of items
	</p>
{/if}
