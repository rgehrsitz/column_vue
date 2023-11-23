<template>
    <div>
        <column-vue-breadcrumbs :selected-path="selectedPath" @breadcrumbClick="handleBreadcrumbClick" />
        <div class="column-vue">
            <div class="column" v-for="(column, columnIndex) in computedColumns" :key="columnIndex">
                <ul>
                    <li v-for="(item, index) in column" :key="item.uuid" :data-uuid="item.uuid" tabindex="0"
                        :class="{ 'selected': isSelected(item) }" @click="handleItemClick(item, columnIndex)"
                        @keydown="handleKeyDown(item, index, $event)">
                        {{ item.name }}
                    </li>

                </ul>
            </div>
        </div>
    </div>
</template>



<script setup>
import { ref, computed, defineProps, defineEmits } from 'vue';
import ColumnVueBreadcrumbs from './ColumnVueBreadcrumbs.vue';


const props = defineProps({
    equipments: Array
});

const emit = defineEmits(['select']);
const selectedPath = ref([]);
const selectedNode = ref(null);
const isSelected = (item) => {
    return selectedPath.value.some(pathItem => pathItem.uuid === item.uuid);
};

const computedColumns = computed(() => {
    let columns = [];
    let currentItems = props.equipments;

    for (let item of selectedPath.value) {
        columns.push(currentItems);
        currentItems = item.children || [];
    }

    if (currentItems && currentItems.length > 0) {
        columns.push(currentItems);
    }

    return columns;
});

const handleItemClick = (item, level) => {

    selectedNode.value = item;
    // Truncate the selectedPath to the current level
    selectedPath.value = selectedPath.value.slice(0, level);

    // Add the new item if it's not the same as the last item in the truncated path
    if (!selectedPath.value.length || selectedPath.value[selectedPath.value.length - 1].uuid !== item.uuid) {
        selectedPath.value.push(item);
    }

    emit('select', item);
};

const handleBreadcrumbClick = (index) => {
    selectedPath.value = selectedPath.value.slice(0, index + 1);
};

const handleKeyDown = (item, index, event) => {
    let targetElement;

    switch (event.key) {
        case 'ArrowUp':
            targetElement = index > 0 ? event.target.previousElementSibling : null;
            break;
        case 'ArrowDown':
            targetElement = event.target.nextElementSibling;
            break;
        case 'ArrowLeft':
            targetElement = findCorrespondingItemInPreviousColumn(item);
            break;
        case 'ArrowRight':
            targetElement = findFirstItemInNextColumn(item);
            break;
        default:
            break;
    }

    if (targetElement) {
        const uuid = targetElement.getAttribute('data-uuid');
        const newItem = findItemByUuid(uuid, props.equipments);

        if (newItem) {
            selectedNode.value = newItem;

            // Update selectedPath for keyboard navigation
            updateSelectedPathForItem(newItem);

            targetElement.focus();
            event.preventDefault();
        }
    }
};

const findCorrespondingItemInPreviousColumn = (currentItem) => {
    if (selectedPath.value.length < 2) {
        // No previous column available
        return null;
    }

    const parentIndex = selectedPath.value.length - 2;
    const parentItem = selectedPath.value[parentIndex];
    return document.querySelector(`.column:nth-child(${parentIndex + 1}) li[data-uuid="${parentItem.uuid}"]`);
};


const findFirstItemInNextColumn = (currentItem) => {
    if (!currentItem.children || currentItem.children.length === 0) {
        // No next column available
        return null;
    }

    const nextColumnIndex = selectedPath.value.length + 1;
    const firstChild = currentItem.children[0];
    return document.querySelector(`.column:nth-child(${nextColumnIndex}) li[data-uuid="${firstChild.uuid}"]`);
};



const findItemByUuid = (uuid, items) => {
    for (const item of items) {
        if (item.uuid === uuid) {
            return item;
        }
        if (item.children && item.children.length > 0) {
            const found = findItemByUuid(uuid, item.children);
            if (found) {
                return found;
            }
        }
    }
    return null;
};

const updateSelectedPathForItem = (item) => {
    // Reset selectedPath and rebuild it based on the new item
    selectedPath.value = [];
    let currentItem = item;
    while (currentItem) {
        selectedPath.value.unshift(currentItem);
        currentItem = findParentItem(currentItem);
    }
};

const findParentItem = (childItem, items = props.equipments, parent = null) => {
    for (const item of items) {
        if (item.uuid === childItem.uuid) {
            return parent;
        }
        if (item.children && item.children.length > 0) {
            const foundParent = findParentItem(childItem, item.children, item);
            if (foundParent) {
                return foundParent;
            }
        }
    }
    return null;
};


</script>

<style scoped>
.selected {
    background-color: rgb(9, 84, 79);
    /* or any other style to indicate selection */
}

.column-vue {
    display: flex;
}

.column {
    margin-right: 20px;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    margin: 5px 0;
    cursor: pointer;
}
</style>
