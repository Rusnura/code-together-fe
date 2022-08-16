<script>
    export let textareaProperties;
    export let cursorInfo;
    export let sessionInfo;

    let putTextareaPropertiesToConsole = function () {
        console.log("DebugComponent: cursors entries: ", textareaProperties.cursors);
    }
</script>

<main>
    <h3>Отладочная информация:</h3>
    Курсор пользователя: {sessionInfo.username}: <i>{cursorInfo.startCursorPosition}/{cursorInfo.endCursorPosition}</i><br>
    {#each textareaProperties.cursors as cursor, i}
        Курсор пользователя: {cursor.owner} ({i}): <i>{cursor.startCursorPosition}/{cursor.endCursorPosition}</i><br>
    {/each}
    <br>
    <button on:click="{putTextareaPropertiesToConsole}">Put textarea properties to console</button>
</main>

<style>

</style>