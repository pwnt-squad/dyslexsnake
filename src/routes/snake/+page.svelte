<script>
	import { onDestroy, onMount } from 'svelte';

	/**
	 *
	 * @param {number} min
	 * @param {number} max
	 */
	function randomInt(min, max) {
		const minCeiled = Math.ceil(min);
		const maxFloored = Math.floor(max);
		return Math.floor(Math.random() * (maxFloored - minCeiled) + minCeiled);
	}

	const Direction = {
		Up: 'Up',
		Down: 'Down',
		Left: 'Left',
		Right: 'Right'
	};

	/** @type {HTMLCanvasElement} */
	let canvas;

	let snake_head_size = $state(20);

	// let snake_head_x = $state(10);
	// let snake_head_y = $state(10);
	// let snake_head_dir = $state(Direction.Left);

	let current_direction = $state(Direction.Left);
	let snake_speed = $state(1);
	// let snake_speed_y = $state(0);

	let window_inner_width = $state(0);
	let window_inner_height = $state(0);

	let canvas_width = $derived(Math.ceil(window_inner_width / 2 / snake_head_size) * snake_head_size);
	let canvas_height = $derived(Math.ceil(window_inner_height / 2 / snake_head_size) * snake_head_size);

	let max_x = $derived(canvas_width / snake_head_size);
	let max_y = $derived(canvas_height / snake_head_size);

	let running = $state(false);
	let dead = $state(false);
	let won = $state(false);

	/**
	 * @typedef SnakeFruit
	 * @property {number} x
	 * @property {number} y
	 * @property {string} letter
	 */
	/** @type {SnakeFruit[]}*/
	let letter_fruits = $state([]);
	/** @type {SnakeFruit}*/
	let check_fruit = $state();

	let current_word = $state("");


	/**
	 * @typedef SnakePos
	 * @property {number} x
	 * @property {number} y
	 * @property {string} dir
	 */


	let body_parts = $state({
		/** @type {SnakePos[]}*/
		parts: [{ x: 10, y: 10, dir: Direction.Right }],
		parts_to_add: 0
	});

	const fruit_radius = 7;

	// render function
	$effect(() => {
		const ctx = canvas.getContext('2d');
		ctx.clearRect(0, 0, canvas_width, canvas_height);

		// render fruits
		for (let fruit of letter_fruits) {
			ctx.fillStyle = 'lightblue';
			ctx.beginPath();
			ctx.arc(fruit.x * snake_head_size+ snake_head_size/2, fruit.y * snake_head_size+ snake_head_size/2, fruit_radius, 0, 2*Math.PI);
			ctx.fill();
			ctx.font = '12px monospace';
			ctx.fillStyle = 'black';
			ctx.fillText(fruit.letter.toUpperCase(), fruit.x * snake_head_size+fruit_radius, fruit.y * snake_head_size + 2*(fruit_radius));
		}

		if (check_fruit) {
			ctx.font = '12px monospace';
			ctx.fillStyle = 'black';
			ctx.fillText(check_fruit.letter.toUpperCase(), check_fruit.x * snake_head_size+2, check_fruit.y * snake_head_size + 2 * (fruit_radius));
		}
		// render snake
		ctx.fillStyle = 'red';
		for (let i = 0; i < body_parts.parts.length; i += 1) {
			ctx.fillRect(body_parts.parts[i].x * snake_head_size, body_parts.parts[i].y * snake_head_size, snake_head_size, snake_head_size);
		}

		if (dead) {
			ctx.font = '30px sans-serif';
			ctx.fillStyle = 'black';
			ctx.fillText('You died', canvas_width / 2 - 60, canvas_height / 2 - 15);
		}

		if (won) {
			ctx.font = '30px sans-serif';
			ctx.fillStyle = 'black';
			ctx.fillText('You won', canvas_width / 2 - 60, canvas_height / 2 - 15);
		}
	});

	const letters = ['a', "b" ,"c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"];

	function generate_fruits() {
		letter_fruits = [];
		for (let letter of letters) {
			while (true) {
				let x = randomInt(0, max_x);
				let y = randomInt(0, max_y);
				let valid = true;
				for (let fruit of letter_fruits) {
					if (fruit.x === x && fruit.y === y) {
						valid = false;
						break;
					}
				}
				if (valid) {
					letter_fruits.push({
						x: x,
						y: y,
						letter: letter
					});
					break;
				}
			}
		}

		// generate check fruit
		while (true) {
			let x = randomInt(0, max_x);
			let y = randomInt(0, max_y);
			let valid = true;
			for (let fruit of letter_fruits) {
				if (fruit.x === x && fruit.y === y) {
					valid = false;
					break;
				}
			}
			if (valid) {
				check_fruit = {
					x: x,
					y: y,
					letter: "✅"
				};
				break;
			}
		}
	}

	function game_tick() {
		let next_dir = current_direction;
		for (let i = 0; i < body_parts.parts.length; i += 1) {
			let old_dir = body_parts.parts[i].dir;
			switch (old_dir) {
				case Direction.Left: {
					body_parts.parts[i].x -= snake_speed;
					break;
				}
				case Direction.Right: {
					body_parts.parts[i].x += snake_speed;
					break;
				}
				case Direction.Up: {
					body_parts.parts[i].y -= snake_speed;
					break;
				}
				case Direction.Down: {
					body_parts.parts[i].y += snake_speed;
					break;
				}
			}
			body_parts.parts[i].dir = next_dir;
			next_dir = old_dir;
		}
		// add a new part
		if (body_parts.parts_to_add > 0) {
			let last_part = body_parts.parts[body_parts.parts.length - 1];
			let x = last_part.x;
			let y = last_part.y;
			switch (next_dir) {
				case Direction.Left: {
					x += 1;
					break;
				}
				case Direction.Right: {
					x -= 1;
					break;
				}
				case Direction.Up: {
					y += 1;
					break;
				}
				case Direction.Down: {
					y -= 1;
					break;
				}
			}
			body_parts.parts.push({
				x: x,
				y: y,
				dir: next_dir
			});
			body_parts.parts_to_add -= 1;
		}
		// check snake in bound
		let head = body_parts.parts[0];
		if (head.x < 0 || head.x >= max_x || head.y < 0 || head.y >= max_y) {
			running = false;
			dead = true;
			current_word = "";
			window.clearInterval(interval);
		}
		// check snake head not on body
		for (let i = 1; i < body_parts.parts.length; i += 1) {
			let part = body_parts.parts[i];
			if (part.x === head.x && part.y === head.y) {
				running = false;
				dead = true;
				current_word = "";
				window.clearInterval(interval);
			}
		}
		// check snake head on letter
		for (let fruit of letter_fruits) {
			if (fruit.x === head.x && fruit.y === head.y) {
				body_parts.parts_to_add += 1;
				current_word += fruit.letter;
				// todo: add the letter to the form
				generate_fruits();
				break;
			}
		}
		// check snake head on checkmark
		if (check_fruit.x === head.x && check_fruit.y === head.y) {
			// todo: validate the form
			running = false;
			won = true;
			window.clearInterval(interval);
		}
	}

	$effect(() => {
		$inspect(body_parts);
	});

	function reset_snake() {
		body_parts = {
			parts: [{ x: 10, y: 10, dir: Direction.Right }],
			parts_to_add: 0
		};
		// window.clearInterval(interval);
		// interval = window.setInterval(game_tick, 200);
		dead = false;
		running = true;
		won = false;
		current_word = "";
		generate_fruits();
	}

	/** @param {KeyboardEvent} event */
	function handle_keydown(event) {
		if (won) {
			return;
		}
		if (!running) {
			running = true;
		}
		switch (event.key) {
			case 'Escape': {
				running = false;
				break;
			}
			case 'ArrowRight': {
				if (current_direction !== Direction.Up) {
					current_direction = Direction.Right;
				}
				break;
			}
			case 'ArrowLeft': {
				if (current_direction !== Direction.Right) {
					current_direction = Direction.Left;
				}
				break;
			}
			case 'ArrowDown': {
				if (current_direction !== Direction.Up) {
					current_direction = Direction.Down;
				}
				break;
			}
			case 'ArrowUp': {
				if (current_direction !== Direction.Down) {
					current_direction = Direction.Up;
				}
				break;
			}
			case ' ': {
				body_parts.parts_to_add += 1;
				break;
			}
		}
		if (dead) {
			reset_snake();
			return;
		}
		console.log(event.key);
	}

	/**
	 * @type {number}
	 */
	let interval;

	$effect(() => {
		if (running) {
			interval = window.setInterval(game_tick, 200);
		} else {
			window.clearInterval(interval);
		}
	});

	onMount(() => {
		generate_fruits();
	});
	// onDestroy(() => {
	// 	window.clearInterval(interval);
	// })
</script>

<svelte:window bind:innerWidth={window_inner_width} bind:innerHeight={window_inner_height} onkeydown={handle_keydown} />

<style>
    h1 {
        font-size: large;
    }

    canvas {
        border: 1px solid black;
        margin: 3em auto auto;
    }
</style>

<canvas bind:this={canvas} width={canvas_width} height={canvas_height}></canvas>

<h1>Current word: {current_word}</h1>
