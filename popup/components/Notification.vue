<template>
    <v-card color="yellow lighten-3">
        <div style="float: right">
            <v-btn
                text
                class="ml-4 mr-2 px-2 primary--text"
                @click="closeNotification(notification.id)"
            >
                {{ $i18n("close") }}
                <v-icon>mdi-close</v-icon>
            </v-btn>
        </div>

        <v-card-text class="primary--text">
            <p v-for="(msg, index) of notification.messages" :key="index">
                {{ msg }}
            </p>
        </v-card-text>
        <v-card-actions>
            <v-row justify="center" align="end" class="mb-1 mt-1">
                <v-col>
                    <v-row align="end" class="ml-2 justify-left my-0">
                        <v-tooltip
                            v-for="(but, index) of notification.buttons"
                            :key="index"
                            top
                            :disabled="!but.tooltip"
                            :open-delay="tooltipDelay"
                        >
                            <template #activator="{ on }">
                                <v-btn
                                    :id="but.id"
                                    class="mr-4 my-2"
                                    v-on="on"
                                    @click="
                                        dispatchActionResponse(
                                            notification.id,
                                            but.action,
                                            but.values
                                        )
                                    "
                                >
                                    {{ but.label }}
                                </v-btn>
                                <span>{{ but.tooltip }}</span>
                            </template>
                        </v-tooltip>
                    </v-row>
                </v-col>
            </v-row>
        </v-card-actions>
    </v-card>
</template>

<script lang="ts">
import { mapActions } from "vuex";
import { names as actionNames } from "../../store/action-names";
import { ButtonAction } from "../../common/Button";
import { configManager } from "../../common/ConfigManager";
import { AddonMessage } from "../../common/AddonMessage";
import { Port } from "../../common/port";
import { tooltipDelay } from "../../common/Timings";

export default {
    mixins: [Port.mixin],
    props: ["notification"],
    data: () => ({
        tooltipDelay: tooltipDelay
    }),
    methods: {
        ...mapActions(actionNames),
        dispatchActionResponse(id: string, action: ButtonAction, data: { [id: string]: string }) {
            const pm = (this as any).postMessage;
            switch (action) {
                case "enableHighSecurityKPRPCConnection":
                    configManager.current.connSLClient = 3;
                    configManager.save();
                    break;
                case "loadUrlUpgradeKee":
                    pm({ loadUrlUpgradeKee: true });
                    break;
                case "disableNotifyWhenEntryUpdated":
                    configManager.current.notifyWhenEntryUpdated = false;
                    configManager.save();
                    break;
                case "launchLoginEditorFromNotification":
                    pm({
                        loginEditor: {
                            uuid: data.uuid,
                            DBfilename: data.fileName
                        }
                    } as AddonMessage);
                    break;
            }
            pm({ removeNotification: id } as AddonMessage);
        },
        closeNotification(id: string) {
            const pm = (this as any).postMessage;
            pm({ removeNotification: id } as AddonMessage);
        }
    }
};
</script>
