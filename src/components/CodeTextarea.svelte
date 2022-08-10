<script>
    export let textareaProperties;
    export let onKeyDownCallback;

    let onKeyDownEvent = function (e) {
        let textarea = document.getElementById("codeTextarea");
        let startCursorPosition = textarea.selectionStart;
        let endCursorPosition = textarea.selectionEnd;
        console.log("Start: ", startCursorPosition, "End: ", endCursorPosition, "Key: ", e.key);
        let cursorInfo = {
            startCursorPosition: startCursorPosition,
            endCursorPosition: endCursorPosition,
            key: e.key
        };
        onKeyDownCallback(cursorInfo);
    };

    let onTypeEventCallback = function (message) {

    };
</script>

<main>
    <textarea id="codeTextarea"
              cols={textareaProperties.cols}
              rows={textareaProperties.rows}
              bind:value={textareaProperties.text}
              on:keyup={onKeyDownEvent}

              on:click={onKeyDownEvent}></textarea>
</main>

<style>

</style>