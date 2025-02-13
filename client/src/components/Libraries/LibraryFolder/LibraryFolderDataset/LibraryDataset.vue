<template>
    <div v-if="dataset">
        <div v-if="!isEditMode">
            <LibraryBreadcrumb :current-id="dataset_id" :full_path="dataset.full_path" />
            <!-- Toolbar -->
            <b-button
                title="Download dataset"
                class="mr-1 mb-2"
                @click="download(datasetDownloadFormat, dataset_id)"
                data-test-id="download-btn">
                <font-awesome-icon icon="download" />
                Download
            </b-button>
            <b-button
                @click="importToHistory"
                title="Import dataset into history"
                class="mr-1 mb-2"
                data-test-id="import-history-btn">
                <font-awesome-icon icon="book" />
                to History
            </b-button>
            <span v-if="dataset.can_user_modify">
                <b-button
                    @click="isEditMode = true"
                    title="Modify library item"
                    class="mr-1 mb-2"
                    data-test-id="modify-btn">
                    <font-awesome-icon icon="pencil-alt" />
                    Modify
                </b-button>
                <b-button
                    title="Attempt to detect the format of dataset"
                    @click="detectDatatype"
                    class="mr-1 mb-2"
                    data-test-id="auto-detect-btn">
                    <font-awesome-icon icon="redo" />
                    Auto-detect datatype
                </b-button>
            </span>
            <b-button
                title="Manage permissions"
                v-if="dataset.can_user_manage"
                class="mr-1 mb-2"
                :to="{
                    name: 'LibraryFolderDatasetPermissions',
                    params: { folder_id: folder_id, dataset_id: dataset_id },
                }"
                data-test-id="permissions-btn">
                <font-awesome-icon icon="users" />
                Permissions
            </b-button>
        </div>
        <div v-if="dataset.is_unrestricted" data-test-id="unrestricted-msg">
            This dataset is unrestricted so everybody with the link can access it.
            <copy-to-clipboard
                message="A link to current dataset was copied to your clipboard"
                :text="currentRouteName"
                title="Copy link to this dataset " />
        </div>
        <!-- Table -->
        <b-table
            v-if="table_items"
            :fields="fields"
            :items="table_items"
            class="dataset_table mt-2"
            thead-class="d-none"
            striped
            small
            data-test-id="dataset-table">
            <template v-slot:cell(name)="row">
                <strong>{{ row.item.name }}</strong>
            </template>
            <template v-slot:cell(value)="row">
                <div v-if="isEditMode">
                    <b-form-input
                        v-if="row.item.name === fieldTitles.name"
                        :value="row.item.value"
                        v-model="modifiedDataset.name" />
                    <DatatypesProvider
                        v-else-if="row.item.name === fieldTitles.file_ext"
                        v-slot="{ item: datatypes, loading: loadingDatatypes }">
                        <SingleItemSelector
                            collection-name="Data Types"
                            :loading="loadingDatatypes"
                            :items="datatypes"
                            :current-item-id="dataset.file_ext"
                            @update:selected-item="onSelectedDatatype" />
                    </DatatypesProvider>
                    <GenomeProvider
                        v-else-if="row.item.name === fieldTitles.genome_build"
                        v-slot="{ item: genomes, loading: loadingGenomes }">
                        <SingleItemSelector
                            collection-name="Genomes"
                            :loading="loadingGenomes"
                            :items="genomes"
                            :current-item-id="dataset.genome_build"
                            @update:selected-item="onSelectedGenome" />
                    </GenomeProvider>
                    <b-form-input
                        v-else-if="row.item.name === fieldTitles.message"
                        :value="row.item.value"
                        v-model="modifiedDataset.message" />
                    <b-form-input
                        v-else-if="row.item.name === fieldTitles.misc_info"
                        :value="row.item.value"
                        v-model="modifiedDataset.misc_info" />
                    <div v-else>{{ row.item.value }}</div>
                </div>
                <div v-else>
                    <div>{{ row.item.value }}</div>
                </div>
            </template>
        </b-table>
        <!-- Edit Controls -->
        <div v-if="isEditMode">
            <b-button @click="isEditMode = false" class="mr-1 mb-2">
                <font-awesome-icon :icon="['fas', 'times']" />
                Cancel
            </b-button>
            <b-button @click="updateDataset" class="mr-1 mb-2">
                <font-awesome-icon :icon="['far', 'save']" />
                Save
            </b-button>
        </div>
        <!-- Peek View -->
        <div v-if="dataset.peek" v-html="dataset.peek" data-test-id="peek-view" />
    </div>
</template>

<script>
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faUsers, faRedo, faPencilAlt, faBook, faDownload, faTimes } from "@fortawesome/free-solid-svg-icons";
import { faSave } from "@fortawesome/free-regular-svg-icons";
import { library } from "@fortawesome/fontawesome-svg-core";
import mod_import_dataset from "components/Libraries/LibraryFolder/TopToolbar/import-to-history/import-dataset";
import { Services } from "components/Libraries/LibraryFolder/services";
import LibraryBreadcrumb from "components/Libraries/LibraryFolder/LibraryBreadcrumb";
import download from "components/Libraries/LibraryFolder/TopToolbar/download";
import CopyToClipboard from "components/CopyToClipboard";
import { Toast } from "ui/toast";
import { fieldTitles } from "components/Libraries/LibraryFolder/LibraryFolderDataset/constants";
import { GenomeProvider, DatatypesProvider } from "components/providers";
import SingleItemSelector from "components/SingleItemSelector";

library.add(faUsers, faRedo, faBook, faDownload, faPencilAlt, faTimes, faSave);

function buildFields(fieldTitles, data) {
    return Object.entries(fieldTitles).flatMap(([property, title]) =>
        data[property] ? { name: title, value: data[property] } : []
    );
}

export default {
    props: {
        dataset_id: {
            type: String,
            required: true,
        },
        folder_id: {
            type: String,
            required: true,
        },
    },
    components: {
        LibraryBreadcrumb,
        CopyToClipboard,
        FontAwesomeIcon,
        GenomeProvider,
        DatatypesProvider,
        SingleItemSelector,
    },
    data() {
        return {
            dataset: undefined,
            modifiedDataset: {},
            currentRouteName: window.location.href,
            datasetDownloadFormat: "uncompressed",
            download: download,
            isEditMode: false,
            fieldTitles: fieldTitles,
            table_items: [],
            fields: [{ key: "name" }, { key: "value" }],
        };
    },
    async created() {
        this.services = new Services({ root: this.root });
        const datasetResponse = await this.services.getDataset(this.dataset_id);
        this.populateDatasetDetailsTable(datasetResponse);
    },
    computed: {
        datasetChanges() {
            const changes = {};
            for (var prop in this.dataset) {
                if (this.dataset[prop] !== this.modifiedDataset[prop]) {
                    changes[prop] = this.modifiedDataset[prop];
                }
            }
            return changes;
        },
        hasChanges() {
            for (var _ in this.datasetChanges) {
                return true;
            }
            return false;
        },
    },
    methods: {
        populateDatasetDetailsTable(data) {
            this.dataset = data;
            Object.assign(this.modifiedDataset, this.dataset);
            this.table_items = buildFields(this.fieldTitles, this.dataset);
        },
        detectDatatype() {
            this.services.updateDataset(
                this.dataset_id,
                { file_ext: "auto" },
                (response) => {
                    this.populateDatasetDetailsTable(response);
                    Toast.success("Changes to library dataset saved.");
                },
                (error) => Toast.error(error)
            );
        },
        updateDataset() {
            if (this.hasChanges) {
                this.services.updateDataset(
                    this.dataset_id,
                    this.datasetChanges,
                    (response) => {
                        this.populateDatasetDetailsTable(response);
                        Toast.success("Changes to library dataset saved.");
                    },
                    (error) => Toast.error(error)
                );
            }
            this.isEditMode = false;
        },
        importToHistory() {
            new mod_import_dataset.ImportDatasetModal({
                selected: { dataset_ids: [this.dataset_id] },
            });
        },
        onSelectedGenome(genome) {
            this.modifiedDataset.genome_build = genome.id;
        },
        onSelectedDatatype(datatype) {
            this.modifiedDataset.file_ext = datatype.id;
        },
    },
};
</script>
