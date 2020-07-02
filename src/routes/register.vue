<template>
	<PublicView wide :heading="$t('create_new_account')">
		<public-stepper class="stepper" :steps="3" :current-step="step" />

		<div v-show="step === 1" class="step-1">
			<template v-if="firstInstall">
				<div class="field-grid">
					<div class="field">
						<h2 class="type-title">{{ $t('create_new_account') }}</h2>
						<p>{{ $t('create_new_account_decription') }}</p>
					</div>
				</div>
				<button type="button" @click="step = 2">{{ $t('next') }}</button>
			</template>
			<template v-else class="field-grid">
				<div class="field-grid">
					<div class="field">
						<h2 class="type-title">{{ $t('create_new_account') }}</h2>
						<p>{{ $t('create_new_account_decription') }}</p>
						<input
							v-model="super_admin_token"
							v-focus
							placeholder="Super-Admin Password..."
							type="checkbox"
						/>
					</div>
				</div>
				<button type="button" @click="step = 2">{{ $t('i_have_read_and_agree') }}</button>
			</template>
		</div>

		<form v-show="step === 2" class="step-2" @submit.prevent="onSubmit(); step = 3;">
			<fieldset>
				<legend class="type-title">{{ $t('fill_in_your_personal_details') }}</legend>
				<div class="field-grid">
					<div class="field">
						<label class="type-label" for="first_name">
							{{ $t('first_name') }}
						</label>
						<input
							id="first_name"
							v-model="first_name"
							v-focus
							name="first_name"
							type="text"
							required
						/>
					</div>
					<div class="field">
						<label class="type-label" for="project">{{ $t('last_name') }}</label>
						<input
							id="last_name"
							v-model="last_name"	
							v-focus							
							name="last_name"
							type="text"
							required
						/>
					</div>
					<div class="field">
						<label class="type-label" for="user_email">{{ $t('user_email') }}</label>
						<input
							id="user_email"
							v-model="user_email"
							name="user_email"
							type="email"
							required
						/>
					</div>
					<div class="field">
						<label class="type-label" for="user_password">
							{{ $t('user_password') }}
						</label>
						<input
							id="user_password"
							v-model="user_password"
							class="password"
							name="user_password"
							type="password" 
							tooltip="this is a tooltip"
							required
						/>
					</div>
				</div>

				<div class="buttons">
					<span class="secondary" @click="step--">{{ $t('back') }}</span>
					<button type="submit">{{ $t('next') }}</button>
				</div>
			</fieldset>
		</form>

		<div v-show="step === 3" class="step-3">
			<h2 class="type-title">{{ $t('all_set') }}</h2>
			<div class="field-grid">
				<div class="field">
					<v-progress-linear class="progress-bar" :value="100" rounded />
					<p>
						{{ $t('account_created') }}
					</p>
					<button type="button" class="button" @click="goToLogin">
						{{ $t('sign_in') }}
					</button>
				</div>
			</div>
		</div>

		<public-notice
			v-if="notice.text"
			slot="notice"
			:loading="false"
			:color="notice.color"
			:icon="notice.icon"
		>
			{{ notice.text }}
		</public-notice>
	</PublicView>
</template>

<script>
import PublicView from '@/components/public-view';
import PublicNotice from '@/components/public/notice';
import axios from 'axios';
import { mapState, mapActions } from 'vuex';
import PublicStepper from '@/components/public/stepper';
import slug from 'slug';
import shortid from 'shortid';
//import InstallRequirements from '@/components/install/requirements';

export default {
	name: 'Install',
	components: {
		PublicView,
		PublicNotice,
		PublicStepper
	},
	data() {
		return {
			step: 1,
			first_name: 'enter your first name',
			last_name: 'enter your last name',
			user_email: 'john.doe@acme.inc',
			user_password: '',
			notice: {
				text: this.$t('already_have_an_account'),
				color: 'blue-grey-100',
				icon: 'outlined_flag'
			},
		};
	},
	methods: {
		async onSubmit() {
			//alert("submitting...")
			
			const {
				first_name,
				last_name,
				user_email,
				user_password
			} = this;
     
			try {
				await axios({
				  url: `https://worksdomain.nl/put_users.php?first_name=${first_name}&last_name=${last_name}&email=${user_email}&password=${user_password}`,
				  method: 'post'
				})
				
			} catch (error) {
				this.error = error;

				console.error(error);

				this.$events.emit('error', {
					notify: error.response?.data?.error?.message,
					error
				});

				this.step = 3;
			}
		},
		async goToLogin() {
			this.$router.push('/login');
		}
	}
};
</script>

<style lang="scss" scoped>
// NOTE: These button and input styles are copied from login.vue and should be extracted to a base component

.button,
button {
	position: relative;
	background-color: var(--button-primary-background-color);
	border: 2px solid var(--button-primary-background-color);
	border-radius: var(--border-radius);
	color: var(--button-primary-text-color);
	height: 60px;
	padding: 18px 20px;
	width: 100%;
	max-width: 154px;
	font-size: 16px;
	font-weight: 400;
	transition: background-color var(--fast) var(--transition);
	display: inline-block;
	text-decoration: none;
	text-align: center;

	&[disabled] {
		cursor: not-allowed;
	}

	&:not([disabled]) {
		&:hover {
			background-color: var(--darkest-gray);
			border-color: var(--darkest-gray);
		}
	}

	&.outline {
		background-color: transparent;
		color: var(--darkest-gray);

		&[disabled] {
			background-color: transparent;
		}

		&:not([disabled]) {
			&:hover {
				background-color: transparent;
			}
		}
	}
}

.select {
	position: relative;
	&:after {
		content: 'arrow_drop_down';
		font-family: var(--family-icon);
		position: absolute;
		right: 12px;
		top: 8px;
		color: var(--input-icon-color);
	}
}

input {
	position: relative;
	width: 100%;
	margin-bottom: 32px;
	border: 0;
	font-size: 16px;
	border: var(--input-border-width) solid var(--input-border-color);
	width: 100%;
	height: 64px;
	padding: 20px 10px;
	background-color: var(--input-background-color);
	color: var(--darker-gray);
	transition: border-color var(--fast) var(--transition);
	border-radius: var(--border-radius);
	font-family: var(--family-monospace);

	&::placeholder {
		color: var(--light-gray);
	}

	&:-webkit-autofill {
		color: var(--darker-gray) !important;
		-webkit-text-fill-color: var(--darker-gray);
		-webkit-box-shadow: 0 0 0px 1000px var(--white) inset;
	}

	&:hover:not([disabled]) {
		transition: none;
		border-color: var(--gray);
		&:focus {
			border-color: var(--darker-gray);
		}
	}

	&[disabled] {
		cursor: not-allowed;
		background-color: var(--input-background-color-disabled);
	}

	&:focus {
		outline: 0;
		border-color: var(--darker-gray);

		&:-webkit-autofill {
			color: var(--darker-gray) !important;
			-webkit-text-fill-color: var(--darker-gray);
			-webkit-box-shadow: 0 0 0px 1000px var(--white) inset;
		}
	}
}

///////////////////////////////////

.buttons {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-top: 16px;
	.secondary {
		transition: color var(--fast) var(--transition);
		text-decoration: none;
		color: var(--input-placeholder-color);
		cursor: pointer;
		font-size: 16px;
		&:hover {
			color: var(--page-text-color);
		}
	}
}

p {
	font-size: 16px;
	line-height: 26px;
	margin-top: 32px;
	margin-bottom: 32px;
	color: var(--blue-grey-300);
}

.stepper {
	margin-top: 4px;

	@media (min-height: 800px) {
		margin-top: 24px;
	}
}

legend {
	margin-bottom: 20px;
}

.stepper {
	margin-bottom: 64px;
	max-width: 320px;
}

.progress-bar {
	margin-top: 32px;
}

.progress-bar-complete {
	margin-top: 32px;
	width: 100%;
	background-color: var(--progress-background-color-accent);
	position: relative;
	height: 4px;
	border-radius: var(--border-radius);
}

.warning {
	color: var(--warning);
}

.field-grid {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	grid-gap: 8px 32px;
}

label {
	margin-bottom: 8px;
}

.password {
	-moz-text-security: disc;
	-webkit-text-security: disc;
	text-security: disc;
}
</style>