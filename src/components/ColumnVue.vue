<template>
    <div class="column-vue">
        <ul>
            <li v-for="item in equipments" :key="item.uuid" @click="selectItem(item)">
                <span :class="`type-icon type-${item.type}`">{{ item.name }}</span>
                <column-vue v-if="item.children && item.children.length" :equipments="item.children" @select="selectItem" />
            </li>
        </ul>
    </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
    equipments: Array
});

const emit = defineEmits(['select']);

const selectItem = (item) => {
    emit('select', item);
};

</script>

<style scoped>
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

.type-icon {
    /* Style for the icon */
}

.type-A {
    /* Example type */
    /* Specific style for type A */
}

.type-B {
    /* Another example type */
    /* Specific style for type B */
}
</style>
