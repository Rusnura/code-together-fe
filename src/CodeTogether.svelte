<script>
	import Header from "./Header.svelte";
	import LoginForm from "./LoginForm.svelte";
	// import SockJS from "sockjs-client"; // why not works?
	import Stomp from "webstomp-client";
	import CodeTextarea from "./components/CodeTextarea.svelte";
	import DebugComponent from "./components/DebugComponent.svelte";

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
		cursors: []
	}

	let cursorInfo = {
		cursorOwner: sessionInfo.username,
		startCursorPosition: 0,
		endCursorPosition: 0,
	}

	let successfulLoginCallback = () => {
		const socket = new WebSocket("ws://192.168.88.14:8081/code-together");
		sessionInfo.stompClient = Stomp.over(socket);
		sessionInfo.stompClient.connect({}, frame => {
			console.log("Connection frame: ", frame);

			sessionInfo.stompClient.subscribe("/code-together/room/" + sessionInfo.roomId, (raw) => {
				console.log("--> Handle: ", "/code-together/room/" + sessionInfo.roomId, ". Message: ", raw);
				let message = JSON.parse(raw.body);
				if (message.type === "connect") {
					handleNewUserConnected(message);
				} else if (message.type === "insert") {
					handleInsert(message);
				} else if (message.type === "backspace") {
					handleBackspace(message);
				} else if (message.type === "delete") {
					handleDelete(message);
				} else if (message.type === "navigation") {
					handleNavigation(message);
				}
			}, {
				"username": sessionInfo.username
			});

			sessionInfo.stompClient.subscribe("/code-together/room/" + sessionInfo.roomId + "/status", (raw) => {
				let message = JSON.parse(raw.body);
				console.log("--> Handle: ", "/code-together/room/" + sessionInfo.roomId + "/status. Message: ", message);
				textareaProperties.text = message.text;

				for (let i = 0; i < message.clients.length; i++) {
					let client = message.clients[i];
					console.log("Cursor inserting processing: ", client);
					if (client.username === sessionInfo.username)
						continue;


					let existingUser = findCursorByOwner(client.username);
					if (!existingUser) {
						textareaProperties.cursors.push({
							owner: client.username,
							startCursorPosition: 0,
							endCursorPosition: 0
						});
					} else {
						existingUser.startCursorPosition = client.startCursorPosition;
						existingUser.endCursorPosition = client.endCursorPosition;
					}
				}

				console.log("All cursors inserted: ", textareaProperties);
			}, {
				"roomId": sessionInfo.roomId
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

		findCursorByOwner(message.username).startCursorPosition = message.startCursorPosition;
		findCursorByOwner(message.username).endCursorPosition = message.endCursorPosition;
	}

	let handleInsert = function (message) {
		if (message.username === sessionInfo.username)
			return;

		let start = findCursorByOwner(message.username).startCursorPosition;
		let end = findCursorByOwner(message.username).endCursorPosition;
		let before = textareaProperties.text.substring(0, start);
		let after = textareaProperties.text.substring(end);
		textareaProperties.text = before + message.key + after;
	}

	let handleBackspace = function (message) {
		if (message.username === sessionInfo.username)
			return;

		let start = findCursorByOwner(message.username).startCursorPosition;
		let end = findCursorByOwner(message.username).endCursorPosition;

		let before, after;
		let newCursorStartPosition, newCursorEndPosition;
		if (start === end) {
			if (start === 0)
				return;
			before = textareaProperties.text.substring(0, start - 1);
			newCursorStartPosition = start - 1;
			newCursorEndPosition = start - 1;
		} else {
			before = textareaProperties.text.substring(0, start);
			newCursorStartPosition = start;
			newCursorEndPosition = start;
		}
		after = textareaProperties.text.substring(end);
		textareaProperties.text = before + after;

		findCursorByOwner(message.username).startCursorPosition = newCursorStartPosition;
		findCursorByOwner(message.username).endCursorPosition = newCursorEndPosition;
	}

	let handleDelete = function (message) {
		if (message.username === sessionInfo.username)
			return;

		let start = findCursorByOwner(message.username).startCursorPosition;
		let end = findCursorByOwner(message.username).endCursorPosition;

		let before, after;
		let newCursorStartPosition, newCursorEndPosition;
		if (start === end) {
			if (textareaProperties.text < end + 1)
				return;
			before = textareaProperties.text.substring(0, start);
			after = textareaProperties.text.substring(end + 1);
		} else {
			before = textareaProperties.text.substring(0, start);
			after = textareaProperties.text.substring(end);
		}
		newCursorStartPosition = start;
		newCursorEndPosition = start;
		textareaProperties.text = before + after;
		findCursorByOwner(message.username).startCursorPosition = newCursorStartPosition;
		findCursorByOwner(message.username).endCursorPosition = newCursorEndPosition;
	}

	let handleNewUserConnected = function (message) {
		if (message.username === sessionInfo.username)
			return;

		let existingUser = findCursorByOwner(message.username);
		if (!existingUser) {
			textareaProperties.cursors.push({
				owner: message.username,
				startCursorPosition: 0,
				endCursorPosition: 0
			});
		} else {
			existingUser.startCursorPosition = 0;
			existingUser.endCursorPosition = 0;
		}
	};

	let failureLoginCallback = function() {
		console.log("failureLoginCallback: sessionInfo=", sessionInfo);
	}

	let textareaEditCallback = function (event) {
		if (sessionInfo.stompClient && sessionInfo.stompClient.connected) {
			sessionInfo.stompClient.send("/code-together/room/" + sessionInfo.roomId, JSON.stringify({
				roomId: sessionInfo.roomId,
				username: sessionInfo.username,
				...cursorInfo,
				key: event.key
			}), {
				"roomId": sessionInfo.roomId,
				"username": sessionInfo.username
			});
		}
	}

	let findCursorByOwner = function (username) {
		let cursors = textareaProperties.cursors.filter(u => u.owner === username);
		return cursors.length === 1 ? cursors[0] : undefined;
	}
</script>

<main>
	{#if !sessionInfo.isUserLoggedIn}
		<Header />

		<LoginForm
				bind:sessionInfo="{sessionInfo}"
				successfulLoginCallback="{successfulLoginCallback}"
				failureLoginCallback="{failureLoginCallback}"
		/>
	{:else}
		<CodeTextarea
				bind:textareaProperties={textareaProperties}
				bind:cursorInfo={cursorInfo}
				onKeyDownCallback="{textareaEditCallback}">
		</CodeTextarea>
		<DebugComponent
				bind:cursorInfo={cursorInfo}
				bind:textareaProperties={textareaProperties}
				bind:sessionInfo={sessionInfo}>
		</DebugComponent>
	{/if}
</main>

<style>

</style>