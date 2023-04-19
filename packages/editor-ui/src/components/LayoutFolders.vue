<template>
	<n8n-menu :items="folderStructure" mode="tabs" noDefaultTab @input="onSelectFolder" />
</template>

<script lang="ts">
import Vue from 'vue';
import { mapStores } from 'pinia';
import { useUIStore } from '@/stores/ui';
import { useTagsStore } from '@/stores/tags';

interface Folder {
	name: String;
	root: Boolean;
	subLevel: Number;
	children?: Folder[];
	id: String;
	icon: String;
	customIconSize: String;
	label: String;
	position: String;
	className: String;
}

export default Vue.extend({
	computed: {
		...mapStores(useTagsStore, useUIStore),
		allTags(): String[] {
			return this.tagsStore.allTags.map((tag) => tag.name);
		},
		subTags(): Object[] {
			return this.allTags.map((tag, idx) => {
				return {
					key: idx,
					tagsArray: tag.split(' > '),
				};
			});
		},
		folderStructure(): Folder[] {
			const folders = [];
			this.createFolderStructure(folders, this.subTags, 0);
			folders.push({
				name: 'Unassigned',
				root: true,
				subLevel: 0,
				children: [],
				id: 'Unassigned_' + folders.length + 1,
				icon: 'folder',
				customIconSize: 'large',
				label: 'Unassigned',
				position: 'top',
				className: '',
			});

			return folders.sort((a, b) => a.name.localeCompare(b.name));
		},
	},
	emits: {
		click: null,
	},
	methods: {
		onSelectFolder(type: string) {
			if (!type) {
				return;
			}

			const idx = Number(type.split('_')[1]);
			const clickedTag = this.subTags[idx]?.tagsArray.join(' > ');
			const tagObj = this.tagsStore.allTags.find((tag) => tag.name === clickedTag);
			if (tagObj) {
				this.$emit('click', tagObj.id);
			} else {
				this.$emit('click', null);
			}
		},
		createFolderStructure(folders, subTags, level) {
			subTags.forEach(({ key, tagsArray }) => {
				const [name, ...subFolders] = tagsArray;
				const existingFolder = folders.find(
					(folder) => folder.name === name && folder.root === (level === 0),
				);
				if (existingFolder) {
					if (subFolders.length) {
						this.createFolderStructure(
							existingFolder.children,
							[{ key, tagsArray: subFolders }],
							level + 1,
						);
					}
				} else {
					const newFolder = {
						name,
						root: level === 0,
						subLevel: level,
						children: [],
						id: name + '_' + key,
						icon: level === 0 ? 'folder' : 'folder-open',
						customIconSize: level === 0 ? 'large' : level === 1 ? 'medium' : 'small',
						label: name,
						position: 'top',
						className: level === 0 ? '' : level === 1 ? 'ml-xs' : 'ml-m',
					};
					folders.push(newFolder);
					if (subFolders.length) {
						this.createFolderStructure(
							newFolder.children,
							[{ key, tagsArray: subFolders }],
							level + 1,
						);
					}
				}
			});
		},
	},
});
</script>

<style lang="scss" scoped>
.menu-container {
	--menu-background: transparent;
	--menu-padding: 0;
	margin-top: 12px;
}
</style>
