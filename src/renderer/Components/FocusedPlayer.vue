<template>
	<div :class="['focused-player', { '--active': focusedPlayer && focusedPlayer.spectarget && focusedPlayer.spectarget !== 'free' }]">
		<div :class="`name --${side}`">
			<img v-if="team.flag" :src="`https://flagcdn.com/h120/${team.flag}.png`" class="flag">
			<img v-else-if="side === 'ct'" src="../../img/ct.svg" class="flag">
			<img v-else src="../../img/t.svg" class="flag">
			<div class="label">{{ player.name }}</div>
		</div>

		<div :class="`life --${side}`">
			<div class="hp">
				<img src="../../img/health.svg">
				<div class="number">{{ player.state.health }}</div>
			</div>

			<div class="armor">
				<img v-if="player.state.helmet" src="../../img/armor.svg">
				<img v-else src="../../img/shield.svg">
				<div class="number">{{ player.state.armor }}</div>
			</div>
		</div>

		<div class="stats">
			<div v-if="player.state.round_kills > 0" class="round-kills">
				<img src="../../img/elimination.svg">
				<div class="number">{{ player.state.round_kills }}</div>
			</div>

			<div :class="`label --round-damage --${side}`">RD DMG</div>
			<div :class="`label --kills --${side}`">K</div>
			<div :class="`label --assists --${side}`">A</div>
			<div :class="`label --deaths --${side}`">D</div>
			<div :class="`label --adr --${side}`">ADR</div>

			<div class="number --round-damage">{{ player.state.round_totaldmg }}</div>
			<div class="number --kills">{{ player.match_stats.kills }}</div>
			<div class="number --assists">{{ player.match_stats.assists }}</div>
			<div class="number --deaths">{{ player.match_stats.deaths }}</div>
			<div class="number --adr">{{ Math.round((adr[player.steamid] || 0)) }}</div>
		</div>

		<div class="ammo-utility">
			<div class="bomb">
				<img v-if="player.state.defusekit" src="../../img/weapons/defuser.svg">
				<img v-else-if="carryingBomb" src="../../img/weapons/c4.svg">
			</div>

			<div class="grenades">
				<img
					v-for="[grenade, active] in grenades"
					:src="require(`../../img/weapons/${grenade}.svg`)"
					:class="['grenade', { '--active' : active }]"
				>
			</div>

			<div v-if="activeWeapon && ! ['Grenade', 'Knife', 'C4'].includes(activeWeapon.type)" class="ammo">
				<div class="ammo-current">{{ activeWeapon.ammo_clip }}</div>
				<div :class="`ammo-reserve --${side}`">/{{ activeWeapon.ammo_reserve }}</div>
			</div>
		</div>
	</div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
	props: [
		'adr',
	],

	data() {
		return {
			player: { match_stats: {}, state: {} },
		}
	},

	computed: {
		...mapGetters({
			map: 'map',
			focusedPlayer: 'player',
		}),

		activeWeapon() {
			for (const key in this.player.weapons) {
				if (this.player.weapons[key].state === 'active') return this.player.weapons[key]
			}
		},

		grenades() {
			const grenades = []

			for (const key in this.player.weapons) {
				const { ammo_reserve, type, name, state } = this.player.weapons[key]

				if (type === 'Grenade') {
					for (let i = 0; i < ammo_reserve; i++) {
						grenades.push([name.replace(/^weapon_/, ''), state === 'active'])
					}
				}
			}

			return grenades.sort((a, b) => {
				if (a[0] !== b[0]) return a[0] > b[0] ? 1 : -1
				return 0
			})
		},

		side() {
			return (this.player.team || 'ct').toLowerCase()
		},

		team() {
			return this.map[`team_${this.side}`]
		},

		carryingBomb() {
			for (const key in this.player.weapons) {
				if (this.player.weapons[key].type === 'C4') return true
			}

			return false
		},
	},

	watch: {
		focusedPlayer(focusedPlayer) {
			if (focusedPlayer.spectarget && focusedPlayer.spectarget !== 'free') this.player = focusedPlayer
		},
	},
}
</script>
