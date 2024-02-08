<!-- adapted from: https://gist.github.com/janwillemtulp/8968bb6cc207b329913eee893a35bc66 -->
<script>
	let answer = 'myPassword';
	let password = '';
	const hash = (s) =>
		s.split('').reduce((a, b) => {
			a = (a << 5) - a + b.charCodeAt(0);
			return a & a;
		}, 0);
	$: console.log(password, hash(password));
</script>

{#if hash(password) === hash(answer)}
	<slot />
{:else}
	<div>
		<label for="password">Password:</label>
		<input id="password" bind:value={password} type="password" />
	</div>
{/if}

<style>
	div {
		font-family: sans-serif;
		font-size: 12px;
		margin-left: 50%;
		width: 400px;
		margin-top: 100px;
	}
	label {
		font-weight: bold;
	}
	input[type='password'] {
		width: 200px;
	}
</style>
