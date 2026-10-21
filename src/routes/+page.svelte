<script>
	let method = $state('caesar');
	let encryptText = $state('');
	let decryptText = $state('');
	let encryptCaesarKey = $state('');
	let decryptCaesarKey = $state('');
	let encryptVigenereKey = $state('');
	let decryptVigenereKey = $state('');
	let decryptOtpKey = $state('');
	let encryptedText = $state('');
	let decryptedText = $state('');
	let generatedOtpKey = $state('');
	let encryptError = $state('');
	let decryptError = $state('');

	function clearResults() {
		encryptedText = '';
		decryptedText = '';
		generatedOtpKey = '';
		encryptError = '';
		decryptError = '';
	}

	/** @param {string} text */
	function isValidText(text) {
		return /^[A-Za-z ]+$/.test(text) && /[A-Za-z]/.test(text);
	}

	function encryptTextValue() {
		encryptError = '';
		encryptedText = '';
		generatedOtpKey = '';

		if (!isValidText(encryptText)) {
			encryptError = 'Skriv inn tekst med engelske bokstaver og mellomrom. Teksten må inneholde minst én bokstav.';
			return;
		}

		if (method === 'caesar') {
			if (!/^(?:[0-9]|1[0-9]|2[0-5])$/.test(encryptCaesarKey)) {
				encryptError = 'Caesar-nøkkelen må være et heltall fra 0 til 25.';
				return;
			}

			encryptedText = transformText(encryptText, String.fromCharCode(65 + Number(encryptCaesarKey)), false);
			return;
		}

		if (method === 'vigenere') {
			if (!/^[A-Za-z]+$/.test(encryptVigenereKey)) {
				encryptError = 'Vigenère-nøkkelen må bare inneholde bokstaver fra A til Z.';
				return;
			}

			encryptedText = transformText(encryptText, encryptVigenereKey.toUpperCase(), false);
			return;
		}

		// One-Time Pad bruker en ny, tilfeldig nøkkelbokstav for hver bokstav i teksten.
		generatedOtpKey = Array.from(encryptText)
			.filter((character) => character !== ' ')
			.map(() => String.fromCharCode(65 + Math.floor(Math.random() * 26)))
			.join('');
		encryptedText = transformText(encryptText, generatedOtpKey, false);
	}

	function decryptTextValue() {
		decryptError = '';
		decryptedText = '';

		if (!isValidText(decryptText)) {
			decryptError = 'Skriv inn tekst med engelske bokstaver og mellomrom. Teksten må inneholde minst én bokstav.';
			return;
		}

		if (method === 'caesar') {
			if (!/^(?:[0-9]|1[0-9]|2[0-5])$/.test(decryptCaesarKey)) {
				decryptError = 'Caesar-nøkkelen må være et heltall fra 0 til 25.';
				return;
			}

			decryptedText = transformText(
				decryptText,
				String.fromCharCode(65 + Number(decryptCaesarKey)),
				true
			);
			return;
		}

		if (method === 'vigenere') {
			if (!/^[A-Za-z]+$/.test(decryptVigenereKey)) {
				decryptError = 'Vigenère-nøkkelen må bare inneholde bokstaver fra A til Z.';
				return;
			}

			decryptedText = transformText(decryptText, decryptVigenereKey.toUpperCase(), true);
			return;
		}

		if (!/^[A-Za-z]+$/.test(decryptOtpKey)) {
			decryptError = 'One-Time Pad-nøkkelen må bare inneholde bokstaver fra A til Z.';
			return;
		}

		const letterCount = Array.from(decryptText).filter((character) => character !== ' ').length;
		if (decryptOtpKey.length !== letterCount) {
			decryptError = `Nøkkelen må ha ${letterCount} bokstaver, én for hver bokstav i teksten (mellomrom telles ikke).`;
			return;
		}

		decryptedText = transformText(decryptText, decryptOtpKey.toUpperCase(), true);
	}

	// Bokstaver og nøkkel behandles som tall fra 0 til 25; mellomrom bruker ingen nøkkelbokstav.
	/** @param {string} text @param {string} key @param {boolean} decrypt */
	function transformText(text, key, decrypt) {
		let keyIndex = 0;

		return Array.from(text)
			.map((character) => {
				if (character === ' ') return character;

				const isLowerCase = character >= 'a' && character <= 'z';
				const textValue = character.toUpperCase().charCodeAt(0) - 65;
				const keyValue = key.charCodeAt(keyIndex % key.length) - 65;
				const shiftedValue = (textValue + (decrypt ? -keyValue : keyValue) + 26) % 26;
				keyIndex += 1;

				const result = String.fromCharCode(65 + shiftedValue);
				return isLowerCase ? result.toLowerCase() : result;
			})
			.join('');
	}
</script>

<svelte:head>
	<title>Kryptografiverktøy</title>
	<meta
		name="description"
		content="Et enkelt verktøy for Caesar, Vigenère og One-Time Pad."
	/>
</svelte:head>

<main>
	<header class="page-header">
		<p class="eyebrow">Kryptografi · demonstrasjon</p>
		<h1>Kryptografiverktøy</h1>
		<p class="intro">
			Krypter og dekrypter tekst med Caesar, Vigenère eller One-Time Pad.
		</p>
	</header>

	<section class="method-picker" aria-labelledby="method-label">
		<label id="method-label" for="method">Velg krypteringsmetode</label>
		<select id="method" bind:value={method} onchange={clearResults}>
			<option value="caesar">Caesar</option>
			<option value="vigenere">Vigenère</option>
			<option value="otp">One-Time Pad</option>
		</select>
	</section>

	<div class="cards">
		<section class="card" aria-labelledby="encrypt-heading">
			<h2 id="encrypt-heading">Krypter</h2>
			<label for="encrypt-text">Tekst</label>
			<textarea
				id="encrypt-text"
				bind:value={encryptText}
				placeholder="Skriv inn teksten du vil kryptere"
				rows="4"
			></textarea>

			{#if method === 'caesar'}
				<label for="encrypt-caesar-key">Nøkkel (0–25)</label>
				<input
					id="encrypt-caesar-key"
					type="number"
					min="0"
					max="25"
					step="1"
					bind:value={encryptCaesarKey}
					placeholder="For eksempel 3"
				/>
			{:else if method === 'vigenere'}
				<label for="encrypt-vigenere-key">Vigenère-nøkkel</label>
				<input
					id="encrypt-vigenere-key"
					bind:value={encryptVigenereKey}
					placeholder="Bare bokstaver A–Z"
				/>
			{:else}
				<p class="hint">En tilfeldig nøkkel genereres automatisk når du krypterer.</p>
			{/if}

			<button type="button" onclick={encryptTextValue}>Krypter tekst</button>

			{#if encryptError}
				<p class="message error" role="alert">{encryptError}</p>
			{/if}
			{#if encryptedText}
				<div class="result" aria-live="polite">
					<h3>Ciphertext</h3>
					<output>{encryptedText}</output>
				</div>
			{/if}
			{#if generatedOtpKey}
				<div class="result key-result" aria-live="polite">
					<h3>One-Time Pad-nøkkel</h3>
					<output>{generatedOtpKey}</output>
					<p>Ta vare på nøkkelen. Den trengs for å dekryptere teksten.</p>
				</div>
			{/if}
		</section>

		<section class="card" aria-labelledby="decrypt-heading">
			<h2 id="decrypt-heading">Dekrypter</h2>
			<label for="decrypt-text">Ciphertext</label>
			<textarea
				id="decrypt-text"
				bind:value={decryptText}
				placeholder="Skriv inn teksten du vil dekryptere"
				rows="4"
			></textarea>

			{#if method === 'caesar'}
				<label for="decrypt-caesar-key">Nøkkel (0–25)</label>
				<input
					id="decrypt-caesar-key"
					type="number"
					min="0"
					max="25"
					step="1"
					bind:value={decryptCaesarKey}
					placeholder="For eksempel 3"
				/>
			{:else if method === 'vigenere'}
				<label for="decrypt-vigenere-key">Vigenère-nøkkel</label>
				<input
					id="decrypt-vigenere-key"
					bind:value={decryptVigenereKey}
					placeholder="Bare bokstaver A–Z"
				/>
			{:else}
				<label for="decrypt-otp-key">One-Time Pad-nøkkel</label>
				<input
					id="decrypt-otp-key"
					bind:value={decryptOtpKey}
					placeholder="Skriv inn nøkkelen manuelt"
				/>
			{/if}

			<button type="button" onclick={decryptTextValue}>Dekrypter tekst</button>

			{#if decryptError}
				<p class="message error" role="alert">{decryptError}</p>
			{/if}
			{#if decryptedText}
				<div class="result" aria-live="polite">
					<h3>Dechiffrert tekst</h3>
					<output>{decryptedText}</output>
				</div>
			{/if}
		</section>
	</div>

	<aside class="notice">
		<strong>Merk:</strong> One-Time Pad-nøkkelen bruker <code>Math.random()</code> for
		demonstrasjon. Den er ikke kryptografisk sikker og må aldri brukes til reell sikkerhet.
	</aside>
</main>

<style>
	:global(*) {
		box-sizing: border-box;
	}

	:global(body) {
		margin: 0;
		background: #f3f5f9;
		color: #1d2738;
		font-family:
			Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
			sans-serif;
	}

	main {
		width: min(100% - 2rem, 1050px);
		margin: 0 auto;
		padding: 3rem 0 4rem;
	}

	.page-header {
		margin-bottom: 2rem;
	}

	.eyebrow {
		margin: 0 0 0.5rem;
		color: #5266b5;
		font-size: 0.8rem;
		font-weight: 700;
		letter-spacing: 0.12em;
		text-transform: uppercase;
	}

	h1,
	h2,
	h3,
	p {
		margin-top: 0;
	}

	h1 {
		margin-bottom: 0.5rem;
		font-size: clamp(2rem, 5vw, 3rem);
		letter-spacing: -0.04em;
	}

	.intro {
		margin-bottom: 0;
		color: #5c6678;
		font-size: 1.05rem;
	}

	.method-picker,
	.card,
	.notice {
		border: 1px solid #e0e5ef;
		border-radius: 14px;
		background: #fff;
		box-shadow: 0 8px 25px rgb(25 38 70 / 5%);
	}

	.method-picker {
		display: grid;
		gap: 0.6rem;
		max-width: 360px;
		margin-bottom: 1.25rem;
		padding: 1rem;
	}

	.cards {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 1.25rem;
	}

	.card {
		display: flex;
		flex-direction: column;
		align-items: stretch;
		gap: 0.65rem;
		padding: 1.35rem;
	}

	h2 {
		margin-bottom: 0.35rem;
		font-size: 1.35rem;
	}

	label {
		font-size: 0.9rem;
		font-weight: 650;
	}

	input,
	select,
	textarea {
		width: 100%;
		border: 1px solid #cbd3e1;
		border-radius: 8px;
		background: #fff;
		color: inherit;
		font: inherit;
		padding: 0.72rem 0.8rem;
	}

	textarea {
		resize: vertical;
	}

	input:focus,
	select:focus,
	textarea:focus {
		border-color: #5266b5;
		outline: 3px solid rgb(82 102 181 / 17%);
	}

	button {
		margin-top: 0.35rem;
		border: 0;
		border-radius: 8px;
		background: #4358a8;
		color: #fff;
		cursor: pointer;
		font: inherit;
		font-weight: 700;
		padding: 0.8rem 1rem;
	}

	button:hover {
		background: #35488f;
	}

	button:focus-visible {
		outline: 3px solid #9ba9e3;
		outline-offset: 2px;
	}

	.hint,
	.result p {
		margin: 0;
		color: #5c6678;
		font-size: 0.9rem;
	}

	.result {
		display: grid;
		gap: 0.45rem;
		margin-top: 0.45rem;
		padding: 0.9rem;
		border-radius: 8px;
		background: #f3f5fa;
	}

	.result h3 {
		margin: 0;
		font-size: 0.9rem;
	}

	output {
		overflow-wrap: anywhere;
		font-family: ui-monospace, "Cascadia Code", Consolas, monospace;
	}

	.key-result {
		border: 1px solid #c6d0f2;
		background: #edf0fc;
	}

	.key-result output {
		font-size: 1.15rem;
		font-weight: 800;
		letter-spacing: 0.08em;
	}

	.message {
		margin: 0.25rem 0 0;
		font-size: 0.9rem;
	}

	.error {
		color: #b42318;
	}

	.notice {
		margin-top: 1.25rem;
		padding: 1rem 1.2rem;
		color: #5c4a1d;
		font-size: 0.92rem;
	}

	@media (max-width: 700px) {
		main {
			padding-top: 2rem;
		}

		.cards {
			grid-template-columns: 1fr;
		}
	}
</style>
