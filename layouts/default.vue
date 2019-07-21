<template lang="pug">
v-app(dark)
	v-toolbar(app)
		v-toolbar-title(class="white--text") ML5 Fast Style Transfer
		v-spacer
	v-content(app)
		nuxt
	v-footer(
		app
	)
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
	data() {
		return {
			clipped: false,
			items: [
				{
					icon: 'apps',
					title: 'Welcome',
					disabled: false,
					to: '/'
				}
			],
			miniVariant: false,
			title: 'Vuetify.js'
		}
	}
}
</script>
