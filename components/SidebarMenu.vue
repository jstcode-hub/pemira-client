<script lang="ts" setup>
const props = defineProps<{
    to: string;
}>();

const route = useRoute();
const active = computed(() => route.path.startsWith(props.to));
</script>

<template>
    <li>
        <NuxtLink
            :to="props.to"
            :class="{
                'bg-muted text-white': active,
                'text-muted-foreground': !active,
            }"
            class="flex px-4 py-2 items-center gap-2 text-sm font-medium rounded-lg cursor-pointer transition hover:bg-muted"
        >
            <slot />
        </NuxtLink>
    </li>
</template>
