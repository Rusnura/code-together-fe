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
			endCursorPosition: message.endCursorPosition
		});
		console.log("!!! CONNECTION EVENT !!! Pushed new cursor to textareaProperties: ", textareaProperties);
	};

	let onTypeEventCallback = function (message) {
		let textarea = document.getElementById("codeTextarea");
		console.log("!!! TYPE EVENT !!!", message);
		let start = message.startCursorPosition;
		let end = message.endCursorPosition;
		let before = textarea.value.substring(0, start);
		let after = textarea.value.substring(end, textarea.value.length);

		textarea.value = before + message.key + after;
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
				onKeyUpCallback="{textareaEditCallback}">
		</CodeTextarea>
	{/if}
</main>

<style>

</style>