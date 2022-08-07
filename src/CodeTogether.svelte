<script>
	import Header from "./Header.svelte";
	import LoginForm from "./LoginForm.svelte";
	// import SockJS from "sockjs-client"; // why not works?
	import Stomp from "webstomp-client";

	let sessionInfo = {
		isUserLoggedIn: false,
		roomId: 'room1',
		username: '',
		stompClient: null
	}

	let textareaInfo = {
		text: ''
	}

	let successfulLoginCallback = () => {
		const socket = new WebSocket("ws://192.168.88.14:8081/code-together");
		sessionInfo.stompClient = Stomp.over(socket);
		sessionInfo.stompClient.connect(
		{},
		frame => {
			console.log("Connection frame: ", frame);

			sessionInfo.stompClient.subscribe("/room/room1", (message) => {
				let jsonMessage = JSON.parse(message.body);
				console.log("Message over sockjs: ", jsonMessage, "isUserEqual: ", jsonMessage.username === sessionInfo.username);
				if (jsonMessage.username === sessionInfo.username) {
					console.log("Ignore. Message from myself!");
				} else {
					textareaInfo.text = jsonMessage.text;
				}
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

	let textareaEditCallback = function () {
		if (sessionInfo.stompClient && sessionInfo.stompClient.connected) {
			sessionInfo.stompClient.send("/code-together/room/{roomId}", JSON.stringify({
				roomId: sessionInfo.roomId,
				username: sessionInfo.username,
				text: textareaInfo.text
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
		<textarea cols="150"
				  rows="20"
				  bind:value={textareaInfo.text}
				  on:keyup={textareaEditCallback}></textarea>
	{/if}
</main>

<style>

</style>