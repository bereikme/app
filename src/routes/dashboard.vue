<template>
	<div class="settings">
		<v-header :breadcrumb="links" icon="home" dashboard /> 

		<v-details :title="$t('dashboard_news')" type="break" open>
			<nav>
				<ul>
					<v-card
						:title="$t('settings_global')"
						:subtitle="$tc('item_count', globalNum, { count: globalNum })"
						element="li"
						:to="`/${currentProjectKey}/settings/global`"
						icon="public"
					/>
					
					<v-card
						:title="$t('settings_default')"
						:subtitle="$tc('item_count', globalNum, { count: globalNum })"
						element="li"
						:to="`/${currentProjectKey}/settings/default`"
						icon="public"
					/>

					<v-card
						:title="$t('settings_collections_fields')"
						:subtitle="
							$tc('collection_count', collectionsNum, { count: collectionsNum })
						"
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
		
		<v-details :title="$t('dashboard_news')" type="break" open>
			<nav>
				<ul>
					<v-card
						:title="$t('dashboard_view_invoices')"
						:subtitle="$tc('item_count', globalNum, { count: globalNum })"
						element="li"
						:to="`/${currentProjectKey}/settings/global`"
						icon="public"
					/>
					
					<v-card
						:title="$t('dashboard_view_balance')"
						:subtitle="$tc('item_count', globalNum, { count: globalNum })"
						element="li"
						:to="`/${currentProjectKey}/settings/default`"
						icon="public"
					/>

					<v-card
						:title="$t('dashboard_add_funds')"
						:subtitle="
							$tc('collection_count', collectionsNum, { count: collectionsNum })
						"
						element="li"
						:to="`/${currentProjectKey}/settings/collections`"
						icon="box"
					/>
				</ul>
			</nav>
		</v-details>
	

		<v-info-sidebar wide>
			<span class="type-note">No settings</span>
		</v-info-sidebar>
	</div>
</template>

<script>
import { version } from '../../../package.json';
import VSignal from '../../components/signal.vue';
import { mapGetters, mapState } from 'vuex';

export default {
	name: 'Settings',
	metaInfo() {
		return {
			title: `${this.$t('settings')}`
		};
	},
	components: {
		VSignal
	},
	data() {
		return {
			roleCount: this.$t('loading'),
			activityCount: this.$t('loading'),
			webhookCount: this.$t('loading')
		};
	},
	computed: {
		...mapGetters(['currentProject']),
		...mapState(['currentProjectKey']),
		globalNum() {
			return Object.keys(this.$store.state.collections.directus_settings.fields).length;
		},
		collectionsNum() {
			return Object.keys(this.$store.state.collections).filter(
				name => name.startsWith('directus_') === false
			).length;
		},
		projectName() {
			return this.currentProject.project_name;
		},
		interfaceCount() {
			return Object.keys(this.$store.state.extensions.interfaces).length;
		},
		version() {
			return version;
		},
		links() {
			return [
				{
					name: this.$t('settings'),
					path: `/${this.currentProjectKey}/settings`
				}
			];
		}
	},
	created() {
		this.getRoleCount();
		this.getActivityCount();
		this.getWebhookCount();
	},
	methods: {
		getRoleCount() {
			this.$api
				.getItems('directus_roles', {
					fields: '-',
					limit: 0,
					meta: 'total_count' 
				})
				.then(res => res.meta)
				.then(({ total_count }) => {
					this.roleCount = this.$tc('role_count', total_count, {
						count: this.$n(total_count)
					});
				})
				.catch(() => {
					this.roleCount = '--';
				});
		},
		getActivityCount() {
			this.$api
				.getItems('directus_activity', {
					fields: '-',
					limit: 0,
					meta: 'total_count'
				})
				.then(res => res.meta)
				.then(({ total_count }) => {
					this.activityCount = this.$tc('event_count', total_count, {
						count: this.$n(total_count)
					});
				})
				.catch(() => {
					this.activityCount = '--';
				});
		},
		getWebhookCount() {
			this.$api
				.getItems('directus_webhooks', {
					limit: 0,
					meta: 'total_count'
				})
				.then(res => res.meta)
				.then(({ total_count }) => {
					this.webhookCount = this.$tc('webhook_count', total_count, {
						count: this.$n(total_count)
					});
				})
				.catch(() => {
					this.webhookCount = '--';
				});
		}
	}
};
</script>

<style lang="scss" scoped>
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
