<script>
    let cursors;

    export let textareaProperties;
    $: {
        cursors = Object.entries(textareaProperties.cursors);
        console.log("DebugComponent: cursors entries: ", cursors);
    }
    export let cursorInfo;
    export let sessionInfo;
</script>

<main>
    <h3>Отладочная информация:</h3>
    Курсор пользователя: {sessionInfo.username}: <i>{cursorInfo.startCursorPosition}/{cursorInfo.endCursorPosition}</i><br>
    {#each cursors as [user, cursor]}
        Курсор пользователя: {user}: <i>{cursor.startCursorPosition}/{cursor.endCursorPosition}</i><br>
    {/each}
</main>

<style>

</style>