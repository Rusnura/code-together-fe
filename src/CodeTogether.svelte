<script>
	import Header from "./Header.svelte";
	import LoginForm from "./LoginForm.svelte";
	// import SockJS from "sockjs-client"; // why not works?
	import Stomp from "webstomp-client";
	import CodeTextarea from "./components/CodeTextarea.svelte";

	let sessionInfo = {
		isUserLoggedIn: false,
		roomId: 'room1',
		username: '',
		stompClient: null
	}

	let textareaProperties = {
		selectionStart: 0,
		selectionEnd: 0,
		cols: 150,
		rows: 20,
		text: '',
		cursors: []
	}

	let successfulLoginCallback = () => {
		const socket = new WebSocket("ws://192.168.88.14:8081/code-together");
		sessionInfo.stompClient = Stomp.over(socket);
		sessionInfo.stompClient.connect(
		{},
		frame => {
			console.log("Connection frame: ", frame);

			sessionInfo.stompClient.subscribe("/code-together/room/room1", (message) => {
				let jsonMessage = JSON.parse(message.body);
				if (jsonMessage.username === sessionInfo.username) {
					console.log("Ignore. Message from myself!");
					return;
				}

				switch (jsonMessage.type) {
					case "CONNECT":
						onConnectEventCallback(jsonMessage);
					break;

					case "TYPE":
						onTypeEventCallback(jsonMessage);
					break;

					default:
						console.log("No event handler found!");
				}
			}, {
				"username": sessionInfo.username
			});
		},
		error => {
			console.log("Connection error: ", error);
		});

		console.log("successfulLoginCallback: sessionInfo=", sessionInfo);
	}

	let failureLoginCallback = function() {
		console.log("failureLoginCallback: sessionInfo=", sessionInfo);
	}

	let textareaEditCallback = function (cursorInfo) {
		if (sessionInfo.stompClient && sessionInfo.stompClient.connected) {
			sessionInfo.stompClient.send("/code-together/room/{roomId}", JSON.stringify({
				roomId: sessionInfo.roomId,
				username: sessionInfo.username,
				startCursorPosition: cursorInfo.startCursorPosition,
				endCursorPosition: cursorInfo.endCursorPosition,
				key: cursorInfo.key
			}), {});
		}
	}

	let onConnectEventCallback = function (message) {
		console.log("!!! CONNECTION EVENT !!!", message);
		textareaProperties.cursors.push({
			roomId: message.roomId,
			username: message.username,
			startCursorPosition: message.startCursorPosition,
			endCursorPosition: message.endCursorPosition,
		});
		console.log("!!! CONNECTION EVENT !!! Pushed new cursor to textareaProperties: ", textareaProperties);
	};

	let onTypeEventCallback = function (message) {
		console.log("!!! TYPE EVENT !!!", message);
		for (let i = 0; i < textareaProperties.cursors.length; i++) {
			let cursorOwnerInfo = textareaProperties.cursors[i];
			if (cursorOwnerInfo.username === message.username) {
				console.log("The owner found: ", cursorOwnerInfo);
				handleKey(cursorOwnerInfo, message);
			}
		}
	}

	let handleKey = function (cursorOwnerInfo, message) {
		let textarea = document.getElementById("codeTextarea");
		let navigationStuff = ["ArrowUp", "ArrowDown", "ArrowRight", "ArrowLeft", "Home", "End", null];
		if (navigationStuff.includes(message.key)) {
			console.log("It's navigation stuff. Processing!!!")
			cursorOwnerInfo.startCursorPosition = message.startCursorPosition;
			cursorOwnerInfo.endCursorPosition = message.endCursorPosition;

			console.log("new cursorOwnerInfo=", cursorOwnerInfo);
		} else {
			console.log("It's NOT navigation stuff. Processing as text!!!")
			let start = cursorOwnerInfo.startCursorPosition;
			let end = cursorOwnerInfo.endCursorPosition;
			let before = textarea.value.substring(0, start);
			let after = textarea.value.substring(end, textarea.value.length);


			// WHAT I NEED DO HERE???
			cursorOwnerInfo.startCursorPosition = message.startCursorPosition;
			cursorOwnerInfo.endCursorPosition = message.endCursorPosition;
			textarea.value = before + message.key + after;
		}
	}
</script>

<main>
	<Header />

	{#if !sessionInfo.isUserLoggedIn}
		<LoginForm
				bind:sessionInfo="{sessionInfo}"
				successfulLoginCallback="{successfulLoginCallback}"
				failureLoginCallback="{failureLoginCallback}"
		/>
	{:else}
		Добро пожаловать, {sessionInfo.username}!<br>
		<CodeTextarea
				bind:textareaProperties={textareaProperties}
				onKeyDownCallback="{textareaEditCallback}">
		</CodeTextarea>
	{/if}
</main>

<style>

</style>