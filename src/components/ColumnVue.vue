<template>
    <div>
        <column-vue-breadcrumbs :selected-path="selectedPath" @breadcrumbClick="handleBreadcrumbClick" />
        <div class="column-vue">
            <div class="column" v-for="(column, columnIndex) in computedColumns" :key="columnIndex"
                @drop="handleColumnDrop(columnIndex, $event)" @dragover="handleDragOver($event)">
                <ul>
                    <li v-for="(item, index) in column" :key="item.uuid" :data-uuid="item.uuid" tabindex="0"
                        :class="{ 'selected': isSelected(item) }" @click="handleItemClick(item, columnIndex)"
                        @keydown="handleKeyDown(item, index, $event)" :title="item.name" draggable="true"
                        @dragstart="handleDragStart(item, $event)" @dragover="handleDragOver($event)"
                        @drop="handleDrop(item, $event)">
                        <span class="item-text">{{ item.name }}</span>
                        <span v-if="item.children && item.children.length" class="mdi mdi-chevron-right"></span>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>




<script setup>
import { ref, computed } from 'vue';
import ColumnVueBreadcrumbs from './ColumnVueBreadcrumbs.vue';
import '@mdi/font/css/materialdesignicons.min.css';



const props = defineProps({
    equipments: Array
});

const emit = defineEmits(['select']);
const selectedPath = ref([]);
const selectedNode = ref(null);

// Existing handleDrop method - remains the same
const handleDrop = (targetItem, event) => {
    event.preventDefault();
    event.stopPropagation(); // Stop the event from propagating further

    const draggedItemData = event.dataTransfer.getData('application/json');
    const draggedItem = JSON.parse(draggedItemData);

    // Check if the dragged item is the same as the target item
    if (draggedItem.uuid === targetItem.uuid) {
        // Do nothing if the item is dropped onto itself
        return;
    }

    if (draggedItem && !isDescendantOf(targetItem, draggedItem)) {
        moveItemToNewParent(draggedItem, targetItem);

        // Update the selected node and path
        selectedNode.value = draggedItem;
        updateSelectedPathForItem(draggedItem);
    }
};




// New handleColumnDrop method
const handleColumnDrop = (columnIndex, event) => {
    event.preventDefault();
    const draggedItemData = event.dataTransfer.getData('application/json');
    const draggedItem = JSON.parse(draggedItemData);

    // Check if the drop is intended for the column (i.e., draggedItem is not null)
    if (draggedItem) {
        const parentItem = columnIndex > 0 ? selectedPath.value[columnIndex - 1] : null;

        // Move the dragged item to the new parent
        moveItemToNewParent(draggedItem, parentItem);

        // Update the selected node and path
        selectedNode.value = draggedItem;
        updateSelectedPathForItem(draggedItem);
    }
};



const handleDragStart = (item, event) => {
    event.dataTransfer.setData('application/json', JSON.stringify(item));
    // Prevent dragging onto its own children
    event.dataTransfer.effectAllowed = 'move';
};

const handleDragOver = (event) => {
    event.preventDefault();
    // Additional logic if needed
};

const isDescendantOf = (potentialDescendant, potentialAncestor) => {
    if (!potentialAncestor.children) return false;
    if (potentialAncestor.children.some(child => child.uuid === potentialDescendant.uuid)) return true;
    return potentialAncestor.children.some(child => isDescendantOf(potentialDescendant, child));
};


const moveItemToNewParent = (draggedItem, newParent) => {
    // Remove the item from its current parent
    removeItemFromParent(draggedItem);

    // If newParent is null, it means the item is moved to the root level
    if (newParent === null) {
        props.equipments.push(draggedItem);
    } else {
        // Ensure the new parent has a children array
        if (!newParent.children) {
            newParent.children = [];
        }
        newParent.children.push(draggedItem);
    }
};

const removeItemFromParent = (item) => {
    const parentItem = findParentItem(item);
    if (parentItem) {
        const index = parentItem.children.findIndex(child => child.uuid === item.uuid);
        if (index > -1) {
            parentItem.children.splice(index, 1);
        }
    } else {
        // If the item is at the root level
        const index = props.equipments.findIndex(rootItem => rootItem.uuid === item.uuid);
        if (index > -1) {
            props.equipments.splice(index, 1);
        }
    }
};

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

const getIconName = (item) => {
    switch (item.type) {
        case 'Sensor': return 'mdi-radar';
        case 'Controller': return 'mdi-gamepad-variant';
        // Add more cases as needed
        default: return 'mdi-alert-circle';
    }
};


</script>

<style scoped>
:root {
    --background-color: #f8f8f8;
    --text-color: #333;
    --border-color: #ddd;
    --selected-background-color: #095453;
    --selected-text-color: #fff;
}

@media (prefers-color-scheme: dark) {
    :root {
        --background-color: #333;
        --text-color: #fff;
        --border-color: #444;
        --selected-background-color: #26a69a;
        --selected-text-color: #fff;
    }
}

.column-vue {
    display: flex;
    background-color: var(--background-color);
}

.column {
    width: 200px;
    margin-right: 20px;
    flex-shrink: 0;
    border-right: 1px solid var(--border-color);
    background-color: var(--background-color);
    padding: 10px;
}

ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
    width: 100%;
}

li {
    margin: 5px 0;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
    align-items: center;
    overflow: hidden;
    color: var(--text-color);
}

li.selected {
    background-color: var(--selected-background-color);
    color: var(--selected-text-color);
}

.item-text {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    flex-grow: 1;
    margin-right: 10px;
}

.mdi-chevron-right {
    /* Style for the chevron icon */
}
</style>
