<script>
  import API_HOST from "../api_config.js";
  import {
    getToken,
    deleteToken,
    setToken,
    getCsrfToken,
    deleteCsrfToken,
    setCsrfToken
  } from "../utils.js";
  import {
    fetchUserData,
    fetchAuthToken,
    fetchCreateUser,
    fetchCsrfToken,
    validateInputs
  } from "../../../shared/auth.js";
  export let isLoggedIn;
  let id, username, email;
  let wantsToSignUp = false;
  let usernameInput = "";
  let passwordInput = "";
  let isMessageError = false;
  let message = "";
  if (getToken()) {
    // if we have a token saved to local storage,
    // then log them in automatically
    fetchUserData_().then(fetchCsrfToken_());
    // Also get a new CSRF token always
  }
  function setUserData(json) {
    id = json.id;
    username = json.username;
    email = json.email;
    isLoggedIn = true;
    isMessageError = false;
    message = "Logged in as " + username;
  }
  function clearUserData() {
    id = null;
    username = null;
    email = null;
    isLoggedIn = false;
    isMessageError = false;
    message = "Logged out";
  }
  async function fetchUserData_() {
    let token = getToken();
    if (token) {
      try {
        fetchUserData(token)
          .then(res => res.json())
          .then(json => {
            if (json.username) {
              setUserData(json);
            } else {
              deleteToken();
            }
          })
          .catch(err => {
            console.log(err);
          });
      } catch (error) {
        console.log("threw error when fetching username: " + error);
        deleteToken();
      }
    } else {
      clearUserData();
      console.log("cannot get username because we have no token");
    }
  }
  function logOut() {
    deleteToken();
    deleteCsrfToken();
    clearUserData();
  }

  async function fetchAuthToken_() {
    if (validateInputs(usernameInput, passwordInput)) {
      fetchAuthToken(usernameInput, passwordInput)
        .then(res => res.json())
        .then(json => {
          if (json.token && json.user) {
            localStorage.setItem("token", json.token);
            setUserData(json.user);
          } else {
            isMessageError = true;
            message = "Error logging in";
          }
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
  function fetchCsrfToken_() {
    let token = getToken();
    if (token && !getCsrfToken()) {
      fetchCsrfToken(token)
        .then(res => res.json())
        .then(json => {
          if (json.csrf_token) {
            setCsrfToken(json.csrf_token);
          } else {
            isMessageError = true;
            message = "Error getting CSRF token";
          }
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
  function logIn() {
    fetchAuthToken_().then(fetchCsrfToken_());
  }
  function signUp() {
    if (validateInputs(usernameInput, passwordInput)) {
      fetchCreateUser(usernameInput, passwordInput)
        .then(res => res.json())
        .then(json => {
          if (json.token) {
            localStorage.setItem("token", json.token);
            setUserData(json);
          } else {
            isMessageError = true;
            message = json.username;
          }
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
</script>

<style>
  .container {
    width: 2in;
    border-style: solid;
    margin: 10px;
    padding: 0 0 5px 0;
  }
  .message-container {
    display: flex;
    justify-content: center;
    text-align: center;
    padding: 10px 0 5px 0;
  }
  .message-success {
    color: #39ff14;
  }
  .message-error {
    color: #ffbcd9;
  }
  button {
    height: 40px;
    width: 80px;
    margin: 5px auto 5px auto;
    background: #999999;
    color: white;
    cursor: pointer;
  }
  .button:active {
    transform: translateY(3px);
  }
  .button-submit {
    display: flex;
  }
  .button-submit:hover {
    background-color: #39ff14;
    color: #222222;
  }
  .button-log-out {
    display: flex;
  }
  .button-container {
    display: block;
    text-align: center;
  }
  .button-log-in {
    display: inline-flex;
  }
  .button-active {
    background-color: cyan;
    color: black;
  }
  .button-text {
    margin: auto;
  }
  .inputs {
    display: block;
    text-align: center;
    margin: 0 auto;
  }
  p {
    margin: 10px 0 0 0;
  }
</style>

<div>
  <div class="container">
    {#if message}
      <div class="message-container">
        <div class={isMessageError ? 'message-error' : 'message-success'}>
          {message}
        </div>
      </div>
    {/if}

    {#if isLoggedIn}
      <div class="button-container">
        <button on:click={logOut} class="button button-log-out">
          <span class="button-text">Log out</span>
        </button>
      </div>
    {:else}
      <div class="button-container">
        <button
          on:click={() => (wantsToSignUp = false)}
          class="button button-log-in {!wantsToSignUp ? 'button-active' : ''}">
          <span class="button-text">Log in</span>
        </button>
        <button
          on:click={() => (wantsToSignUp = true)}
          class="button button-log-in {wantsToSignUp ? 'button-active' : ''}">
          <span class="button-text">Sign up</span>
        </button>
      </div>
      <div class="inputs">
        <p>
          <input
            type="text"
            name="username"
            bind:value={usernameInput}
            placeholder="username" />
        </p>
        <p>
          <input
            type="password"
            name="password"
            bind:value={passwordInput}
            placeholder="password" />
        </p>
      </div>
      <div class="button-container">
        {#if wantsToSignUp}
          <button class="button button-submit" on:click={signUp}>
            <span class="button-text">Submit</span>
          </button>
        {:else}
          <button class="button button-submit" on:click={logIn}>
            <span class="button-text">Submit</span>
          </button>
        {/if}
      </div>
    {/if}
  </div>
</div>
