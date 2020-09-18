<template>
	<v-app>
		<v-app-bar app color="primary" dark>
			<div class="d-flex align-center">Chat Bot</div>
		</v-app-bar>

		<v-main>
			<div class="chat_scrollview">
				<div class="chat_messagelist">
					<div v-for="(message, index) in toChat" :key="index" class="message" :class="message.owner">
						<!-- if chatBot message then apply typed.js, send second message with delay -->
						<vue-typed-js v-if="message.owner == 'him'" :startDelay="message.ask ? 200 : 0" :strings="[message.text]" :typeSpeed="20" :showCursor="false">
							<span class="typing"></span>
						</vue-typed-js>
						<span v-else> {{ message.text }}</span>
					</div>
					<!-- render spinner if user typing -->
					<spinner v-if="input != ''" />
				</div>
			</div>

			<div class="chat_input">
				<v-col cols="12" md="12">
					<!-- trigger send method on textarea enter  -->
					<v-textarea v-model="input" :disabled="disable || current == 0" placeholder="Type your message" rows="3" filled hideDetails noResize rounded autofocus @keyup.enter="send"></v-textarea>
				</v-col>
				<v-col cols="12" md="12">
					<div class="my-2">
						<v-btn v-if="current == 0" xLarge color="primary" @click="send">Let's Chat</v-btn>
						<v-btn v-else :disabled="disable" xLarge color="primary" @click="send">Send Message</v-btn>
					</div>
				</v-col>
			</div>
		</v-main>
	</v-app>
</template>

<script>
import spinner from './components/spinner'
// import VueTypedJs from 'vue-typed-js'

export default {
	name: 'App',
	components: { spinner },
	data: () => ({
		// user collected data
		user: {
			name: '',
			age: 0,
			location: '',
			feeling: '',
			hobby: '',
		},
		current: 0,
		input: '',
		toChat: [],
		messages: [],
	}),
	computed: {
		// check when to disable btn
		disable() {
			if (this.current == this.messages.length) return true
			else return false
		},
	},
	mounted() {
		// get messages with axios
		this.$axios.get('http://localhost:8080/messages.json').then((response) => {
			this.messages = response.data
		})
	},
	methods: {
		send() {
			// avoid error on the end
			if (this.current == this.messages.length) return

			if (this.input != '') {
				// add user input to messages
				this.messages.splice(this.current, 0, {
					text: this.input,
					owner: 'me',
				})

				// store user input
				switch (this.messages[this.current - 1].ask) {
					case 'name':
						console.log('name')
						this.user.name = this.input
						break
					case 'age':
						console.log('age')
						this.user.age = this.input
						break
					case 'location':
						console.log('location')
						this.user.location = this.input
						break
					case 'feeling':
						console.log('feeling')
						this.user.feeling = this.input
						break
					case 'hobby':
						console.log('hobby')
						this.user.hobby = this.input
						break
					default:
						console.log(`default value`)
				}

				// reset input
				this.input = ''
			}

			if (this.messages[this.current].owner == 'me') {
				// user messages
				this.toChat.push(this.messages[this.current])
				this.current += 1
				this.send()
			} else {
				// chatBot messages
				if (typeof this.messages[this.current].ask === 'undefined') {
					// regular message
					this.toChat.push(this.messages[this.current])
					this.current += 1
					this.send()
				} else {
					// question message
					this.toChat.push(this.messages[this.current])
					this.current += 1
				}
			}
		},
	},
}
</script>

<style lang="scss">
::-webkit-scrollbar {
	display: none;
}
.v-application {
	width: 100%;
	height: 100vh;
}
.v-main {
	display: flex;
	flex-direction: column;
}

.chat_scrollview {
	overflow-y: auto;
	overflow-x: hidden;
	-webkit-overflow-scrolling: touch;
	overscroll-behavior: contain;
	scroll-snap-type: y proximity;
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;

	.chat_messagelist {
		display: flex;
		flex-wrap: wrap;
		flex-direction: column;
		justify-content: flex-end;
		align-items: flex-end;
		min-height: 100%;
		padding: 0 17px 220px;

		.message {
			display: flex;
			flex-wrap: wrap;
			align-self: flex-start;
			align-items: flex-end;

			width: fit-content;
			padding: 20px;
			border-radius: 30px;
			margin-bottom: 2px;
			&.him {
				background: #eee;
			}
			&.me {
				align-self: flex-end;
				background: #0084ff;
				color: #fff;
				border-bottom-right-radius: 0;
			}
		}
		.message:last-child {
			scroll-snap-align: end;
			scroll-margin-block-end: 220px;
		}
	}
}

.chat_input {
	position: absolute;
	right: 0;
	bottom: 0;

	padding: 10px;
	z-index: 100;
	width: 100%;
	background: white;
	box-shadow: 0 15px 20px 5px #a8a8a8;
	padding: 0 5px;
}
</style>
