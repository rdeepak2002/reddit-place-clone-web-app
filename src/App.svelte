<script lang="ts">
	import axios from "axios";

	let isProd = location.protocol === 'https:';

	let serverUrl = isProd ? "" : "http://localhost:3000";

	let imageUrl = `${serverUrl}/static/image.png`;

	let xInput = 0;
	let yInput = 0;
	let redInput = 0;
	let greenInput = 0;
	let blueInput = 0;

	function isInt(value) {
		return !isNaN(value) && (function(x) { return (x | 0) === x; })(parseFloat(value))
	}

	const submitPixel = (e) => {
		e.preventDefault();

		if (xInput === undefined || yInput === undefined || redInput === undefined
				|| greenInput === undefined || blueInput === undefined) {
			console.error("Invalid input");
			return;
		}

		if (!isInt(xInput) || !isInt(yInput) || !isInt(redInput)
				|| !isInt(greenInput) || !isInt(blueInput)) {
			console.error("Invalid input");
			return;
		}

		let data = JSON.stringify({
			"x": xInput,
			"y": yInput,
			"red": redInput,
			"green": greenInput,
			"blue": blueInput
		});

		let config = {
			method: 'post',
			url: `${serverUrl}/set_pixel`,
			headers: {
				'Content-Type': 'application/json'
			},
			data : data
		};

		axios(config)
				.then(function (response) {
					console.log(JSON.stringify(response.data));
					location.reload();
				})
				.catch(function (error) {
					console.log(error);
				});
	}
</script>

<main>
	<img id="output_image" style="border: dotted black; width: 300px; height: 300px;" alt="image"
		 src={imageUrl}/>
	<form on:submit|preventDefault={submitPixel}>
		<label>
			X
			<input type="number" id="x-input" bind:value={xInput}/>
		</label>
		<label>
			Y
			<input type="number" id="y-input" bind:value={yInput}/>
		</label>
		<label>
			Red
			<input type="number" id="red-input" bind:value={redInput}/>
		</label>
		<label>
			Green
			<input type="number" id="green-input" bind:value={greenInput}/>
		</label>
		<label>
			Blue
			<input type="number" id="blue-input" bind:value={blueInput}/>
		</label>
		<button type="submit" onsubmit="submitPixel(e)">Set Pixel</button>
	</form>
</main>

<style>
</style>