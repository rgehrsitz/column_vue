<template>
    <div>
        <column-vue-breadcrumbs :selected-path="selectedPath" @breadcrumbClick="handleBreadcrumbClick" />
        <div class="column-vue">
            <div class="column" v-for="(column, columnIndex) in computedColumns" :key="columnIndex">
                <ul>
                    <li v-for="item in column" :key="item.uuid" :class="{ 'selected': isSelected(item) }"
                        @click="handleItemClick(item, columnIndex)">
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
const isSelected = (item) => {
    return selectedPath.value.some(selectedItem => selectedItem.uuid === item.uuid);
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
