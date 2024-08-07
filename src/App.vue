<template lang="pug">
.pretalx-schedule(:style="{'--scrollparent-width': scrollParentWidth + 'px', '--schedule-max-width': scheduleMaxWidth + 'px', '--pretalx-sticky-date-offset': days && days.length > 1 ? '48px' : '0px'}", :class="showGrid ? ['grid-schedule'] : ['list-schedule']")
	template(v-if="scheduleError")
		.schedule-error
			.error-message An error occurred while loading the schedule. Please try again later.
	template(v-else-if="schedule && sessions")
		.modal-overlay(v-if="showFilterModal", @click.stop="showFilterModal = false")
			//- .modal-box(@click.stop="")
			//- 	h3 Tracks
			//- 	.checkbox-line(v-for="track in allTracks", :key="track.value", :style="{'--track-color': track.color}")
			//- 		bunt-checkbox(type="checkbox", :label="track.label", :name="track.value + track.label", v-model="track.selected", :value="track.value", @input="onlyFavs = false")
			//- 		.track-description(v-if="getLocalizedString(track.description).length") {{ getLocalizedString(track.description) }}
		.settings
			v-select(v-model="selectedTracks", :closeOnSelect="false", :options="allTracks", multiple, placeholder="Tracks")
			v-select(v-model="selectedRooms", :closeOnSelect="false", :options="allRooms", multiple, placeholder="Rooms")
			v-select(v-model="selectedSessionTypes", :closeOnSelect="false", :options="allSessionTypes", multiple, placeholder="Session Types")
			//- bunt-button.filter-tracks(v-if="this.schedule.tracks.length", @click="showFilterModal=true")
				svg#filter(viewBox="0 0 752 752")
					path(d="m401.57 264.71h-174.75c-6.6289 0-11.84 5.2109-11.84 11.84 0 6.6289 5.2109 11.84 11.84 11.84h174.75c5.2109 17.523 21.312 30.309 40.727 30.309 18.941 0 35.52-12.785 40.254-30.309h43.098c6.6289 0 11.84-5.2109 11.84-11.84 0-6.6289-5.2109-11.84-11.84-11.84h-43.098c-5.2109-17.523-21.312-30.309-40.254-30.309-19.414 0-35.516 12.785-40.727 30.309zm58.723 11.84c0 10.418-8.5234 18.469-18.469 18.469s-18.469-8.0508-18.469-18.469 8.5234-18.469 18.469-18.469c9.4727-0.003906 18.469 8.0469 18.469 18.469z")
					path(d="m259.5 359.43h-32.676c-6.6289 0-11.84 5.2109-11.84 11.84s5.2109 11.84 11.84 11.84h32.676c5.2109 17.523 21.312 30.309 40.727 30.309 18.941 0 35.52-12.785 40.254-30.309h185.17c6.6289 0 11.84-5.2109 11.84-11.84s-5.2109-11.84-11.84-11.84h-185.17c-5.2109-17.523-21.312-30.309-40.254-30.309-19.418 0-35.52 12.785-40.73 30.309zm58.723 11.84c0 10.418-8.5234 18.469-18.469 18.469-9.9453 0-18.469-8.0508-18.469-18.469s8.5234-18.469 18.469-18.469c9.9453 0 18.469 8.0508 18.469 18.469z")
					path(d="m344.75 463.61h-117.92c-6.6289 0-11.84 5.2109-11.84 11.84s5.2109 11.84 11.84 11.84h117.92c5.2109 17.523 21.312 30.309 40.727 30.309 18.941 0 35.52-12.785 40.254-30.309h99.926c6.6289 0 11.84-5.2109 11.84-11.84s-5.2109-11.84-11.84-11.84h-99.926c-5.2109-17.523-21.312-30.309-40.254-30.309-19.418 0-35.52 12.785-40.727 30.309zm58.723 11.84c0 10.418-8.5234 18.469-18.469 18.469s-18.469-8.0508-18.469-18.469 8.5234-18.469 18.469-18.469 18.469 8.0508 18.469 18.469z")
				template Filter
				template(v-if="filteredTracks.length") ({{ filteredTracks.length }})
			bunt-button.fav-toggle(v-if="favs.length", @click="onlyFavs = !onlyFavs; if (onlyFavs) resetFilteredTracks()", :class="onlyFavs ? ['active'] : []")
				svg#star(viewBox="0 0 24 24")
					polygon(
						:style="{fill: '#FFA000', stroke: '#FFA000'}"
						points="14.43,10 12,2 9.57,10 2,10 8.18,14.41 5.83,22 12,17.31 18.18,22 15.83,14.41 22,10"
					)
				template {{ favs.length }}
			template(v-if="!inEventTimezone")
				bunt-select(name="timezone", :options="[{id: schedule.timezone, label: schedule.timezone}, {id: userTimezone, label: userTimezone}]", v-model="currentTimezone", @blur="saveTimezone")
			template(v-else)
				div.timezone-label.bunt-tab-header-item {{ schedule.timezone }}
		bunt-tabs.days(v-if="days && days.length > 1", :active-tab="currentDay && currentDay.format()", ref="tabs" :class="showGrid? ['grid-tabs'] : ['list-tabs']")
			bunt-tab(v-for="day in days", :id="day.format()", :header="day.format(dateFormat)", @selected="changeDay(day)")
		grid-schedule(v-if="showGrid",
			:sessions="sessions",
			:rooms="rooms",
			:currentDay="currentDay",
			:now="now",
			:scrollParent="scrollParent",
			:favs="favs",
			@changeDay="currentDay = $event",
			@fav="fav($event)",
			@unfav="unfav($event)")
		linear-schedule(v-else,
			:sessions="sessions",
			:rooms="rooms",
			:currentDay="currentDay",
			:now="now",
			:scrollParent="scrollParent",
			:favs="favs",
			@changeDay="currentDay = $event",
			@fav="fav($event)",
			@unfav="unfav($event)")
		.modal(v-if="showModal")
			.modal-content
				.modal-header
					.h3.modal-title Warning
				.modal-body.p Please login to add a session to your personal schedule.
				.modal-footer
					.button(@click="closeModal") OK
	bunt-progress-circular(v-else, size="huge", :page="true")
</template>
<script>
import Vue from 'vue'
import Buntpapier from 'buntpapier'
import moment from 'moment-timezone'
import LinearSchedule from 'components/LinearSchedule'
import GridSchedule from 'components/GridSchedule'
import { findScrollParent, getLocalizedString } from 'utils'
import vSelect from 'vue-select'

Vue.component('v-select', vSelect)

Vue.use(Buntpapier)

export default {
	name: 'PretalxSchedule',
	components: { LinearSchedule, GridSchedule },
	props: {
		eventUrl: String,
		locale: String,
		format: {
			type: String,
			default: 'grid'
		},
		version: {
			type: String,
			default: ''
		}
	},
	provide () {
		return {
			eventUrl: this.eventUrl
		}
	},
	data () {
		return {
			moment,
			getLocalizedString,
			scrollParentWidth: Infinity,
			schedule: null,
			userTimezone: null,
			now: moment(),
			currentDay: null,
			currentTimezone: null,
			showFilterModal: false,
			favs: [],
			allTracks: [],
			onlyFavs: false,
			scheduleError: false,
			selectedTracks: [],
			showModal: false,
			allRooms: [],
			allSessionTypes: [],
			selectedRooms: [],
			selectedSessionTypes: [],
		}
	},
	computed: {
		scheduleMaxWidth () {
			return this.schedule ? Math.min(this.scrollParentWidth, 78 + this.schedule.rooms.length * 650) : this.scrollParentWidth
		},
		showGrid () {
			return this.scrollParentWidth > 710 && this.format !== 'list' // if we can't fit two rooms together, switch to list
		},
		roomsLookup () {
			if (!this.schedule) return {}
			return this.schedule.rooms.reduce((acc, room) => { acc[room.id] = room; return acc }, {})
		},
		tracksLookup () {
			if (!this.schedule) return {}
			return this.schedule.tracks.reduce((acc, t) => { acc[t.id] = t; return acc }, {})
		},
		filteredTracks () {
			return this.allTracks.filter(t => t.selected)
		},
		speakersLookup () {
			if (!this.schedule) return {}
			return this.schedule.speakers.reduce((acc, s) => { acc[s.code] = s; return acc }, {})
		},
		sessions () {
			if (!this.schedule || !this.currentTimezone) return
			const sessions = []
			for (const session of this.schedule.talks.filter(s => s.start)) {
				if (this.onlyFavs && !this.favs.includes(session.code)) continue
				if (this.filteredTracks && this.filteredTracks.length && !this.filteredTracks.find(t => t.id === session.track)) continue
				sessions.push({
					id: session.code,
					title: session.title,
					abstract: session.abstract,
					start: moment.tz(session.start, this.currentTimezone),
					end: moment.tz(session.end, this.currentTimezone),
					speakers: session.speakers?.map(s => this.speakersLookup[s]),
					track: this.tracksLookup[session.track],
					room: this.roomsLookup[session.room],
					fav_count: session.fav_count,
					do_not_record: session.do_not_record,
					tags: session.tags
				})
			}
			sessions.sort((a, b) => a.start.diff(b.start))
			return sessions
		},
		rooms () {
			return this.schedule.rooms.filter(r => this.sessions.some(s => s.room === r))
		},
		days () {
			if (!this.sessions) return
			const days = []
			for (const session of this.sessions) {
				if (days[days.length - 1] && days[days.length - 1].isSame(session.start, 'day')) continue
				days.push(session.start.clone().startOf('day'))
			}
			return days
		},
		inEventTimezone () {
			if (!this.schedule || !this.schedule.talks) return false
			const example = this.schedule.talks[0].start
			return moment.tz(example, this.userTimezone).format('Z') === moment.tz(example, this.schedule.timezone).format('Z')
		},
		dateFormat () {
			// Defaults to dddd DD. MMMM for: all grid schedules with more than two rooms, and all list schedules with less than five days
			// After that, we start to shorten the date string, hoping to reduce unwanted scroll behaviour
			if ((this.showGrid && this.schedule && this.schedule.rooms.length > 2) || !this.days || !this.days.length) return 'dddd DD. MMMM'
			if (this.days && this.days.length <= 5) return 'dddd DD. MMMM'
			if (this.days && this.days.length <= 7) return 'dddd DD. MMM'
			return 'ddd DD. MMM'
		},
		eventSlug () {
			let url = ''
			if (this.eventUrl.startsWith('http')) {
				url = new URL(this.eventUrl)
			} else {
				url = new URL('http://example.org/' + this.eventUrl)
			}
			return url.pathname.replace(/\//g, '')
		}
	},
	async created () {
		moment.locale(this.locale)
		this.userTimezone = moment.tz.guess()
		let version = ''
		if (this.version)
			version = `v/${this.version}/`
		const url = `${this.eventUrl}schedule/${version}widgets/schedule.json`
		const legacyUrl = `${this.eventUrl}schedule/${version}widget/v2.json`
		// fetch from url, but fall back to legacyUrl if url fails
		try {
			this.schedule = await (await fetch(url)).json()
		} catch (e) {
			try {
				this.schedule = await (await fetch(legacyUrl)).json()
			} catch (e) {
				this.scheduleError = true
				return
			}
		}
		this.currentTimezone = localStorage.getItem(`${this.eventSlug}_timezone`)
		this.currentTimezone = [this.schedule.timezone, this.userTimezone].includes(this.currentTimezone) ? this.currentTimezone : this.schedule.timezone
		this.currentDay = this.days[0]
		this.now = moment().tz(this.currentTimezone)
		setInterval(() => this.now = moment().tz(this.currentTimezone), 30000)
		if (!this.scrollParentResizeObserver) {
			await this.$nextTick()
			this.onWindowResize()
		}
		this.schedule.tracks.forEach(t => { t.value = t.id; t.label = getLocalizedString(t.name); this.allTracks.push(t) })
		this.schedule.rooms.forEach(t => { t.value = t.id; t.label = getLocalizedString(t.name); this.allRooms.push(t) })
		this.favs = this.pruneFavs(await this.loadFavs(), this.schedule)
		const fragment = window.location.hash.slice(1)
		if (fragment && fragment.length === 10) {
			const initialDay = moment(fragment, 'YYYY-MM-DD')

			const filteredDays = this.days.filter(d => d.format('YYYYMMDD') === initialDay.format('YYYYMMDD'))
			if (filteredDays.length) {
				this.currentDay = filteredDays[0]
			}
		}
	},
	async mounted () {
		// We block until we have either a regular parent or a shadow DOM parent
		await new Promise((resolve) => {
			const poll = () => {
				if (this.$el.parentElement || this.$el.getRootNode().host) return resolve()
				setTimeout(poll, 100)
			}
			poll()
		})
		this.scrollParent = findScrollParent(this.$el.parentElement || this.$el.getRootNode().host)
		if (this.scrollParent) {
			this.scrollParentResizeObserver = new ResizeObserver(this.onScrollParentResize)
			this.scrollParentResizeObserver.observe(this.scrollParent)
			this.scrollParentWidth = this.scrollParent.offsetWidth
		} else { // scrolling document
			window.addEventListener('resize', this.onWindowResize)
			this.onWindowResize()
		}
	},
	destroyed () {
		// TODO destroy observers
	},
	methods: {
		changeDay (day) {
			if (day.isSame(this.currentDay)) return
			this.currentDay = moment(day, this.currentTimezone).startOf('day')
			window.location.hash = day.format('YYYY-MM-DD')
		},
		onWindowResize () {
			this.scrollParentWidth = document.body.offsetWidth
		},
		saveTimezone () {
			localStorage.setItem(`${this.eventSlug}_timezone`, this.currentTimezone)
		},
		onScrollParentResize (entries) {
			this.scrollParentWidth = entries[0].contentRect.width
		},
		async loadFavs () {
			const dataCached = localStorage.getItem(`${this.eventSlug}_favs`)
			let userData = []
			try {
				userData = await (await fetch(`/api/events/${this.eventSlug}/favourite-talk/`,
						{
							method: 'GET',
							headers: {
								'Content-Type': 'application/json'
							}
						})).json()
			} catch (e) {
				console.error('error happened when trying to load favourite talk')
			}
			let data
			if (userData.length !== 0) {
				data = JSON.stringify(userData)
			} else {
				data = dataCached
			}
			if (data) {
				try {
					return JSON.parse(data)
				} catch {
					localStorage.setItem(`${this.eventSlug}_favs`, '[]')
				}
			}
			return []
		},
		pruneFavs (favs, schedule) {
			const talks = schedule.talks || []
			const talkIds = talks.map(e => e.code)
			return favs.filter(e => talkIds.includes(e))
		},
		closeModal() {
			this.showModal = false;
		},
		async saveFavs () {
			try {
				const response = await (await fetch(`/api/events/${this.eventSlug}/favourite-talk/`,
						{
							method: 'POST',
							headers: {
								'Content-Type': 'application/json'
							},
							body: JSON.stringify(this.favs)
						}))				
				if (response.status === 400) {
					const data = await response.json()
					if (data === 'user_not_logged_in') {
						this.showModal = true;
						return
					}
				}
			} catch (e) {
				console.error(`error happened when trying to save favourite talk: ${JSON.stringify(this.favs)}`)
			}
			localStorage.setItem(`${this.eventSlug}_favs`, JSON.stringify(this.favs))
		},
		async fav (id) {
			if (!this.favs.includes(id)) {
				this.favs.push(id)
				await this.saveFavs()
			}
		},
		async unfav (id) {
			this.favs = this.favs.filter(elem => elem !== id)
			await this.saveFavs()
			if (!this.favs.length) this.onlyFavs = false
		},
		resetFilteredTracks () {
			this.allTracks.forEach(t => t.selected = false)
		}
	}
}
</script>
<style lang="stylus">
@import 'styles/global.styl'
.schedule-error
	color: $clr-error
	font-size: 18px
	text-align: center
	padding: 32px
	.error-message
		margin-top: 16px
.pretalx-schedule
	display: flex
	flex-direction: column
	min-height: 0
	height: 100%
	font-size: 14px
	&.grid-schedule
		min-width: min-content
		max-width: var(--schedule-max-width)
		margin: 0 auto
	&.list-schedule
		min-width: 0
	.modal-overlay
		position: fixed
		z-index: 1000
		top: 0
		left: 0
		width: 100%
		height: 100%
		background-color: rgba(0,0,0,0.4)
		.modal-box
			width: 600px
			max-width: calc(95% - 64px)
			border-radius: 32px
			padding: 4px 32px
			margin-top: 32px
			background: white
			margin-left: auto
			margin-right: auto
			.checkbox-line
				margin: 16px 8px
				.bunt-checkbox.checked .bunt-checkbox-box
					background-color: var(--track-color)
					border-color: var(--track-color)
				.track-description
					color: $clr-grey-600
					margin-left: 32px
	.settings
		margin-left: 18px
		align-self: flex-start
		display: flex
		align-items: center
		position: sticky
		z-index: 100
		left: 18px
		width: 100%
		.fav-toggle
			margin-right: 8px
			display: flex
			&.active
				border: #FFA000 2px solid
			.bunt-button-text
				display: flex
				align-items: center
			svg
				width: 20px
				height: 20px
				margin-right: 6px
		.filter-tracks
			margin-right: 8px
			display: flex
			.bunt-button-text
				display: flex
				align-items: center
				padding-right: 8px
			svg
				width: 36px
				height: 36px
				margin-right: 6px
		.bunt-select
			max-width: 300px
			padding-right: 8px
		.timezone-label
			cursor: default
			color: $clr-secondary-text-light
		// .bunt-select, .timezone-label
		// 	margin-left: auto
	.days
		background-color: $clr-white
		tabs-style(active-color: var(--pretalx-clr-primary), indicator-color: var(--pretalx-clr-primary), background-color: transparent)
		overflow-x: auto
		position: sticky
		top: var(--pretalx-sticky-top-offset, 0px)
		left: 0
		margin-bottom: 0
		flex: none
		min-width: 0
		max-width: var(--schedule-max-width)
		height: 48px
		z-index: 30
		.bunt-tabs-header
			min-width: min-content
		.bunt-tabs-header-items
			justify-content: center
			min-width: min-content
			.bunt-tab-header-item
				min-width: min-content
			.bunt-tab-header-item-text
				white-space: nowrap

:host,
:root {
  --vs-colors--lightest: rgba(60,60,60,0.26);
  --vs-colors--light: rgba(60,60,60,0.5);
  --vs-colors--dark: #333;
  --vs-colors--darkest: rgba(0,0,0,0.15);
  --vs-search-input-color: inherit;
  --vs-search-input-bg: #fff;
  --vs-search-input-placeholder-color: inherit;
  --vs-font-size: 1rem;
  --vs-line-height: 1.4;
  --vs-state-disabled-bg: #f8f8f8;
  --vs-state-disabled-color: var(--vs-colors--light);
  --vs-state-disabled-controls-color: var(--vs-colors--light);
  --vs-state-disabled-cursor: not-allowed;
  --vs-border-color: var(--vs-colors--lightest);
  --vs-border-width: 1px;
  --vs-border-style: solid;
  --vs-border-radius: 4px;
  --vs-actions-padding: 4px 6px 0 3px;
  --vs-controls-color: var(--vs-colors--light);
  --vs-controls-size: 1;
  --vs-controls--deselect-text-shadow: 0 1px 0 #fff;
  --vs-selected-bg: #f0f0f0;
  --vs-selected-color: var(--vs-colors--dark);
  --vs-selected-border-color: var(--vs-border-color);
  --vs-selected-border-style: var(--vs-border-style);
  --vs-selected-border-width: var(--vs-border-width);
  --vs-dropdown-bg: #fff;
  --vs-dropdown-color: inherit;
  --vs-dropdown-z-index: 1000;
  --vs-dropdown-min-width: 160px;
  --vs-dropdown-max-height: 350px;
  --vs-dropdown-box-shadow: 0px 3px 6px 0px var(--vs-colors--darkest);
  --vs-dropdown-option-bg: #000;
  --vs-dropdown-option-color: var(--vs-dropdown-color);
  --vs-dropdown-option-padding: 3px 20px;
  --vs-dropdown-option--active-bg: #5897fb;
  --vs-dropdown-option--active-color: #fff;
  --vs-dropdown-option--deselect-bg: #fb5858;
  --vs-dropdown-option--deselect-color: #fff;
  --vs-transition-timing-function: cubic-bezier(1, -0.115, 0.975, 0.855);
  --vs-transition-duration: 150ms;
}
.v-select {
  font-family: inherit;
  position: relative;
  *box-sizing: border-box;
  width: 200px
}
:root {
  --vs-transition-timing-function: cubic-bezier(1, 0.5, 0.8, 1);
  --vs-transition-duration: 0.15s;
}
.vs__fade-enter-active,
.vs__fade-leave-active {
  pointer-events: none;
  transition: opacity var(--vs-transition-duration) var(--vs-transition-timing-function);
}
.vs__fade-enter,
.vs__fade-leave-to {
  opacity: 0;
}
:root {
  --vs-disabled-bg: var(--vs-state-disabled-bg);
  --vs-disabled-color: var(--vs-state-disabled-color);
  --vs-disabled-cursor: var(--vs-state-disabled-cursor);
}
.vs--disabled .vs__clear,
.vs--disabled .vs__dropdown-toggle,
.vs--disabled .vs__open-indicator,
.vs--disabled .vs__search,
.vs--disabled .vs__selected {
  background-color: var(--vs-disabled-bg);
  cursor: var(--vs-disabled-cursor);
}
.v-select[dir=rtl] .vs__actions {
  padding: 0 3px 0 6px;
}
.v-select[dir=rtl] .vs__clear {
  margin-left: 6px;
  margin-right: 0;
}
.v-select[dir=rtl] .vs__deselect {
  margin-left: 0;
  margin-right: 2px;
}
.v-select[dir=rtl] .vs__dropdown-menu {
  text-align: right;
}
.vs__dropdown-toggle {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: var(--vs-search-input-bg);
  border: var(--vs-border-width) var(--vs-border-style) var(--vs-border-color);
  border-radius: var(--vs-border-radius);
  display: flex;
  padding: 0 0 4px;
  white-space: normal;
}
.vs__selected-options {
  display: flex;
  flex-basis: 100%;
  flex-grow: 1;
  flex-wrap: wrap;
  padding: 0 2px;
  position: relative;
}
.vs__actions {
  align-items: center;
  display: flex;
  padding: var(--vs-actions-padding);
}
.vs--searchable .vs__dropdown-toggle {
  cursor: text;
}
.vs--unsearchable .vs__dropdown-toggle {
  cursor: pointer;
}
.vs--open .vs__dropdown-toggle {
  border-bottom-color: transparent;
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
}
.vs__open-indicator {
  fill: var(--vs-controls-color);
  transform: scale(var(--vs-controls-size));
  transition: transform var(--vs-transition-duration) var(--vs-transition-timing-function);
  transition-timing-function: var(--vs-transition-timing-function);
}
.vs--open .vs__open-indicator {
  transform: rotate(180deg) scale(var(--vs-controls-size));
}
.vs--loading .vs__open-indicator {
  opacity: 0;
}
.vs__clear {
  fill: var(--vs-controls-color);
  background-color: transparent;
  border: 0;
  cursor: pointer;
  margin-right: 8px;
  padding: 0;
}
.vs__dropdown-menu {
  background: var(--vs-dropdown-bg);
  border: var(--vs-border-width) var(--vs-border-style) var(--vs-border-color);
  border-radius: 0 0 var(--vs-border-radius) var(--vs-border-radius);
  border-top-style: none;
  box-shadow: var(--vs-dropdown-box-shadow);
  box-sizing: border-box;
  color: var(--vs-dropdown-color);
  display: block;
  left: 0;
  list-style: none;
  margin: 0;
  max-height: var(--vs-dropdown-max-height);
  min-width: var(--vs-dropdown-min-width);
  overflow-y: auto;
  padding: 5px 0;
  position: absolute;
  text-align: left;
  top: calc(100% - var(--vs-border-width));
  width: 100%;
  z-index: var(--vs-dropdown-z-index);
}
.vs__no-options {
  text-align: center;
}
.vs__dropdown-option {
  clear: both;
  color: var(--vs-dropdown-option-color);
  cursor: pointer;
  display: block;
  line-height: 1.42857143;
  padding: var(--vs-dropdown-option-padding);
  white-space: nowrap;
}
.vs__dropdown-option--highlight {
  background: var(--vs-dropdown-option--active-bg);
  color: var(--vs-dropdown-option--active-color);
}
.vs__dropdown-option--deselect {
  background: var(--vs-dropdown-option--deselect-bg);
  color: var(--vs-dropdown-option--deselect-color);
}
.vs__dropdown-option--disabled {
  background: var(--vs-state-disabled-bg);
  color: var(--vs-state-disabled-color);
  cursor: var(--vs-state-disabled-cursor);
}
.vs__selected {
  align-items: center;
  background-color: var(--vs-selected-bg);
  border: var(--vs-selected-border-width) var(--vs-selected-border-style) var(--vs-selected-border-color);
  border-radius: var(--vs-border-radius);
  color: var(--vs-selected-color);
  display: flex;
  line-height: var(--vs-line-height);
  margin: 4px 2px 0;
  padding: 0 0.25em;
  z-index: 0;
}
.vs__deselect {
  fill: var(--vs-controls-color);
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: none;
  border: 0;
  cursor: pointer;
  display: inline-flex;
  margin-left: 4px;
  padding: 0;
  text-shadow: var(--vs-controls--deselect-text-shadow);
}
.vs--single .vs__selected {
  background-color: transparent;
  border-color: transparent;
}
.vs--single.vs--loading .vs__selected,
.vs--single.vs--open .vs__selected {
  opacity: 0.4;
  position: absolute;
}
.vs--single.vs--searching .vs__selected {
  display: none;
}
.vs__search::-webkit-search-cancel-button {
  display: none;
}
.vs__search::-ms-clear,
.vs__search::-webkit-search-decoration,
.vs__search::-webkit-search-results-button,
.vs__search::-webkit-search-results-decoration {
  display: none;
}
.vs__search,
.vs__search:focus {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: none;
  border: 1px solid transparent;
  border-left: none;
  box-shadow: none;
  color: var(--vs-search-input-color);
  flex-grow: 1;
  font-size: var(--vs-font-size);
  line-height: var(--vs-line-height);
  margin: 4px 0 0;
  max-width: 100%;
  outline: none;
  padding: 0 7px;
  width: 0;
  z-index: 1;
}
.vs__search::-moz-placeholder {
  color: var(--vs-search-input-placeholder-color);
}
.vs__search:-ms-input-placeholder {
  color: var(--vs-search-input-placeholder-color);
}
.vs__search::placeholder {
  color: var(--vs-search-input-placeholder-color);
}
.vs--unsearchable .vs__search {
  opacity: 1;
}
.vs--unsearchable:not(.vs--disabled) .vs__search {
  cursor: pointer;
}
.vs--single.vs--searching:not(.vs--open):not(.vs--loading) .vs__search {
  opacity: 0.2;
}
.vs__spinner {
  align-self: center;
  -webkit-animation: vSelectSpinner 1.1s linear infinite;
  animation: vSelectSpinner 1.1s linear infinite;
  border: 0.9em solid rgba(99,99,99,0.1);
  border-left-color: rgba(60,60,60,0.45);
  font-size: 5px;
  opacity: 0;
  overflow: hidden;
  text-indent: -9999em;
  transform: translateZ(0) scale(var(--vs-controls--spinner-size, var(--vs-controls-size)));
  transition: opacity 0.1s;
}
.vs__spinner,
.vs__spinner:after {
  border-radius: 50%;
  height: 5em;
  transform: scale(var(--vs-controls--spinner-size, var(--vs-controls-size)));
  width: 5em;
}
.vs--loading .vs__spinner {
  opacity: 1;
}
@-moz-keyframes vSelectSpinner {
  0% {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(1turn);
  }
}
@-webkit-keyframes vSelectSpinner {
  0% {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(1turn);
  }
}
@-o-keyframes vSelectSpinner {
  0% {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(1turn);
  }
}
@keyframes vSelectSpinner {
  0% {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(1turn);
  }
}
</style>
