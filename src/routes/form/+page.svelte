<script>
	import { goto } from '$app/navigation';
	import Snake from '$lib/Snake.svelte';
	let nom = $state('');
	let prenom = $state('');

	// Messages
	let erreur = $state('');      // pour le formulaire
	let diceError = $state('');   // pour le mini-jeu de date / dé

	// Téléphone roulette séquentielle
	let digits = $state(Array(10).fill(null));
	digits[0] = 0;
	let currentDigit = $state(0);
	let currentIndex = $state(1);
	let intervalIdPhone = $state(null);

	// Date via mini-jeu de dés
	let dateField = $state('  /  /    ');
	let diceValue = $state(null); // chiffre final du dé
	let diceStep = $state(0); // position dans la date (0-7)
	let diceRolling = $state(false);
	let diceFace = $state(0); // chiffre affiché sur le cube pendant animation
	let diceAnim = $state(false);

	// --- Téléphone ---
	function resetPhone() {
		digits = Array(10).fill(null);
		digits[0] = 0;
		currentDigit = 0;
		currentIndex = 1;
		if (intervalIdPhone) clearInterval(intervalIdPhone);
	}

	function startPhone() {
		if (intervalIdPhone) clearInterval(intervalIdPhone);
		if (currentIndex >= digits.length) return;
		intervalIdPhone = setInterval(() => {
			currentDigit = (currentDigit + 1) % 10;
		}, 200);
	}

	function stopPhone() {
		if (intervalIdPhone) clearInterval(intervalIdPhone);

		if (currentIndex < digits.length) {
			digits[currentIndex] = currentDigit;
			currentIndex++;
		}

		if (currentIndex < digits.length) {
			startPhone();
		}
	}

	// --- Dé 3D ---
	function rollDice3D() {
		// Dès qu'on clique, on efface le message Game Over
		diceError = '';

		if (diceStep >= 8) return;
		diceRolling = true;
		diceAnim = true;
		let startTime = Date.now();
		const interval = setInterval(() => {
			diceFace = Math.floor(Math.random() * 10); // 0-9
			if (Date.now() - startTime > 1200) {
				clearInterval(interval);
				diceRolling = false;
				diceAnim = false;
				diceValue = diceFace;
			}
		}, 50);
	}

	function validateDice() {
		if (diceValue === null || diceError) return;
		const positions = [0,1,3,4,6,7,8,9];
		let arr = dateField.split('');
		arr[positions[diceStep]] = diceValue;
		dateField = arr.join('');
		diceStep++;
		diceValue = null;

		checkDateProgressive(); // contrôle progressif
	}

	// --- Contrôle progressif de la date ---
	function checkDateProgressive() {
		const today = new Date();
		const currentYear = today.getFullYear().toString(); // ex: '2025'
		const arr = dateField.split('');

		// Vérifier jour
		if (arr[0] !== ' ' && Number(arr[0]) > 3) return gameOver();
		if (arr[1] !== ' ' && arr[0] === '3' && Number(arr[1]) > 1) return gameOver();

		// Vérifier mois
		if (arr[3] !== ' ' && Number(arr[3]) > 1) return gameOver();
		if (arr[4] !== ' ' && arr[3] === '1' && Number(arr[4]) > 2) return gameOver();

		// Vérifier année
		for (let i = 6; i <= 9; i++) {
			if (arr[i] !== ' ' && Number(arr[i]) > Number(currentYear[i-6])) {
				return gameOver();
			}
		}
	}

	function gameOver() {
		resetDate();
		diceError = 'GAME\nOVER\n!'; // multi-lignes
	}

	// --- Reset complet ---
	function resetForm() {
		nom = '';
		prenom = '';
		resetPhone();
		resetDate();
		erreur = '';
		diceError = '';
		window.location.reload();
	}

	function resetDate() {
		dateField = '  /  /    ';
		diceStep = 0;
		diceValue = null;
		diceRolling = false;
		diceFace = 0;
		diceAnim = false;
	}

	// --- Soumission formulaire ---
	function handleSubmit() {
		erreur = '';
		diceError = '';
		const regexNomPrenom = /^[^0-9]+$/;

		if (!nom || !prenom || digits.includes(null) || dateField.includes(' ')) {
			erreur = 'Merci de correctement remplir tous les champs.';
			resetForm();
			return;
		}

		if (!regexNomPrenom.test(nom) || !regexNomPrenom.test(prenom)) {
			erreur = 'Merci de correctement remplir tous les champs.';
			resetForm();
			return;
		}

		const [jj, mm, aaaa] = dateField.split('/').map(Number);
		const dateTest = new Date(aaaa, mm - 1, jj);
		const today = new Date();
		if (isNaN(dateTest.getTime()) || dateTest > today) {
			erreur = 'Merci de correctement remplir tous les champs.';
			resetForm();
			return;
		}

		const numeroValide = digits.join('');
		const regexTel = /^(0[1-7])[0-9]{8}$/;
		if (!regexTel.test(numeroValide)) {
			erreur = 'Merci de correctement remplir tous les champs.';
			resetForm();
			return;
		}

		goto('/fin');
	}
	$effect(() => {
		$inspect(prenom);
	});
</script>

<style>
	html, body {
		margin: 0;
		padding: 0;
		width: 100%;
		height: 100%;
	}

	.wrapper {
		width: 100%;
		height: 100vh;
		overflow-x: scroll;
		overflow-y: hidden;
		scroll-behavior: smooth;
		position: relative;
		background: #000;
	}

	.wrapper::-webkit-scrollbar { display: none; }
	.wrapper { -ms-overflow-style: none; scrollbar-width: none; }

	form {
		max-width: 650px;
		margin: 2rem auto;
		padding: 2rem;
		background-color: #1a1a1a;
		color: #fff;
		font-family: 'Press Start 2P', cursive;
		border: 2px solid #00ff00;
		border-radius: 10px;
		position: relative;
	}

	h2 { text-align:center; margin-bottom:1.5rem; color:#00ff00; }
	label,button {
      color:white;
	}

	label { display:block; margin-top:1rem; margin-bottom:0.5rem; }
	input { width:100%; padding:0.5rem; font-size:0.9rem; border:2px solid #00ff00; border-radius:5px; background:#000; color:#fff;}
	input:focus { outline:none; border-color:#ff3d00; }

	.erreur { color:#ff3d00; margin-top:1rem; text-align:center; font-size:0.8rem; }

	.wheel-container, .dice-container { display:flex; justify-content:center; gap:0.5rem; margin:1rem 0;}
	.digit { width:3rem; height:3rem; background:#000; color:#00ff00; display:flex; justify-content:center; align-items:center; border:2px solid #00ff00; font-size:1.5rem; font-family: 'Press Start 2P', cursive;}

	.wheel-buttons, .dice-buttons { display:flex; justify-content:center; gap:1rem; margin-bottom:1rem;}
	.wheel-buttons button, .dice-buttons button { padding:0.5rem 1rem; font-size:0.8rem; background:#00ff00; color:#000; border:2px solid #fff; cursor:pointer; text-transform:uppercase; transition:0.2s;}
	.wheel-buttons button:hover, .dice-buttons button:hover { background:#fff; color:#00ff00; transform:scale(1.1); box-shadow:0 0 10px #00ff00;}
	.date-field { width:100%; padding:0.5rem; font-size:1rem; text-align:center; border:2px solid #00ff00; border-radius:5px; background:#000; color:#fff; margin-bottom:0.5rem;}

	/* Dé 3D */
	.dice-box {
		width: 80px;
		height: 80px;
		perspective: 600px;
		margin:auto;
	}
	.dice-cube {
		width: 100%;
		height: 100%;
		position: relative;
		transform-style: preserve-3d;
		transform: rotateX(0deg) rotateY(0deg);
		transition: transform 1s ease-out;
	}
	.face {
		position: absolute;
		width: 80px;
		height: 80px;
		background: #000;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 1.2rem;
		text-align: center;
		border: 2px solid #00ff00;
		padding: 0.2rem;
		box-sizing: border-box;
		word-break: break-word;
		white-space: pre-line; /* pour afficher les sauts de ligne */
		color: #00ff00;
	}
	.face-error { color: #ff3d00; } /* couleur rouge pour Game Over */
	.face1 { transform: rotateY(0deg) translateZ(40px); }
	.face2 { transform: rotateY(90deg) translateZ(40px); }
	.face3 { transform: rotateY(180deg) translateZ(40px); }
	.face4 { transform: rotateY(-90deg) translateZ(40px); }
	.face5 { transform: rotateX(90deg) translateZ(40px); }
	.face6 { transform: rotateX(-90deg) translateZ(40px); }

	.hidden-div {
		position: absolute;
		top: 2rem;
		left: calc(100% + 50px);
		width: 500px;
		height: 500px;
		background: #222;
		border: 2px solid #00ff00;
		color: #00ff00;
		font-family: 'Press Start 2P', cursive;
		box-sizing: content-box;
		margin-right: 1em;
	}
</style>

<div class="wrapper">
	<form on:submit|preventDefault={handleSubmit}>
		<h2>Formulaire</h2>

		<label>Nom</label>
		<input type="text" bind:value={nom} />

		<label>Prénom -></label>
		<input type="text" bind:value={prenom} disabled />

		<label>Date de naissance (jj/mm/aaaa)</label>
		<input class="date-field" type="text" bind:value={dateField} readonly />

		<div class="dice-container">
			<div class="dice-box">
				<div class="dice-cube" style="transform: rotateX({diceAnim ? Math.random()*720 : 0}deg) rotateY({diceAnim ? Math.random()*720 : 0}deg);">
					<div class="face face1" class:face-error={diceError}>{diceError || diceFace}</div>
					<div class="face face2" class:face-error={diceError}>{diceError || (diceFace+1)%10}</div>
					<div class="face face3" class:face-error={diceError}>{diceError || (diceFace+2)%10}</div>
					<div class="face face4" class:face-error={diceError}>{diceError || (diceFace+3)%10}</div>
					<div class="face face5" class:face-error={diceError}>{diceError || (diceFace+4)%10}</div>
					<div class="face face6" class:face-error={diceError}>{diceError || (diceFace+5)%10}</div>
				</div>
			</div>
		</div>

		<div class="dice-buttons">
			<button type="button" on:click={rollDice3D} disabled={diceStep >= 8}>Lancer Dé</button>
			<button type="button" on:click={validateDice} disabled={diceValue === null || diceError}>Valider</button>
		</div>

		<label>Téléphone</label>
		<div class="wheel-container">
			{#each digits as digit, i}
				<div class="digit">{digit !== null ? digit : (i === currentIndex ? currentDigit : '-')}</div>
			{/each}
		</div>
		<div class="wheel-buttons">
			<button type="button" on:click={startPhone} disabled={currentIndex>=digits.length}>Start</button>
			<button type="button" on:click={stopPhone} disabled={currentIndex>digits.length}>Stop</button>
			<button type="button" on:click={resetForm}>Reset</button>
		</div>

		<button type="submit">Envoyer</button>

		{#if erreur}<div class="erreur">{erreur}</div>{/if}
	</form>

	<div class="hidden-div">
		<Snake bind:textOutput={prenom}></Snake>
	</div>
</div>

