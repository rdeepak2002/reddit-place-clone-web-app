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
    let user: { id: string, name: string, imageUrl: string, email: string, id_token: string } = undefined;
    let loadingAuthState = true;
    let expirationTime = undefined;
    let currentTime = Math.floor(Date.now() / 1000);
    let madeFirstRequest = false;

    // update current time global variable every 500 milliseconds
    setInterval(() => {
        currentTime = Math.floor(Date.now() / 1000);
    }, 500)

    // check if a variable is an integer
    const isInt = (value) => {
        return !isNaN(value) && (function (x) {
            return (x | 0) === x;
        })(parseFloat(value))
    }

    // handle form submission
    const submitPixel = (e) => {
        e.preventDefault();
        drawPixel(xInput, yInput, redInput, greenInput, blueInput);
    }

    // send pixel to server
    const drawPixel = (xInput, yInput, redInput, greenInput, blueInput) => {
        console.log("Sending draw pixel request...");

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
            data: data
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

                madeFirstRequest = true;
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

        // initiate an invalid draw to display timer
        drawPixel(-1, -1, 0, 0, 0);
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
        gapi.load('auth2', function () {
            // @ts-ignore
            gapi.auth2.init({
                client_id: "512758578665-7mrm2s66ub7rd983qu3sv51ohgh79pqh.apps.googleusercontent.com"
            }).then(() => {
                loadingAuthState = false;
            })
        });

        // @ts-ignore
        gapi.signin2.render('my-signin2', {
            'width': 300,
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
    <div class="container">
        <img id="output_image" alt="Reddit place canvas." src={imageUrl}/>

        {#if expirationTime !== undefined && isInt(expirationTime) && (expirationTime - currentTime) >= 0}
            <p>{expirationTime - currentTime} seconds until you can place a new pixel</p>
        {:else if (user && !loadingAuthState && madeFirstRequest)}
            <form on:submit|preventDefault={submitPixel} class="form">
                <div>
                    <label for="x-input">X</label>
                    <input type="number" id="x-input" bind:value={xInput}/>
                </div>

                <div>
                    <label for="y-input">Y</label>
                    <input type="number" id="y-input" bind:value={yInput}/>
                </div>

                <div>
                    <label for="red-input">Red</label>
                    <input type="number" id="red-input" bind:value={redInput}/>
                </div>

                <div>
                    <label for="green-input">Green</label>
                    <input type="number" id="green-input" bind:value={greenInput}/>
                </div>

                <div>
                    <label for="blue-input">Blue</label>
                    <input type="number" id="blue-input" bind:value={blueInput}/>
                </div>

                <button type="submit">Set Pixel</button>
            </form>
        {/if}

        <div class="auth-container">
            {#if (user && !loadingAuthState && madeFirstRequest)}
                <a href="javascript:void(0);" on:click={signOut}>Sign out</a>
            {/if}

            <div id="my-signin2"
                 style={`visibility: ${(user === undefined && loadingAuthState === false) ? 'visible' : 'hidden'}`}></div>
        </div>
    </div>
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

    .container {
        height: 100vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        align-content: center;
    }

    .auth-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        align-content: center;
    }

    .form {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        align-content: center;
    }
</style>