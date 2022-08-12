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
		cols: 150,
		rows: 20,
		text: '',
		cursors: {}
	}

	let cursorInfo = {
		startCursorPosition: 0,
		endCursorPosition: 0,
	}

	let successfulLoginCallback = () => {
		const socket = new WebSocket("ws://192.168.88.14:8081/code-together");
		sessionInfo.stompClient = Stomp.over(socket);
		sessionInfo.stompClient.connect({}, frame => {
			console.log("Connection frame: ", frame);



			sessionInfo.stompClient.subscribe("/code-together/room/room1", (raw) => {
				let message = JSON.parse(raw.body);
				console.log("Get message. ", message);
				if (message.type === "connect") {
					handleNewUserConnected(message);
				} else if (message.type === "insert") {
					handleInsert(message);
				} else if (message.type === "navigation") {
					handleNavigation(message);
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

	let handleNavigation = function (message) {
		if (message.username === sessionInfo.username)
			return;

		textareaProperties.cursors[message.username].startCursorPosition = message.startCursorPosition;
		textareaProperties.cursors[message.username].endCursorPosition = message.endCursorPosition;
	}

	let handleInsert = function (message) {
		if (message.username === sessionInfo.username)
			return;

		let start = textareaProperties.cursors[message.username].startCursorPosition;
		let end = textareaProperties.cursors[message.username].endCursorPosition;
		let before = textareaProperties.text.substring(0, start);
		let after = textareaProperties.text.substring(end);
		textareaProperties.text = before + message.key + after;
	}

	let handleNewUserConnected = function (message) {
		if (message.username === sessionInfo.username)
			return;

		textareaProperties.cursors[message.username] = {
			startCursorPosition: 0,
			endCursorPosition: 0
		};
	};

	let failureLoginCallback = function() {
		console.log("failureLoginCallback: sessionInfo=", sessionInfo);
	}

	let textareaEditCallback = function (event) {
		if (sessionInfo.stompClient && sessionInfo.stompClient.connected) {
			sessionInfo.stompClient.send("/code-together/room/{roomId}", JSON.stringify({
				roomId: sessionInfo.roomId,
				username: sessionInfo.username,
				...cursorInfo,
				key: event.key
			}), {});
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
				bind:cursorInfo={cursorInfo}
				onKeyDownCallback="{textareaEditCallback}">
		</CodeTextarea>
	{/if}
</main>

<style>

</style>