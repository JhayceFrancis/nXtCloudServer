<!--
  - SPDX-FileCopyrightText: 2023 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->
<template>
	<tr>
		<th class="files-list__row-checkbox">
			<!-- TRANSLATORS Label for a table footer which summarizes the columns of the table -->
			<span class="hidden-visually">{{ t('files', 'Total rows summary') }}</span>
		</th>

		<!-- Link to file -->
		<td class="files-list__row-name">
			<!-- Icon or preview -->
			<span class="files-list__row-icon" />

			<!-- Summary -->
			<span>{{ summary }}</span>
		</td>

		<!-- Actions -->
		<td class="files-list__row-actions" />

		<!-- Size -->
		<td v-if="isSizeAvailable"
			class="files-list__column files-list__row-size">
			<span>{{ totalSize }}</span>
		</td>

		<!-- Mtime -->
		<td v-if="isMtimeAvailable"
			class="files-list__column files-list__row-mtime" />

		<!-- Custom views columns -->
		<th v-for="column in columns"
			:key="column.id"
			:class="classForColumn(column)">
			<span>{{ column.summary?.(nodes, currentView) }}</span>
		</th>
	</tr>
</template>

<script lang="ts">
import { formatFileSize } from '@nextcloud/files'
import { translate } from '@nextcloud/l10n'
import Vue from 'vue'

import { useFilesStore } from '../store/files.ts'
import { usePathsStore } from '../store/paths.ts'

export default Vue.extend({
	name: 'FilesListTableFooter',

	components: {
	},

	props: {
		isMtimeAvailable: {
			type: Boolean,
			default: false,
		},
		isSizeAvailable: {
			type: Boolean,
			default: false,
		},
		nodes: {
			type: Array,
			required: true,
		},
		summary: {
			type: String,
			default: '',
		},
		filesListWidth: {
			type: Number,
			default: 0,
		},
	},

	setup() {
		const pathsStore = usePathsStore()
		const filesStore = useFilesStore()
		return {
			filesStore,
			pathsStore,
		}
	},

	computed: {
		currentView() {
			return this.$navigation.active
		},

		dir() {
			// Remove any trailing slash but leave root slash
			return (this.$route?.query?.dir || '/').replace(/^(.+)\/$/, '$1')
		},

		currentFolder() {
			if (!this.currentView?.id) {
				return
			}

			if (this.dir === '/') {
				return this.filesStore.getRoot(this.currentView.id)
			}
			const fileId = this.pathsStore.getPath(this.currentView.id, this.dir)
			return this.filesStore.getNode(fileId)
		},

		columns() {
			// Hide columns if the list is too small
			if (this.filesListWidth < 512) {
				return []
			}
			return this.currentView?.columns || []
		},

		totalSize() {
			// If we have the size already, let's use it
			if (this.currentFolder?.size) {
				return formatFileSize(this.currentFolder.size, true)
			}

			// Otherwise let's compute it
			return formatFileSize(this.nodes.reduce((total, node) => total + node.size || 0, 0), true)
		},
	},

	methods: {
		classForColumn(column) {
			return {
				'files-list__row-column-custom': true,
				[`files-list__row-${this.currentView.id}-${column.id}`]: true,
			}
		},

		t: translate,
	},
})
</script>

<style scoped lang="scss">
// Scoped row
tr {
	margin-bottom: 300px;
	border-top: 1px solid var(--color-border);
	// Prevent hover effect on the whole row
	background-color: transparent !important;
	border-bottom: none !important;

	td {
		user-select: none;
		// Make sure the cell colors don't apply to column headers
		color: var(--color-text-maxcontrast) !important;
	}
}
</style>
