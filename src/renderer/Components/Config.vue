<template>
	<div v-if="applicable" class="config">
		<h1>CS:GO HUD Series Data</h1>

		<div class="buttons">
			<button @click.prevent="matches.push({})">
				Add Match
			</button>

			<button v-if="matches.length > 0" @click.prevent="submit">
				Save
			</button>
		</div>

		<div class="input-group primary-team">
			<label for="primary-team">
				Name of Team that should be on the left
			</label>
			<input type="text" id="primary-team" v-model="primaryTeam">
		</div>

		<div class="match" v-for="(match, i) in matches">
			<div class="number centered">
				#{{ i + 1 }}
			</div>

			<div class="input-group">
				<label :for="`map-${i}`">Map</label>
				<input type="text" :id="`map-${i}`" v-model="matches[i].map" required>
			</div>

			<div class="input-group">
				<label>Picked by</label>
				<label :for="`picked-${i}-left`" class="inline">left</label>
				<input type="radio" :id="`picked-${i}-left`" value="0" v-model="matches[i].picked">
				<input type="radio" :id="`picked-${i}-right`" value="1" v-model="matches[i].picked">
				<label :for="`picked-${i}-right`" class="inline">right</label>

				<div>
					<input type="radio" :id="`picked-${i}-neither`" :value="null" v-model="matches[i].picked">
					<label :for="`picked-${i}-neither`" class="inline">neither</label>
				</div>
			</div>

			<div class="input-group">
				<label :for="`score-left-${i}`">Score</label>
				<input type="number" :id="`score-left-${i}`" min="0" v-model="matches[i].scoreLeft">

				<label class="inline" :for="`score-right-${i}`">:</label>
				<input type="number" :id="`score-right-${i}`" min="0" v-model="matches[i].scoreRight">
			</div>

			<div class="input-group centered">
				<input type="checkbox" :id="`currently-playing-${i}`" v-model="matches[i].currentlyPlaying">
				<label :for="`currently-playing-${i}`" class="inline">
					Currently Playing
				</label>
			</div>

			<div class="input-group centered">
				<button @click.prevent="matches.splice(i, 1)">
					Remove Match
				</button>
			</div>
		</div>
	</div>
</template>

<script>
import { ipcRenderer } from 'electron'

export default {
	data() {
		return {
			matches: [{}],
			primaryTeam: 'Das Deutsche Volk',
		}
	},

	methods: {
		submit() {
			ipcRenderer.send('seriesData', {
				matches: this.matches,
				primaryTeam: this.primaryTeam,
			})
		},
	},

	computed: {
		applicable() {
			return window.location.hash === '#config'
		},
	},
}
</script>