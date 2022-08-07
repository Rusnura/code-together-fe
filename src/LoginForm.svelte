<script>
    export let sessionInfo;
    export let successfulLoginCallback;
    export let failureLoginCallback;

    let doLogin = function () {
        if (!sessionInfo.username) {
            failureLoginCallback();
            return;
        }

        successfulLoginCallback();
        sessionInfo.isUserLoggedIn = true;
    }
</script>
<main>
    <div>
        Пожалуйста, авторизуйтесь:
    </div>
    <div>
        <input type="text" bind:value="{sessionInfo.username}" placeholder="Ваше имя" />
        <button on:click={doLogin}>Войти как {sessionInfo.username}</button>
    </div>
</main>