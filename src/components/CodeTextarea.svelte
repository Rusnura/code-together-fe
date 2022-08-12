<script>
    export let textareaProperties;
    export let onKeyDownCallback;
    export let cursorInfo;

    let onKeyDownEvent = function (e) {
        console.log("onKeyDownEvent: ", e, "Key={", e.key, "}");
        onKeyDownCallback(e);
    };

    let onKeyUpEvent = function (e) {
        console.log("onKeyUpEvent: ", e.key);
        let textarea = document.getElementById("codeTextarea");
        cursorInfo.startCursorPosition = textarea.selectionStart;
        cursorInfo.endCursorPosition = textarea.selectionEnd;
        onKeyDownCallback({key: null});
    }
</script>

<main>
    <textarea id="codeTextarea"
              cols={textareaProperties.cols}
              rows={textareaProperties.rows}
              bind:value={textareaProperties.text}
              on:keydown={onKeyDownEvent}
              on:keyup={onKeyUpEvent}
              on:click={onKeyUpEvent}></textarea>

    <div>
        My start position:<br>
        <input type="text" readonly bind:value={cursorInfo.startCursorPosition} />
        <br>
        My end position:<br>
        <input type="text" readonly bind:value={cursorInfo.endCursorPosition} />
    </div>
</main>

<style>

</style>