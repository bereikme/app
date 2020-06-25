<template>
	<div class="settings">
		<v-header :breadcrumb="links" :icon-link="`/${currentProjectKey}/settings`" settings>
		</v-header>
		
		<v-details :title="$t('settings_project')" type="break" open>
			<nav>
				<ul>
					<v-card
						:title="$t('dashboard_how_to_start')"
						element="li"
						:to="`/${currentProjectKey}/settings/global`"
						icon="public"
					/>
					
					<v-card
						:title="$t('dashboard_how_to_start')"
					element="li"
						:to="`/${currentProjectKey}/settings/default`"
						icon="public"
					/>

					<v-card
						:title="$t('settings_collections_fields')"
					
						element="li"
						:to="`/${currentProjectKey}/settings/collections`"
						icon="box"
					/>

					<v-card
						:title="$t('settings_permissions')"
						:subtitle="roleCount"
						element="li"
						:to="`/${currentProjectKey}/settings/roles`"
						icon="group"
					/>

					<v-card
						:title="$t('settings_webhooks')"
						:subtitle="webhookCount"
						element="li"
						:to="`/${currentProjectKey}/settings/webhooks`"
						icon="send"
					/>
				</ul>
			</nav>
		</v-details>
		
		<v-details :title="$t('settings_project')" type="break" open>
			<nav>
				<ul>
					<v-card
						:title="$t('dashboard_how_to_start')"
						element="li"
						:to="`/${currentProjectKey}/settings/global`"
						icon="public"
					/>
					
					<v-card
						:title="$t('dashboard_how_to_start')"
					element="li"
						:to="`/${currentProjectKey}/settings/default`"
						icon="public"
					/>

					<v-card
						:title="$t('settings_collections_fields')"
					
						element="li"
						:to="`/${currentProjectKey}/settings/collections`"
						icon="box"
					/>

					<v-card
						:title="$t('settings_permissions')"
						:subtitle="roleCount"
						element="li"
						:to="`/${currentProjectKey}/settings/roles`"
						icon="group"
					/>

					<v-card
						:title="$t('settings_webhooks')"
						:subtitle="webhookCount"
						element="li"
						:to="`/${currentProjectKey}/settings/webhooks`"
						icon="send"
					/>
				</ul>
			</nav>
		</v-details>

		<v-form
			:fields="fields"
			:values="values"
			collection="directus_settings"
			@stage-value="stageValue"
		/>
		<v-info-sidebar wide>
			<span class="type-note">No settings</span>
		</v-info-sidebar>
	</div>
</template>

<script>
import { mapState } from 'vuex';

export default {
	name: 'SettingsGlobal',
	metaInfo() {
		return {
			title: `${this.$t('settings')} | ${this.$t('settings_global')}`
		};
	},
	data() {
		return {
			saving: false,
			edits: {}
		};
	},
	computed: {
		...mapState({
			settings: state => state.settings.values,
			fields: state => state.collections.directus_settings.fields,
			currentProjectKey: state => state.currentProjectKey
		}),
		values() {
			return {
				...this.settings,
				...this.edits
			};
		},
		links() {
			return [
				{
					name: this.$t('settings'),
					path: `/${this.currentProjectKey}/settings`
				},
				{
					name: this.$t('settings_global'),
					path: `/${this.currentProjectKey}/settings/global`
				}
			];
		},
		editing() {
			return Object.keys(this.edits).length > 0;
		}
	},
	methods: {
		stageValue({ field, value }) {
			if (this.settings[field] == value) {
				return this.$delete(this.edits, field);
			}

			return this.$set(this.edits, field, value);
		},
		save() {
			this.saving = true;

			this.$store
				.dispatch('setSettings', this.edits)
				.then(() => {
					this.saving = false;
					this.edits = {};

					// Update the current project's info in the store when saving settings, this makes sure
					// the logo / name in the top left etc will update
					this.$store.dispatch('updateProjectInfo', this.currentProjectKey);

					this.$router.push(`/${this.currentProjectKey}/settings`);
					this.$notify({
						title: this.$t('settings_saved'),
						color: 'green',
						iconMain: 'check'
					});
				})
				.catch(error => {
					this.saving = false;
					this.$events.emit('error', {
						notify: error.message || this.$t('something_went_wrong_body'),
						error
					});
				});
		}
	}
};
</script>

<style lang="scss" scoped>
.settings-global {
	padding: var(--page-padding-top) var(--page-padding) var(--page-padding-bottom);
}

.settings {
	padding: var(--page-padding-top) var(--page-padding) var(--page-padding-bottom);
}

nav ul {
	padding: 0;
	display: grid;
	grid-template-columns: repeat(auto-fill, var(--card-size));
	grid-gap: var(--card-horizontal-gap);

	li {
		display: block;
	}
}

.signal {
	fill: var(--white);
}

</style>
