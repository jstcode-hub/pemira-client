<template>
    <aside class="sticky top-0 h-screen border-r">
        <ResponsiveSpace />
        <div class="h-20 px-4 border-b border-dashed">
            <h2 class="text-3xl">PEMIRA</h2>
            <div class="text-sm font-semibold">E-Vote HIMATIFA</div>
        </div>
        <div class="p-4">
            <ul class="grid gap-2">
                <slot />
            </ul>
        </div>
    </aside>
</template>
