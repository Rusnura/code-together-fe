<script>
    let keys;
    let values;
    let entries;

    export let textareaProperties;
    $: {
        keys = Object.keys(textareaProperties.cursors);
        console.log("DebugComponent: cursors keys: ", keys);
        values = Object.values(textareaProperties.cursors);
        console.log("DebugComponent: cursors values: ", values);
        entries = Object.entries(textareaProperties.cursors);
        console.log("DebugComponent: cursors entries: ", entries);
    }
    export let cursorInfo;
</script>

<main>
    <h2>Отладочная информация:</h2>
    <h3>Информация о моём курсоре:</h3>
    Позиция начала:<br>
    <input type="text" readonly bind:value={cursorInfo.startCursorPosition} />
    <br>
    Конечная позиция:<br>
    <input type="text" readonly bind:value={cursorInfo.endCursorPosition} />
    <hr>
    {#each entries as [key, value]}
        Пользователь: {key}<br>
        Позиция начала:<br>
        <input type="text" readonly bind:value={value.startCursorPosition} />
        <br>
        Конечная позиция:<br>
        <input type="text" readonly bind:value={value.endCursorPosition} />
        <br>
    {/each}
</main>

<style>

</style>