/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <div class="sql-model">
    <m-list-box>
      <div slot="text">{{$t('Datasource')}}</div>
      <div slot="content">
        <m-datasource
                ref="refDs"
                @on-dsData="_onDsData"
                :data="{ type:type,datasource:datasource }">
        </m-datasource>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('SQL Type')}}</div>
      <div slot="content">
        <div style="display: inline-block;">
          <m-sql-type
                  @on-sqlType="_onSqlType"
                  :sql-type="sqlType">
          </m-sql-type>
        </div>
        <div v-if="!sqlType" style="display: inline-block;padding-left: 10px;margin-top: 2px;">
          <x-checkbox-group v-model="showType">
            <x-checkbox :label="'TABLE'" :disabled="isDetails">{{$t('Table')}}</x-checkbox>
            <x-checkbox :label="'ATTACHMENT'" :disabled="isDetails">{{$t('Attachment')}}</x-checkbox>
          </x-checkbox-group>
        </div>
      </div>
    </m-list-box>
    <template v-if="!sqlType && showType.length">
      <m-list-box>
        <div slot="text">{{$t('Title')}}</div>
        <div slot="content">
          <x-input
            type="input"
            v-model="title"
            :placeholder="$t('Please enter the title of email')"
            autocomplete="off">
          </x-input>
        </div>
      </m-list-box>
      <m-list-box>
        <div slot="text">{{$t('Recipient')}}</div>
        <div slot="content">
          <m-email v-model="receivers" :disabled="isDetails" :repeat-data="receiversCc"></m-email>
        </div>
      </m-list-box>
      <m-list-box>
        <div slot="text">{{$t('Cc')}}</div>
        <div slot="content">
          <m-email v-model="receiversCc" :disabled="isDetails" :repeat-data="receivers"></m-email>
        </div>
      </m-list-box>
    </template>
    <m-list-box v-show="type === 'HIVE'">
      <div slot="text">{{$t('SQL Parameter')}}</div>
      <div slot="content">
        <x-input
                :disabled="isDetails"
                type="input"
                v-model="connParams"
                :placeholder="$t('Please enter format') + ' key1=value1;key2=value2...'"
                autocomplete="off">
        </x-input>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('SQL Statement')}}</div>
      <div slot="content">
        <div class="from-mirror">
          <textarea
                  id="code-sql-mirror"
                  name="code-sql-mirror"
                  style="opacity: 0;">
          </textarea>
        </div>
      </div>
    </m-list-box>
    <m-list-box v-if="type === 'HIVE'">
      <div slot="text">{{$t('UDF Function')}}</div>
      <div slot="content">
        <m-udfs
                ref="refUdfs"
                @on-udfsData="_onUdfsData"
                :udfs="udfs"
                :type="type">
        </m-udfs>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('Custom Parameters')}}</div>
      <div slot="content">
        <m-local-params
                ref="refLocalParams"
                @on-udpData="_onUdpData"
                :udp-list="localParams">
        </m-local-params>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('Pre Statement')}}</div>
      <div slot="content">
        <m-statement-list
          ref="refPreStatements"
          @on-statement-list="_onPreStatements"
          :statement-list="preStatements">
        </m-statement-list>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('Post Statement')}}</div>
      <div slot="content">
        <m-statement-list
          ref="refPostStatements"
          @on-statement-list="_onPostStatements"
          :statement-list="postStatements">
        </m-statement-list>
      </div>
    </m-list-box>
  </div>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import mUdfs from './_source/udfs'
  import mListBox from './_source/listBox'
  import mSqlType from './_source/sqlType'
  import mDatasource from './_source/datasource'
  import mLocalParams from './_source/localParams'
  import mStatementList from './_source/statementList'
  import disabledState from '@/module/mixin/disabledState'
  import mEmail from '@/conf/home/pages/projects/pages/definition/pages/list/_source/email'
  import codemirror from '@/conf/home/pages/resource/pages/file/pages/_source/codemirror'

  let editor

  export default {
    name: 'sql',
    data () {
      return {
        // Data source type
        type: '',
        // data source
        datasource: '',
        // Return to the selected data source
        rtDatasource: '',
        // Sql statement
        sql: '',
        // Custom parameter
        localParams: [],
        // UDF function
        udfs: '',
        // Sql type
        sqlType: 0,
        // Email title
        title: '',
        // Form/attachment
        showType: ['TABLE'],
        // Sql parameter
        connParams: '',
        // Pre statements
        preStatements: [],
        // Post statements
        postStatements: [],
        // recipients
        receivers: [],
        // copy to
        receiversCc: []
      }
    },
    mixins: [disabledState],
    props: {
      backfillItem: Object,
      createNodeId: Number
    },
    methods: {
      /**
       * return sqlType
       */
      _onSqlType (a) {
        this.sqlType = a
      },
      /**
       * return udfs
       */
      _onUdfsData (a) {
        this.udfs = a
      },
      /**
       * return Custom parameter
       */
      _onUdpData (a) {
        this.localParams = a
      },
      /**
       * return data source
       */
      _onDsData (o) {
        this.type = o.type
        this.rtDatasource = o.datasource
      },
      /**
       * return pre statements
       */
      _onPreStatements (a) {
        this.preStatements = a
      },
      /**
       * return post statements
       */
      _onPostStatements (a) {
        this.postStatements = a
      },
      /**
       * verification
       */
      _verification () {
        if (!editor.getValue()) {
          this.$message.warning(`${i18n.$t('Please enter a SQL Statement(required)')}`)
          return false
        }

        // datasource Subcomponent verification
        if (!this.$refs.refDs._verifDatasource()) {
          return false
        }

        // udfs Subcomponent verification Verification only if the data type is HIVE
        if (this.type === 'HIVE') {
          if (!this.$refs.refUdfs._verifUdfs()) {
            return false
          }
        }

        // localParams Subcomponent verification
        if (!this.$refs.refLocalParams._verifProp()) {
          return false
        }

        // preStatements Subcomponent verification
        if (!this.$refs.refPreStatements._verifProp()) {
          return false
        }

        // postStatements Subcomponent verification
        if (!this.$refs.refPostStatements._verifProp()) {
          return false
        }

        // storage
        this.$emit('on-params', {
          type: this.type,
          datasource: this.rtDatasource,
          sql: editor.getValue(),
          udfs: this.udfs,
          sqlType: this.sqlType,
          title: this.title,
          receivers: this.receivers.join(','),
          receiversCc: this.receiversCc.join(','),
          showType: (() => {
            /**
             * Special processing return order TABLE,ATTACHMENT
             * Handling checkout sequence
             */
            let showType = this.showType
            if (showType.length === 2 && showType[0] === 'ATTACHMENT') {
              return [showType[1], showType[0]].join(',')
            } else {
              return showType.join(',')
            }
          })(),
          localParams: this.localParams,
          connParams: this.connParams,
          preStatements: this.preStatements,
          postStatements: this.postStatements
        })
        return true
      },
      /**
       * Processing code highlighting
       */
      _handlerEditor () {
        // editor
        editor = codemirror('code-sql-mirror', {
          mode: 'sql',
          readOnly: this.isDetails
        })

        this.keypress = () => {
          if (!editor.getOption('readOnly')) {
            editor.showHint({
              completeSingle: false
            })
          }
        }

        // Monitor keyboard
        editor.on('keypress', this.keypress)

        editor.setValue(this.sql)

        return editor
      },
      _getReceiver () {
        let param = {}
        let current = this.router.history.current
        if (current.name === 'projects-definition-details') {
          param.processDefinitionId = current.params.id
        } else {
          param.processInstanceId = current.params.id
        }
        this.store.dispatch('dag/getReceiver', param).then(res => {
          this.receivers = res.receivers && res.receivers.split(',') || []
          this.receiversCc = res.receiversCc && res.receiversCc.split(',') || []
        })
      }
    },
    watch: {
      // Listening to sqlType
      sqlType (val) {
        if (val) {
          this.showType = []
        }
        if (val !== 0) {
          this.title = ''
          this.receivers = []
          this.receiversCc = []
        }
      },
      // Listening data source
      type (val) {
        if (val !== 'HIVE') {
          this.connParams = ''
        }
      },
      //
      showType (val) {
        if (!val.length) {
          this.title = ''
          this.receivers = []
          this.receiversCc = []
        }
      }
    },
    created () {
      let o = this.backfillItem

      // Non-null objects represent backfill
      if (!_.isEmpty(o)) {
        // backfill
        this.type = o.params.type || ''
        this.datasource = o.params.datasource || ''
        this.sql = o.params.sql || ''
        this.udfs = o.params.udfs || ''
        this.sqlType = o.params.sqlType
        this.connParams = o.params.connParams || ''
        this.localParams = o.params.localParams || []
        this.showType = o.params.showType.split(',') || []
        this.preStatements = o.params.preStatements || []
        this.postStatements = o.params.postStatements || []
        this.title = o.params.title || ''
        this.receivers = o.params.receivers && o.params.receivers.split(',') || []
        this.receiversCc = o.params.receiversCc && o.params.receiversCc.split(',') || []
      }
      if (!_.some(this.store.state.dag.tasks, { id: this.createNodeId }) &&
        this.router.history.current.name !== 'definition-create') {
        this._getReceiver()
      }
    },
    mounted () {
      setTimeout(() => {
        this._handlerEditor()
      }, 200)
    },
    destroyed () {
      /**
       * Destroy the editor instance
       */
      if (editor) {
        editor.toTextArea() // Uninstall
        editor.off($('.code-sql-mirror'), 'keypress', this.keypress)
      }
    },
    computed: {},
    components: { mListBox, mDatasource, mLocalParams, mUdfs, mSqlType, mStatementList, mEmail }
  }
</script>
