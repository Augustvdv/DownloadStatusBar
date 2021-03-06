<template>
    <div id="DownloadStatusBarTooltip" class="dsb-tooltip" v-if="tooltipShown" :class="`dsb-theme-${theme}`" :style="{left: left}" :ref="'tooltip'">
        <table class="dsb-tooltip-table">
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">Filename:</th>
                <td class="dsb-tooltip-cell">{{ filename }}</td>
            </tr>
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">Url:</th>
                <td class="dsb-tooltip-cell">{{ download.downloadItem.url }}</td>
            </tr>
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">Referrer:</th>
                <td class="dsb-tooltip-cell">{{ download.downloadItem.referrer || 'None' }}</td>
            </tr>
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">Speed:</th>
                <td class="dsb-tooltip-cell">{{ downloadSpeed }}</td>
            </tr>
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">Status:</th>
                <td class="dsb-tooltip-cell">{{ status }}</td>
            </tr>
            <tr v-if="download.downloadItem.state !== 'complete'">
                <th class="dsb-tooltip-heading">Progress:</th>
                <td class="dsb-tooltip-cell">{{ progress }}</td>
            </tr>
            <tr v-else>
                <th class="dsb-tooltip-heading">Filesize:</th>
                <td class="dsb-tooltip-cell">{{ filesize }}</td>
            </tr>
            <tr class="dsb-tooltip-table-row">
                <th class="dsb-tooltip-heading">MIME type:</th>
                <td class="dsb-tooltip-cell">{{ download.downloadItem.mime || 'Unknown' }}</td>
            </tr>
            <tr v-if="isImage">
                <th class="dsb-tooltip-heading">Preview:</th>
                <td class="dsb-tooltip-cell"><img class="dsb-preview" :src="download.downloadItem.url"></td>
            </tr>
        </table>
    </div>
</template>

<script lang="ts">
    import Vue from 'vue';
    import events from './events';
    import * as helpers from '../helpers';
    import {DSBDownload} from '../DSBDownload';

    export default Vue.extend({
        name: 'tooltip',
        props: {
            theme: String
        },
        data(): {
            download: DSBDownload | null,
            element: HTMLElement | null,
            tooltipShown: boolean,
            left: string,
        } {
            return {
                download: null,
                element: null,
                tooltipShown: false,
                left: '0',
            }
        },
        computed: {
            filename(): string {
                return this.download!.downloadItem.filename;
            },
            status(): string {
                return this.download!.status();
            },

            progress(): string {
                return this.download!.progress();
            },

            downloaded(): string {
                return helpers.formatFileSize(this.download!.downloadItem.bytesReceived);
            },

            totalsize(): string {
                return helpers.formatFileSize(this.download!.downloadItem.totalBytes);
            },

            filesize(): string {
                return helpers.formatFileSize(this.download!.downloadItem.fileSize);
            },

            percentDone(): number {
                return this.download!.percentDownloaded();
            },

            isImage(): boolean {
                if (!this.download) {
                    return false;
                }

                return this.download.isImage();
            },

            downloadSpeed(): string {
                if (!this.download) {
                    return 'Unknown';
                }

                return `${helpers.formatFileSize(this.download.calculateDownloadSpeed())}/s`;
            },
        },

        methods: {
            calculateLeftPosition(): string {
                if (this.element) {
                    let downloadOffset = this.element.offsetLeft;

                    if (downloadOffset + this.$el.offsetWidth > window.innerWidth) {
                        return `${window.innerWidth - this.$el.offsetWidth}px`;
                    } else {
                        return `${downloadOffset}px`;
                    }
                }

                return '0';
            },
        },

        updated() {
            this.left = this.calculateLeftPosition();
        },

        mounted() {
            events.$on('showTooltip', (download: DSBDownload, element: HTMLElement) => {
                this.tooltipShown = true;
                this.download = download;
                this.element = element;
            });

            events.$on('hideTooltip', () => {
                this.tooltipShown = false;
            });
        },
    });
</script>

<style lang="scss" scoped>
    @import "../scss/variables";
    @import "../scss/mixins";

    #DownloadStatusBarTooltip {
        @include reset;
        background : light-theme("background");
        border     : 1px solid light-theme("border");
        bottom     : 100%;
        color      : light-theme("text");
        cursor     : default;
        left       : 0;
        margin     : 0;
        max-width  : 100%;
        min-width  : 300px;
        overflow   : hidden;
        padding    : 10px;
        position   : absolute;
        width      : auto;
        z-index    : 10;

        .dsb-tooltip-table {
            @include reset;
            max-width : 100%;
        }

        .dsb-tooltip-table-row {
            @include reset;

            + .dsb-tooltip-table-row {
                .dsb-tooltip-heading, .dsb-tooltip-cell { padding-top : 5px }
            }
        }

        .dsb-tooltip-heading, .dsb-tooltip-cell {
            @include reset;
            background      : light-theme("background");
            border-collapse : collapse;
            line-height     : 1.2;
        }

        .dsb-tooltip-cell {
            overflow      : hidden;
            padding-left  : 5px;
            text-overflow : ellipsis;
            word-break    : break-all;
        }

        .dsb-tooltip-heading {
            font-weight    : bold;
            text-align     : right !important;
            vertical-align : top;
            white-space    : nowrap;
        }

        .dsb-preview {
            max-width  : 300px;
            max-height : 200px;
            width      : auto;
            height     : auto;
        }

        &.dsb-theme-dark {
            background : dark-theme("background");
            border     : 1px solid dark-theme("border");
            color      : dark-theme("text");

            .dsb-tooltip-heading, .dsb-tooltip-cell {
                background : dark-theme("background");
                color      : dark-theme("text");
            }
        }
    }
</style>