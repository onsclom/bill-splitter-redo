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

	let taxPercentage = 8.1;
	$: tax = subtotal * (taxPercentage / 100);

	let tipPercentage = 20;
	$: tip = subtotal * (tipPercentage / 100);

	let items = [] as {
		name: string;
		value: number;
		payers: string[];
		id: string;
	}[];

	let itemNumber = items.length;
	let itemName = `item ${++itemNumber}`;
	let itemValue = 10;
	let itemPayers = [] as string[];

	$: itemSum = items.reduce((sum, item) => sum + item.value, 0);
	$: itemsMatchSubtotal = itemSum.toFixed(2) === subtotal.toFixed(2);
</script>

<h1>bill splitter 🧾 💵</h1>

<section>
	<h2>people</h2>
	{#each people as person}
		<div style="display: flex; gap: 1rem; align-items: center;">
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
		+ 😃
	</button>
</section>

<section>
	<h2>totals</h2>
	<p>
		<label>
			subtotal $<input type="number" bind:value={subtotal} />
		</label>
	</p>

	<div style="display: flex; gap: 1rem; align-items: center">
		<label>
			tax %<input type="number" autocomplete="off" bind:value={taxPercentage} />
		</label>
		<span> = </span>
		<label>
			tax $<input type="number" autocomplete="off" bind:value={tax} disabled />
		</label>
	</div>

	<div style="display: flex; gap: 1rem; align-items: center">
		<label>
			tip %<input type="number" autocomplete="off" bind:value={tipPercentage} />
		</label>
		<span> = </span>
		<label>
			tip $<input type="number" autocomplete="off" bind:value={tip} disabled />
		</label>
	</div>

	<p>
		total <strong>
			${(subtotal + tax + tip).toFixed(2)}
		</strong>
	</p>
</section>

<section>
	<h2>items</h2>

	{#each items as item}
		<div>
			<button on:click={() => (items = items.filter((i) => i !== item))}> ❌ </button>
			<strong>{item.name}</strong> -
			<strong>${item.value}</strong>
			{item.payers.length > 1 ? ' split between ' : ' paid by '}
			<strong>
				{item.payers.map((payer) => people.find((person) => person.id === payer)?.name).join(', ')}
			</strong>
		</div>
	{/each}

	<br />

	<div>
		<label>name <input type="text" bind:value={itemName} /></label>
		<label>price $<input type="number" bind:value={itemValue} /></label>
		<label>
			payers <br />
			{#each people as person}
				<label>
					<input type="checkbox" bind:group={itemPayers} value={person.id} />
					{person.name}
					{itemPayers.includes(person.id)
						? `($${Number((itemValue / itemPayers.length).toFixed(2))})`
						: ''}
				</label>
			{/each}
		</label>
		<button
			on:click={() => {
				items = [
					...items,
					{ name: itemName, value: itemValue, payers: itemPayers, id: crypto.randomUUID() }
				];
				itemName = `item ${++itemNumber}`;
				itemValue = 10;
				itemPayers = [];
			}}
			disabled={itemPayers.length === 0}
		>
			+ item
		</button>
	</div>
</section>

<section>
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
</section>
