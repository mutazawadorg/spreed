<!--
  - @copyright Copyright (c) 2022 Marco Ambrosini <marcoambrosini@icloud.com>
  -
  - @author Marco Ambrosini <marcoambrosini@icloud.com>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
-->

<template>
	<div v-if="!loading && active">
		<template v-for="type in sharedItemsOrder">
			<div v-if="sharedItems[type]" :key="type">
				<NcAppNavigationCaption :title="getTitle(type)" />
				<SharedItems :type="type"
					:limit="limit(type)"
					:items="sharedItems[type]" />
				<NcButton v-if="hasMore(type, sharedItems[type])"
					type="tertiary-no-background"
					class="more"
					:wide="true"
					@click="showMore(type)">
					<template #icon>
						<DotsHorizontal :size="20" />
					</template>
					{{ getButtonTitle(type) }}
				</NcButton>
			</div>
		</template>
		<NcRelatedResourcesPanel class="related-resources"
			provider-id="talk"
			:item-id="conversation.token"
			@has-resources="value => hasRelatedResources = value" />
		<template v-if="projectsEnabled">
			<NcAppNavigationCaption :title="t('spreed', 'Projects')" />
			<CollectionList v-if="getUserId && token"
				:id="token"
				type="room"
				:name="conversation.displayName"
				:is-active="active" />
		</template>
		<NcEmptyContent v-else-if="!hasSharedItems && !hasRelatedResources"
			:title="t('spreed', 'No shared items')">
			<template #icon>
				<FolderMultipleImage :size="20" />
			</template>
		</NcEmptyContent>
		<SharedItemsBrowser v-if="showSharedItemsBrowser"
			:shared-items="sharedItems"
			:active-tab.sync="browserActiveTab"
			@close="showSharedItemsBrowser = false" />
	</div>
</template>

<script>
import { CollectionList } from 'nextcloud-vue-collections'
import { loadState } from '@nextcloud/initial-state'

import SharedItems from './SharedItems.vue'
import { SHARED_ITEM } from '../../../constants.js'
import NcAppNavigationCaption from '@nextcloud/vue/dist/Components/NcAppNavigationCaption.js'
import NcRelatedResourcesPanel from '@nextcloud/vue/dist/Components/NcRelatedResourcesPanel.js'
import NcEmptyContent from '@nextcloud/vue/dist/Components/NcEmptyContent.js'
import SharedItemsBrowser from './SharedItemsBrowser/SharedItemsBrowser.vue'
import DotsHorizontal from 'vue-material-design-icons/DotsHorizontal.vue'
import FolderMultipleImage from 'vue-material-design-icons/FolderMultipleImage.vue'
import NcButton from '@nextcloud/vue/dist/Components/NcButton.js'
import sharedItems from '../../../mixins/sharedItems.js'

export default {

	name: 'SharedItemsTab',

	components: {
		SharedItems,
		CollectionList,
		NcAppNavigationCaption,
		NcRelatedResourcesPanel,
		NcButton,
		NcEmptyContent,
		SharedItemsBrowser,
		DotsHorizontal,
		FolderMultipleImage,
	},

	mixins: [sharedItems],

	props: {

		active: {
			type: Boolean,
			required: true,
		},
	},

	data() {
		return {
			showSharedItemsBrowser: false,
			browserActiveTab: '',
			projectsEnabled: loadState('core', 'projects_enabled', false),
			hasRelatedResources: false,
		}
	},

	computed: {
		getUserId() {
			return this.$store.getters.getUserId()
		},

		token() {
			return this.$store.getters.getToken()
		},

		conversation() {
			return this.$store.getters.conversation(this.token)
		},

		loading() {
			return !this.$store.getters.isOverviewLoaded(this.token)
		},

		sharedItems() {
			return this.$store.getters.sharedItems(this.token)
		},

		hasSharedItems() {
			return Object.keys(this.$store.getters.sharedItems(this.token)).length > 0
		},
	},

	watch: {
		active(newValue) {
			if (newValue) {
				this.getSharedItemsOverview()
			}
		},
	},

	methods: {
		getSharedItemsOverview() {
			this.$store.dispatch('getSharedItemsOverview', { token: this.token })
		},

		hasMore(type, items) {
			return Object.values(items).length > this.limit(type)
		},

		showMore(type) {
			this.browserActiveTab = type
			this.showSharedItemsBrowser = true
		},

		limit(type) {
			if (type === SHARED_ITEM.TYPES.DECK_CARD || type === SHARED_ITEM.TYPES.LOCATION || type === SHARED_ITEM.TYPES.POLL) {
				return 2
			} else {
				return 6
			}
		},

		getButtonTitle(type) {
			switch (type) {
			case SHARED_ITEM.TYPES.MEDIA:
				return t('spreed', 'Show all media')
			case SHARED_ITEM.TYPES.FILE:
				return t('spreed', 'Show all files')
			case SHARED_ITEM.TYPES.POLL:
				return t('spreed', 'Show all polls')
			case SHARED_ITEM.TYPES.DECK_CARD:
				return t('spreed', 'Show all deck cards')
			case SHARED_ITEM.TYPES.VOICE:
				return t('spreed', 'Show all voice messages')
			case SHARED_ITEM.TYPES.LOCATION:
				return t('spreed', 'Show all locations')
			case SHARED_ITEM.TYPES.AUDIO:
				return t('spreed', 'Show all audio')
			case SHARED_ITEM.TYPES.OTHER:
			default:
				return t('spreed', 'Show all other')
			}
		},
	},
}
</script>

<style lang="scss" scoped>
@use 'sass:math';
@import '../../../assets/variables';

.more {
	margin-top: 8px;
}

// Override default NcRelatedResourcesPanel styles
.related-resources {
	&::v-deep .related-resources__header {
		margin: 14px 0 !important;
		padding: 0 8px 0 math.div($clickable-area, 2) !important;
		h5 {
			opacity: .7 !important;
			color: var(--color-primary-element) !important;
		}
	}
}
</style>
