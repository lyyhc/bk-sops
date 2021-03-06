/**
* Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community
* Edition) available.
* Copyright (C) 2017-2019 THL A29 Limited, a Tencent company. All rights reserved.
* Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
* http://opensource.org/licenses/MIT
* Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
* an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
* specific language governing permissions and limitations under the License.
*/
<template>
    <div class="config-wrapper">
        <div class="config-title-wrapper">
            <span class="title-border">|</span>
            <span class="title-text">{{templateTitle}}</span>
        </div>
        <div class="config-name-wrapper">
            <div class="name-show-mode" v-if="isShowMode">
                <h3 class="canvas-name" :title="tName">{{tName}}</h3>
                <span class="common-icon-edit" @click="onNameEditing"></span>
            </div>
            <BaseInput
                v-else
                class="canvas-name-input"
                ref="canvasNameInput"
                :title="tName"
                v-model="tName"
                :placeholder="i18n.placeholder"
                v-validate="templateNameRule"
                data-vv-name="templateName"
                :name="'templateName'"
                :has-error="errors.has('templateName')"
                @input="onInputName"
                @blur="onInputBlur"
                @enter="onInputBlur">
            </BaseInput>
            <span class="name-error common-error-tip error-msg">{{ errors.first('templateName') }}</span>
        </div>
        <div class="canvas-operation-wrapper">
            <bk-button
                type="primary"
                class="save-canvas"
                :loading="templateSaving"
                @click="onSaveTemplate(false)">
                {{ i18n.save }}
            </bk-button>
            <bk-button
                type="primary"
                class="task-btn"
                :loading="createTaskSaving"
                @click="onSaveTemplate(true)">
                {{ createTaskBtnText }}
            </bk-button>
            <router-link class="bk-button bk-button-default" :to="getHomeUrl()">{{ i18n.return }}</router-link>
        </div>
    </div>
</template>
<script>
    import '@/utils/i18n.js'
    import { mapMutations } from 'vuex'
    import { NAME_REG, STRING_LENGTH } from '@/constants/index.js'
    import BaseInput from '@/components/common/base/BaseInput.vue'

    export default {
        name: 'ConfigBar',
        components: {
            BaseInput
        },
        props: [
            'name', 'cc_id', 'template_id', 'type', 'common', 'templateSaving',
            'createTaskSaving', 'isTemplateDataChanged'
        ],
        data () {
            return {
                i18n: {
                    placeholder: gettext('请输入名称'),
                    NewProcess: gettext('新建流程'),
                    editProcess: gettext('编辑流程'),
                    addTask: gettext('新建任务'),
                    saveTplAndcreateTask: gettext('保存并新建任务'),
                    save: gettext('保存'),
                    return: gettext('返回')
                },
                tName: this.name.trim(),
                templateNameRule: {
                    required: true,
                    max: STRING_LENGTH.TEMPLATE_NAME_MAX_LENGTH,
                    regex: NAME_REG
                },
                isShowMode: true
            }
        },
        computed: {
            templateTitle () {
                return this.$route.query.template_id === undefined ? this.i18n.NewProcess : this.i18n.editProcess
            },
            createTaskBtnText () {
                return (this.isTemplateDataChanged || this.type === 'new') ? this.i18n.saveTplAndcreateTask : this.i18n.addTask
            }
        },
        watch: {
            name (val) {
                this.tName = val
            }
        },
        methods: {
            ...mapMutations('template/', [
                'setTemplateName'
            ]),
            onInputName (val) {
                this.$emit('onChangeName', val)
            },
            onSaveTemplate (saveAndCreate = false) {
                this.$validator.validateAll().then((result) => {
                    if (!result) return
                    this.tName = this.tName.trim()
                    this.setTemplateName(this.tName)
                    if (saveAndCreate && !this.isTemplateDataChanged && this.type !== 'new') {
                        const taskUrl = this.getTaskUrl()
                        this.$router.push(taskUrl)
                    } else {
                        this.$emit('onSaveTemplate', saveAndCreate)
                    }
                })
            },
            getHomeUrl () {
                let url = `/template/home/${this.cc_id}/`
                if (this.common) {
                    url += '?common=1'
                }
                return url
            },
            getTaskUrl () {
                let url = `/template/newtask/${this.cc_id}/selectnode/?template_id=${this.template_id}`
                if (this.common) {
                    url += '&common=1'
                }
                return url
            },
            onNameEditing () {
                this.isShowMode = false
                this.$nextTick(() => {
                    this.$refs.canvasNameInput.focus()
                    this.$refs.canvasNameInput.select()
                })
            },
            onInputBlur () {
                this.$validator.validateAll().then((result) => {
                    if (!result) {
                        return
                    }
                    this.isShowMode = true
                })
            }
        }
    }
</script>
<style lang="scss" scoped>
@import '@/scss/config.scss';
.config-wrapper {
    position: relative;
    height: 60px;
    background: #f4f7fa;
    border-bottom: 1px solid $commonBorderColor;
    text-align: center;
    .config-title-wrapper {
        float: left;
        margin: 19px 0 0 20px;
        .title-border {
            color: #a3c5fd;
        }
        .title-text {
            line-height: 20px;
            margin-left: 10px;
            font-size: 14px;
            font-weight: 600;
            color:#313238;
        }
    }
    .config-name-wrapper {
        margin: 0 auto;
        padding-top: 15px;
        width: 430px;
        .name-show-mode {
            display: inline-block;
        }
        .canvas-name {
            display: inline-block;
            margin: 0;
            max-width: 400px;
            height: 30px;
            line-height: 30px;
            font-size: 14px;
            font-weight: normal;
            overflow: hidden;
            text-overflow:ellipsis;
            white-space: nowrap;
            color: #606266;
        }
        .common-icon-edit {
            float: right;
            margin: 6px 0 0 4px;
            color: #546a9e;
            vertical-align: 9px;
            cursor: pointer;
            &:hover {
                color: #3480ff;
            }
        }
        .canvas-name-input {
            padding: 4px 10px;
            max-width: 400px;
            height: auto;
            line-height: 12px;
            font-size: 14px;
            overflow: hidden;
            text-overflow:ellipsis;
            white-space: nowrap;
            border: 1px solid $commonBorderColor;
            outline: none;
            &:focus {
                border-color: $blueDefault;
            }
        }
        .name-error {
            position: absolute;
            margin: 7px 0 0 10px;
            font-size: 12px;
        }
    }
    .canvas-operation-wrapper {
        position: absolute;
        top: 14px;
        right: 20px;
        .save-canvas {
            width: 90px;
            height: 32px;
            line-height: 32px;
            margin-left: 36px;
        }
        .bk-button-default {
            width: 90px;
            height: 32px;
            line-height: 32px;
            margin-left: 10px;
        }
        .bk-button.bk-primary {
            height: 32px;
            line-height: 32px;
            margin-left: 10px;
        }
    }
}
</style>
