<template>
    <v-toolbar flat dense color="blue-grey lighten-5">
        <v-toolbar-items>
            <v-menu offset-y v-if="storages.length > 1" activator="">
                <template v-slot:activator="{ props }">
                    <v-btn icon class="storage-select-button mr-3" v-bind="props">
                        <v-icon>mdi-arrow-down-drop-circle-outline</v-icon>
                    </v-btn>
                </template>
                <v-list class="storage-select-list">
                    <v-list-item
                        v-for="(item, index) in storages"
                        :key="index"
                        :disabled="item.code === storageObject.code"
                        @click="changeStorage(item.code)"
                    >
                        <template v-slot:prepend>
                          <v-icon :icon="item.icon"></v-icon>
                        </template>
                        
                        <v-list-item-title><span>{{ item.name }}</span></v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-menu>
            <v-btn text :input-value="path === '/'" @click="changePath('/')">
                <v-icon class="mr-2" :icon="storageObject.icon"></v-icon>
                <span>{{storageObject.name}}</span>
            </v-btn>
            <template :key="index + '-vfor'" v-for="(segment, index) in pathSegments">
                <v-icon :k="index + '-icon'" icon="mdi-chevron-right"></v-icon>
                <v-btn
                    text
                    :input-value="index === pathSegments.length - 1"
                    :k="index + '-btn'"
                    @click="changePath(segment.path)"
                >{{ segment.name }}</v-btn>
            </template>
        </v-toolbar-items>
        <div class="flex-grow-1"></div>

        <template v-if="$vuetify.display.smAndUp">
            <v-tooltip bottom v-if="pathSegments.length > 0">
                <template v-slot:activator="{ props }">
                    <v-btn icon @click="goUp" v-bind="props">
                        <v-icon>mdi-arrow-up-bold-outline</v-icon>
                    </v-btn>
                </template>
                <span v-if="pathSegments.length === 1"><span>Up to "root"</span></span>
                <span v-else><span>Up to "{{pathSegments[pathSegments.length - 2].name}}"</span></span>
            </v-tooltip>
            <v-menu
                v-model="newFolderPopper"
                :close-on-content-click="false"
                :nudge-width="200"
                offset-y
            >
                <template v-slot:activator="{ props }">
                    <v-btn v-if="path" icon v-bind="props" title="Create Folder">
                        <v-icon icon="mdi-folder-plus-outline"></v-icon>
                    </v-btn>
                </template>
                <v-card>
                    <v-card-text>
                        <v-text-field label="Name" v-model="newFolderName" hide-details></v-text-field>
                    </v-card-text>
                    <v-card-actions>
                        <div class="flex-grow-1"></div>
                        <v-btn @click="newFolderPopper = false" depressed><span>Cancel</span></v-btn>
                        <v-btn
                            color="success"
                            :disabled="!newFolderName"
                            depressed
                            @click="mkdir"
                        >Create Folder</v-btn>
                    </v-card-actions>
                </v-card>
            </v-menu>
            <v-btn v-if="path" icon @click="$refs.inputUpload.click()" title="Upload Files">
                <v-icon>mdi-plus-circle</v-icon>
                <input v-show="false" ref="inputUpload" type="file" multiple @change="addFiles" />
            </v-btn>
        </template>
    </v-toolbar>
</template>

<script>
export default {
    props: {
        storages: Array,
        storage: String,
        path: String,
        endpoints: Object,
        axios: Function
    },
    data() {
        return {
            newFolderPopper: false,
            newFolderName: ""
        };
    },
    computed: {
        pathSegments() {
            let path = "/",
                isFolder = this.path[this.path.length - 1] === "/",
                segments = this.path.split("/").filter(item => item);

            segments = segments.map((item, index) => {
                path +=
                    item + (index < segments.length - 1 || isFolder ? "/" : "");
                return {
                    name: item,
                    path
                };
            });

            return segments;
        },
        storageObject() {
            return this.storages.find(item => item.code == this.storage);
        }
    },
    methods: {
        changeStorage(code) {
            if (this.storage != code) {
                this.$emit("storage-changed", code);
                this.$emit("path-changed", "");
            }
        },
        changePath(path) {
            this.$emit("path-changed", path);
        },
        goUp() {
            let segments = this.pathSegments,
                path =
                    segments.length === 1
                        ? "/"
                        : segments[segments.length - 2].path;
            this.changePath(path);
        },
        async addFiles(event) {
            this.$emit("add-files", event.target.files);
            this.$refs.inputUpload.value = "";
        },
        async mkdir() {
            this.$emit("loading", true);
            let url = this.endpoints.mkdir.url
                .replace(new RegExp("{storage}", "g"), this.storage)
                .replace(new RegExp("{path}", "g"), this.path + this.newFolderName);

            let config = {
                url,
                method: this.endpoints.mkdir.method || "post"
            };

            await this.axios.request(config);
            this.$emit("folder-created", this.newFolderName);
            this.newFolderPopper = false;
            this.newFolderName = "";
            this.$emit("loading", false);
        }
    }
};
</script>

<style lang="scss" scoped>
/* Storage Menu - alternate appearance
.storage-select-button ::v-deep(.v-btn__content) {
    flex-wrap: wrap;
    font-size: 11px;

    .v-icon {
        width: 100%;
        font-size: 22px;
    }
}
*/

.storage-select-list .v-list-item--disabled {
    background-color: rgba(0, 0, 0, 0.25);
    color: #fff;

    .v-icon {
        color: #fff;
    }
}
</style>