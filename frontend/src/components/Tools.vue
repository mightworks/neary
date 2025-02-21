<template>
    <div id="alt-window" class="font-mulish flex flex-col gap-3 w-full overflow-y-scroll">
        <div class="p-8 pt-[5.5rem] max-w-3xl">
            <div class="flex items-center justify-between">
                <SectionHeading section-name="Add Tools" @on-click="onBackButtonClick" />
                <Button @buttonClick="router.push('/setup')" buttonSize="small" buttonType="btn-outline-light" class="flex items-center gap-1">
                    Manage Plugins
                </Button>
            </div>
            <div class="mt-6 space-y-4">
                <template v-for="tool in filteredTools" :key="tool.name">
                    <Card>
                        <template v-slot:icon>
                            <div class="flex items-center justify-center h-9 w-9 rounded shadow bg-neutral-100 mt-0.5">
                                <Icon :icon="tool.plugin_icon ? tool.plugin_icon : 'mdi:function'" class="text-nearyyellow-200/90 w-5 h-5" />
                            </div>
                        </template>
                        <div class="leading-7">
                            <div class="text-field-default-foreground text-sm font-medium">{{ tool.display_name }}</div>
                            <div class="text-sm text-nearygray-400">{{ tool.description }}</div>
                        </div>
                        <template v-slot:button>
                            <Icon icon="heroicons:plus" @click="addTool(tool.name)"
                            class="cursor-pointer shrink-0 ml-4 mr-2 w-5 h-5" />
                        </template>
                    </Card>
                </template>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import { useRouter } from 'vue-router';
import { useAppStore } from '@/store/index.js';
import SectionHeading from './common/SectionHeading.vue'
import Card from './common/Card.vue'
import Button from './common/Button.vue'
import { Icon } from '@iconify/vue';

const store = useAppStore();
const router = useRouter();

const filteredTools = computed(() => {
    if (store.selectedConversation && store.selectedConversation.plugins && store.availablePlugins) {
        let selectedTools = [];

        store.selectedConversation.plugins.forEach(plugin => {
            if (plugin.functions.tools) {
                Object.entries(plugin.functions.tools).forEach(([name, details]) => {
                    selectedTools.push(name);
                });
            }
        });

        let tools = [];

        store.availablePlugins.forEach(plugin => {
            if (plugin.is_enabled && plugin.functions.tools) {
                Object.entries(plugin.functions.tools).forEach(([name, details]) => {
                    if (!selectedTools.includes(name)) {
                        tools.push({
                            name: name,
                            plugin_icon: plugin.icon,
                            display_name: details.display_name,
                            description: details.description
                        });
                    }
                });
            }
        });

        return tools;
    }
    return [];
});

const addTool = (toolName) => {
    const availablePlugin = store.availablePlugins.find(plugin => {
        return plugin.functions.tools && Object.keys(plugin.functions.tools).includes(toolName);
    });

    if (availablePlugin) {
        let pluginInstance = store.selectedConversation.plugins.find(plugin => plugin.name === availablePlugin.name);

        if (!pluginInstance) {
            pluginInstance = {
                "plugin_id": availablePlugin.id,
                "conversation_id": store.selectedConversation.id,
                "name": availablePlugin.name,
                "display_name": availablePlugin.display_name,
                "description": availablePlugin.description,
                "icon": availablePlugin.icon,
                "author": availablePlugin.author,
                "url": availablePlugin.url,
                "version": availablePlugin.version,
                "data": null,
                "settings": availablePlugin.settings,
                "functions": {"tools": {}},
                "is_enabled": true
            };
            store.selectedConversation.plugins.push(pluginInstance);
        }

        if (!pluginInstance.functions.tools) {
            pluginInstance.functions.tools = {}
        }
        pluginInstance.functions['tools'][toolName] = availablePlugin.functions['tools'][toolName];
        store.updateConversation(store.selectedConversation);
    }
}

const onBackButtonClick = () => {
    router.go(-1);
};

</script>