<template>
    <nav class="breadcrumbs">
        <ul>
            <li v-for="(item, index) in selectedPath" :key="item.uuid">
                <a href="#" @click.prevent="onBreadcrumbClick(index)">
                    {{ item.name }}
                </a>
                <span v-if="index < selectedPath.length - 1"> / </span>
            </li>
        </ul>
    </nav>
</template>

<script setup>

const props = defineProps({
    selectedPath: {
        type: Array,
        required: true
    }
});

const emit = defineEmits(['breadcrumbClick']);

const onBreadcrumbClick = (index) => {
    emit('breadcrumbClick', index);
};
</script>

<style scoped>
.breadcrumbs ul {
    list-style: none;
    padding: 0;
    display: flex;
    align-items: center;
}

.breadcrumbs li {
    margin-right: 8px;
}

.breadcrumbs a {
    color: #007bff;
    text-decoration: none;
}

.breadcrumbs a:hover {
    text-decoration: underline;
}
</style>
