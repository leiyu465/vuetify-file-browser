<template>
    <v-list-group :value="item.basename">
        <template v-slot:activator="{ props }" >
            <v-list-item
                v-bind="props"
                prepend-icon="open ? 'mdi-folder-open-outline' : 'mdi-folder-outline'"
                :title="item.basename"
            ></v-list-item>
        </template>
        <template v-if="item.children && item.children.length">
            <MyTreeview :key="i" :item="item" v-for="(item, i) in item.children"  />
        </template>
    </v-list-group>
</template>

<script>
const MyTreeview = {
    name: "MyTreeview",
    props: {
        item: Object,
    },
    data() {
        return {
           
        };
    },
    methods: {
        
    },
    watch: {
        
    },
    created() {
        // this.init();
    }
};
export default MyTreeview
</script>
