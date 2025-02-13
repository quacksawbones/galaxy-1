<template>
    <div>
        <message :message="message" :status="status"></message>
        <b-tabs>
            <b-tab title="Toolshed Tools">
                <b-tabs>
                    <b-tab title="HTML Sanitized">
                        <base-grid :is-loaded="isLoaded" :columns="toolshedColumns" id="sanitize-allow-grid">
                            <template v-slot:rows>
                                <template v-for="(row, blockedIdx) in toolshedBlocked">
                                    <tr :key="blockedIdx">
                                        <td>{{ row.tool_name }}</td>
                                        <td>
                                            <span>{{ row.tool_id[0] }}</span>
                                        </td>
                                        <td>
                                            <button @click="allowHTML(row.ids.owner)">{{ row.tool_id[2] }}</button>
                                        </td>
                                        <td>
                                            <button @click="allowHTML(row.ids.repository)">{{ row.tool_id[3] }}</button>
                                        </td>
                                        <td>
                                            <button @click="allowHTML(row.ids.tool)">{{ row.tool_id[4] }}</button>
                                        </td>
                                        <td>
                                            <button @click="allowHTML(row.ids.full)">{{ row.tool_id[5] }}</button>
                                        </td>
                                    </tr>
                                </template>
                            </template>
                        </base-grid>
                    </b-tab>
                    <b-tab title="HTML Rendered">
                        <base-grid :is-loaded="isLoaded" :columns="columns" id="sanitize-allow-grid">
                            <template v-slot:rows>
                                <template v-for="(row, allowedIdx) in toolshedAllowed">
                                    <tr :key="allowedIdx">
                                        <td>{{ row.tool_name }}</td>
                                        <td>
                                            <template v-for="(part, part_idx) in row.tool_id">
                                                <span :key="part_idx">{{ part }}</span>
                                            </template>
                                        </td>
                                        <td>
                                            <button @click="sanitizeHTML(row.ids.allowed)">Sanitize HTML</button>
                                        </td>
                                    </tr>
                                </template>
                            </template>
                        </base-grid>
                    </b-tab>
                </b-tabs>
            </b-tab>
            <b-tab title="Local Tools">
                <b-tabs>
                    <b-tab title="HTML Sanitized">
                        <base-grid :is-loaded="isLoaded" :columns="columns" id="sanitize-allow-grid">
                            <template v-slot:rows>
                                <template v-for="(row, localBlockedIdx) in localBlocked">
                                    <tr :key="localBlockedIdx">
                                        <td>{{ row.tool_name }}</td>
                                        <td>{{ row.tool_id[0] }}</td>
                                        <td>
                                            <button @click="allowHTML(row.ids.full)">Render HTML</button>
                                        </td>
                                    </tr>
                                </template>
                            </template>
                        </base-grid>
                    </b-tab>
                    <b-tab title="HTML Rendered">
                        <base-grid :is-loaded="isLoaded" :columns="columns" id="sanitize-allow-grid">
                            <template v-slot:rows>
                                <template v-for="(row, localAllowedIdx) in localAllowed">
                                    <tr :key="localAllowedIdx">
                                        <td>{{ row.tool_name }}</td>
                                        <td>{{ row.tool_id[0] }}</td>
                                        <td>
                                            <button @click="sanitizeHTML(row.ids.allowed)">Sanitize HTML</button>
                                        </td>
                                    </tr>
                                </template>
                            </template>
                        </base-grid>
                    </b-tab>
                </b-tabs>
            </b-tab>
        </b-tabs>
    </div>
</template>

<script>
import BaseGrid from "./BaseGrid.vue";
import axios from "axios";
import { getAppRoot } from "onload/loadConfig";
import Message from "../Message.vue";

export default {
    data() {
        return {
            isLoaded: false,
            localAllowed: [],
            localBlocked: [],
            toolshedAllowed: [],
            toolshedBlocked: [],
            columns: [
                { text: "Tool Name", dataIndex: "tool_name" },
                { text: "Tool ID", dataIndex: "tool_id" },
                { text: "Action", dataIndex: "action" },
            ],
            toolshedColumns: [
                { text: "Tool Name", dataIndex: "tool_name" },
                { text: "Toolshed" },
                { text: "Owner" },
                { text: "Repository" },
                { text: "Tool" },
                { text: "Version" },
            ],
            message: "",
            status: "",
        };
    },

    components: {
        message: Message,
        "base-grid": BaseGrid,
    },

    created() {
        axios
            .get(`${getAppRoot()}api/sanitize_allow`)
            .then((response) => {
                this.isLoaded = true;
                this.localAllowed = response.data.allowed_local;
                this.localBlocked = response.data.blocked_local;
                this.toolshedAllowed = response.data.allowed_toolshed;
                this.toolshedBlocked = response.data.blocked_toolshed;
                this.message = response.data.message;
                this.status = response.data.status;
            })
            .catch((error) => {
                console.error(error);
            });
    },

    methods: {
        allowHTML(tool_id) {
            axios
                .put(`${getAppRoot()}api/sanitize_allow?tool_id=${tool_id}`, {
                    params: {
                        tool_id: tool_id,
                    },
                })
                .then((response) => {
                    this.localAllowed = response.data.allowed_local;
                    this.localBlocked = response.data.blocked_local;
                    this.toolshedAllowed = response.data.allowed_toolshed;
                    this.toolshedBlocked = response.data.blocked_toolshed;
                    this.message = response.data.message;
                    this.status = response.data.status;
                })
                .catch((error) => {
                    console.error(error);
                });
        },
        sanitizeHTML(tool_id) {
            axios
                .delete(`${getAppRoot()}api/sanitize_allow`, {
                    params: {
                        tool_id: tool_id,
                    },
                })
                .then((response) => {
                    this.localAllowed = response.data.allowed_local;
                    this.localBlocked = response.data.blocked_local;
                    this.toolshedAllowed = response.data.allowed_toolshed;
                    this.toolshedBlocked = response.data.blocked_toolshed;
                    this.message = response.data.message;
                    this.status = response.data.status;
                })
                .catch((error) => {
                    console.error(error);
                });
        },
    },
};
</script>
