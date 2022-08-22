<script>
    export let textareaProperties;
    $: styledText = doHighlightText(textareaProperties.text);
    export let onKeyDownCallback;
    export let cursorInfo;

    let onKeyDownEvent = function(e) {
        let textarea = document.getElementById("codeTextarea");
        cursorInfo.startCursorPosition = textarea.selectionStart;
        cursorInfo.endCursorPosition = textarea.selectionEnd;
        onKeyDownCallback(e);
    };

    let doHighlightText = function (text) {
        let cursors = textareaProperties.cursors.sort((a, b) => b.startCursorPosition - a.startCursorPosition);
        let processed = [];
        console.log("+++ doHighlightText(cursors): ", cursors);
        let styled = text;
        for (let cursor of cursors) {
            if (cursor.owner === cursorInfo.cursorOwner)
                continue;

            let start = cursor.startCursorPosition;
            let end = cursor.endCursorPosition;
            let isProcessedAlready = processed.filter(p => p.start === start).length > 0;
            if (isProcessedAlready) {
                continue;
            }

            if (start === end && start > 0) {
                let before = styled.substring(0, start - 1);
                let middle = styled.substring(start - 1, end);
                let after = styled.substring(end, styled.length);
                styled = before + "<mark style='display: inline; border-radius: 3px; background-color: #8c97d5'>" + middle + "</mark>" + after;
            } else {
                let before = styled.substring(0, start);
                let middle = styled.substring(start, end);
                let after = styled.substring(end, styled.length);
                styled = before + "<mark style='display: inline; border-radius: 3px; background-color: #8c97d5'>" + middle + "</mark>" + after;
            }
            processed.push({start: start, end: end, length: length});
        }
        return styled;
    }

    let onClickEvent = function(e) {
        console.log("onClickEvent: ", e.key);
        let textarea = document.getElementById("codeTextarea");
        cursorInfo.startCursorPosition = textarea.selectionStart;
        cursorInfo.endCursorPosition = textarea.selectionEnd;
        onKeyDownCallback({key: null});
    }

    let handleScroll = function() {
        let textarea = document.getElementById("codeTextarea");
        document.getElementsByClassName("backdrop")[0].scrollTop = textarea.scrollTop;
    }
</script>

<main>
    <div class="container">
        <div class="backdrop">
            <div class="highlights">{@html styledText}</div>
        </div>
        <textarea id="codeTextarea"
                  spellcheck="false"
                  bind:value={textareaProperties.text}
                  on:keydown={onKeyDownEvent}
                  on:click={onClickEvent}
                  on:scroll={handleScroll}
        ></textarea>
    </div>
</main>

<style>
    *, *::before, *::after {
        box-sizing: border-box;
    }

    .container, .backdrop, textarea {
        width: 100%;
        height: 500px;
    }

    .highlights, textarea {
        padding: 10px;
        font: 20px/28px 'Consolas', sans-serif;
        letter-spacing: 1px;
    }

    .container {
        display: block;
        margin: 0 auto;
        transform: translateZ(0);
        -webkit-text-size-adjust: none;
    }

    .backdrop {
        position: absolute;
        z-index: 1;
        border: 2px solid #685972;
        background-color: #fff;
        overflow: auto;
        pointer-events: none;
        transition: transform 1s;
    }

    .highlights {
        white-space: pre-wrap;
        word-wrap: break-word;
        color: transparent;
    }

    textarea {
        display: block;
        position: absolute;
        z-index: 2;
        margin: 0;
        border: 2px solid #74637f;
        border-radius: 0;
        color: #444;
        background-color: transparent;
        overflow: auto;
        resize: none;
        transition: transform 1s;
    }

    textarea:focus {
        outline: none;
        box-shadow: 0 0 0 2px #c6aada;
    }
</style>