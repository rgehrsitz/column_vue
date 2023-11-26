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
                        @drop="handleDrop(item, $event)" @contextmenu="handleContextMenu($event, item)">
                        <span class="item-text">{{ item.name }}</span>
                        <span v-if="item.children && item.children.length" class="mdi mdi-chevron-right"></span>
                    </li>
                </ul>
            </div>
        </div>
    </div>
    <div v-if="showContextMenu" :style="{ top: contextMenuPosition.y + 'px', left: contextMenuPosition.x + 'px' }"
        class="context-menu" @click.stop>
        <ul>
            <!-- Use `clipboard` to determine if 'Paste' should be disabled -->
            <li :class="{ disabled: !isCutEnabled }" @click="cutItem">Cut</li>
            <li :class="{ disabled: !isCopyEnabled }" @click="copyItem">Copy</li>
            <li :class="{ disabled: !isPasteEnabled }" @click="pasteItem">Paste</li>





        </ul>
    </div>
</template>




<script setup>
import { ref, computed, watch } from 'vue';
import ColumnVueBreadcrumbs from './ColumnVueBreadcrumbs.vue';
import '@mdi/font/css/materialdesignicons.min.css';
import { v4 as uuidv4 } from 'uuid';

const clipboard = ref(null);

const isPasteEnabled = computed(() => clipboard.value !== null);
const isCutEnabled = computed(() => selectedNode.value !== null);
const isCopyEnabled = computed(() => selectedNode.value !== null);


const showContextMenu = ref(false);
const contextMenuItem = ref(null);
const contextMenuPosition = ref({ x: 0, y: 0 });


const generateNewUuidsForSubtree = (node) => {
    const newNode = { ...node, uuid: uuidv4() }; // Assign a new UUID
    if (newNode.children && newNode.children.length > 0) {
        newNode.children = newNode.children.map(generateNewUuidsForSubtree); // Recursively update children
    }
    return newNode;
};

const handleContextMenu = (event, item) => {
    event.preventDefault();
    console.log('Context menu triggered for:', item);
    showContextMenu.value = true;
    contextMenuItem.value = item; // Store the item for which the context menu is opened
    selectedNode.value = item; // Set the clicked item as selected
    contextMenuPosition.value = { x: event.clientX, y: event.clientY };
    console.log('Right-clicked item:', item); // Log the right-clicked item
    console.log('Selected node after right-click:', selectedNode.value);
    window.addEventListener('click', closeContextMenu);
};

watch(showContextMenu, (newValue) => {
    if (!newValue) {
        // Remove the listener when the context menu is not shown
        window.removeEventListener('click', closeContextMenu);
    }
});

const closeContextMenu = () => {
    showContextMenu.value = false;
};

const cutItem = () => {
    if (!selectedNode.value) {
        console.log('Cut operation is not allowed');
        return; // Exit if no node is selected
    }
    console.log('Cutting item:', selectedNode.value);
    clipboard.value = { ...selectedNode.value }; // Clone the selected node
    removeItemFromParent(selectedNode.value); // Remove from current parent
    removeFromSelectedPath(selectedNode.value); // New function to update selectedPath
    closeContextMenu();
    selectedNode.value = null; // Deselect node after cutting
};

const copyItem = () => {
    if (!selectedNode.value) {
        console.log('Copy operation is not allowed');
        return; // Exit if no node is selected
    }
    console.log('Copying item:', selectedNode.value);
    clipboard.value = generateNewUuidsForSubtree(JSON.parse(JSON.stringify(selectedNode.value))); // Deep clone the selected node with new UUIDs
    closeContextMenu();
};


const pasteItem = () => {
    if (!clipboard.value || !selectedNode.value) {
        console.log('Paste operation is not allowed');
        return; // Exit if clipboard is empty or no node is selected
    }
    console.log('Pasting item:', clipboard.value);

    // Rest of the paste logic
    const clone = JSON.parse(JSON.stringify(clipboard.value)); // Deep clone the clipboard item
    if (!selectedNode.value.children) {
        selectedNode.value.children = []; // If selected node has no children array, create it
    }
    selectedNode.value.children.push(clone); // Add clone to children of selected node
    closeContextMenu();

    // Clear the clipboard after pasting
    clipboard.value = null;
};

const removeFromSelectedPath = (item) => {
    const index = selectedPath.value.findIndex(pathItem => pathItem.uuid === item.uuid);
    if (index !== -1) {
        // Remove the item and any of its descendants from the selected path
        selectedPath.value.splice(index);
    }
};


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
    console.log('Removing item from parent:', item);
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

    // New logic for Ctrl+C and Ctrl+V
    if (event.ctrlKey || event.metaKey) { // Check if Ctrl or Command (for Mac) is pressed
        switch (event.key) {
            case 'c': // Ctrl+C
                if (selectedNode.value) {
                    clipboard = { ...selectedNode.value };
                }
                break;
            case 'v': // Ctrl+V
                if (clipboard && selectedNode.value) {
                    // Implement logic to paste the clipboard content as a child of the selected node
                    // You might need to update the data structure and ensure unique identifiers
                }
                break;
        }
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

/* Add styles for your context menu */
.context-menu {
    position: absolute;
    border: 1px solid var(--border-color);
    background-color: var(--background-color);
    z-index: 1000;
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

.context-menu ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.context-menu ul li {
    padding: 8px 12px;
    cursor: pointer;
    color: var(--text-color);
}

.context-menu ul li:hover {
    background-color: var(--selected-background-color);
}

.context-menu ul li.disabled {
    color: grey;
    pointer-events: none;
}
</style>
