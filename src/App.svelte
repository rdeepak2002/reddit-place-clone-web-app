<script lang="ts">
	import axios from "axios";

	console.log("Init App.svelte");

	let isProd = location.protocol === 'https:';
	let serverUrl = isProd ? "" : "http://localhost:3000";
	let imageUrl = `${serverUrl}/static/image.png`;
	let xInput = 0;
	let yInput = 0;
	let redInput = 0;
	let greenInput = 0;
	let blueInput = 0;
	let user: {id: string, name: string, imageUrl: string, email: string, id_token: string} = undefined;
	let loadingAuthState = true;
	let expirationTime = undefined;
	let currentTime = Math.floor(Date.now() / 1000);

	setInterval(() => {
		currentTime = Math.floor(Date.now() / 1000);
	}, 500)

	// check if a variable is an integer
	const isInt = (value) => {
		return !isNaN(value) && (function(x) { return (x | 0) === x; })(parseFloat(value))
	}

	// send pixel to server
	const submitPixel = (e) => {
		e.preventDefault();

		if (user === undefined || user.id_token === undefined) {
			console.error("Invalid user");
			return;
		}

		if (xInput === undefined || yInput === undefined || redInput === undefined
				|| greenInput === undefined || blueInput === undefined) {
			console.error("Undefined input");
			return;
		}

		if (!isInt(xInput) || !isInt(yInput) || !isInt(redInput)
				|| !isInt(greenInput) || !isInt(blueInput)) {
			console.error("Non-integer input");
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
				'Authorization': `Bearer ${user.id_token.toString()}`,
				'Content-Type': 'application/json'
			},
			data : data
		};

		axios(config)
				.then(function (response) {
					if (response?.data?.status === "success") {
						console.log(response?.data);
						location.reload();
					} else if (response?.data?.status === "ratelimit") {
						console.log(response?.data);
						expirationTime = response?.data?.message;

						if (expirationTime !== undefined) {
							expirationTime = parseInt(expirationTime)
						}
					} else {
						console.log(response?.data);
					}
				})
				.catch(function (error) {
					console.log(error);
				});
	}

	// when user signs in
	const onSuccess = (googleUser) => {
		console.log('Logged in as: ' + googleUser.getBasicProfile().getName());
		const profile = googleUser.getBasicProfile();

		user = {
			id: profile.getId(),
			name: profile.getName(),
			imageUrl: profile.getImageUrl(),
			email: profile.getEmail(),
			id_token: googleUser.getAuthResponse().id_token
		}
	}

	// when unable to sign in user
	const onFailure = (error) => {
		console.log("Failure for login", error);
		user = undefined;
	}

	// when auth has loaded
	(window as any).onLoadCallback = () => {
		console.log("Auth state loaded");

		// @ts-ignore
		gapi.load('auth2', function() {
			// @ts-ignore
			gapi.auth2.init({
				client_id: "512758578665-7mrm2s66ub7rd983qu3sv51ohgh79pqh.apps.googleusercontent.com"
			}).then(() => {
				loadingAuthState = false;
			})
		});

		// @ts-ignore
		gapi.signin2.render('my-signin2', {
			'longtitle': true,
			'theme': 'light',
			'onsuccess': onSuccess,
			'onfailure': onFailure
		});
	}

	// function to sign out
	const signOut = () => {
		// @ts-ignore
		const auth2 = gapi.auth2.getAuthInstance();
		auth2.signOut().then(() => {
			console.log("Signed out");
			user = undefined;
			location.reload();
		});
	}
</script>

<svelte:head>
	<script src="https://apis.google.com/js/platform.js?onload=onLoadCallback" async defer></script>
</svelte:head>

<main>
	<img id="output_image" alt="Reddit place canvas." src={imageUrl}/>

	{#if user !== undefined && loadingAuthState === false}
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

		<a href="javascript:void(0);" on:click={signOut}>Sign out</a>
	{/if}

	{#if expirationTime !== undefined && isInt(expirationTime) && expirationTime - currentTime >= 0}
		<p>{expirationTime - currentTime} seconds until you can place a new pixel</p>
	{/if}

	<div id="my-signin2" style={`visibility: ${(user === undefined && loadingAuthState === false) ? 'visible' : 'hidden'}`}></div>
</main>

<style>
	img {
		border: dotted black;
		width: 300px;
		height: 300px;
		image-rendering: pixelated;
		image-rendering: -moz-crisp-edges;
		image-rendering: crisp-edges;
	}
	
	a:hover {
		cursor: pointer;
	}
</style>