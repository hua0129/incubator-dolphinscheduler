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
  <m-list-construction :title="'Mysql ' + $t('Manage')">
    <template slot="content">
      <div class="servers-wrapper mysql-model" v-show="mysqlList.length">
        <div class="row" v-for="(item,$index) in mysqlList">
          <div class="col-md-2">
            <div class="text-num-model text">
              <div class="title">
                <span :title="$t('Health status')">{{$t('Health status')}}</span>
              </div>
              <div class="value-p">
                <span class="state">
                  <i class="iconfont success" v-if="item.state">&#xe607;</i>
                  <i class="iconfont error" v-else>&#xe626;</i>
                </span>
              </div>
              <div class="text-1">
                {{$t('Health status')}}
              </div>
            </div>
          </div>
          <div class="col-md-3">
            <div class="text-num-model text">
              <div class="title">
                <span :title="$t('Max connections')">{{$t('Max connections')}} - {{item.date | formatDate}}</span>
              </div>
              <div class="value-p">
                <b :style="{color:color[0]}">{{item.maxConnections}}</b>
              </div>
              <div class="text-1">
                {{$t('Max connections')}}
              </div>
            </div>
          </div>
          <div class="col-md-3">
            <div class="text-num-model text">
              <div class="title">
                <span :title="$t('Threads connections')">{{$t('Threads connections')}}</span>
              </div>
              <div class="value-p">
                <b :style="{color:color[8]}">{{item.threadsConnections}}</b>
              </div>
              <div class="text-1">
                {{$t('Threads connections')}}
              </div>
            </div>
          </div>
          <div class="col-md-2">
            <div class="text-num-model text">
              <div class="title">
                <span :title="$t('Max used connections')">{{$t('Max used connections')}}</span>
              </div>
              <div class="value-p">
                <b :style="{color:color[2]}">{{item.maxUsedConnections}}</b>
              </div>
              <div class="text-1">
                {{$t('Max used connections')}}
              </div>
            </div>
          </div>
          <div class="col-md-2">
            <div class="text-num-model text">
              <div class="title">
                <span :title="$t('Threads running connections')">{{$t('Threads running connections')}}</span>
              </div>
              <div class="value-p">
                <b :style="{color:color[4]}">{{item.threadsRunningConnections}}</b>
              </div>
              <div class="text-1">
                {{$t('Threads running connections')}}
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-if="!mysqlList.length">
        <m-no-data></m-no-data>
      </div>
      <m-spin :is-spin="isLoading" ></m-spin>
    </template>
  </m-list-construction>
</template>
<script>
  import { mapActions } from 'vuex'
  import mList from './_source/zookeeperList'
  import mSpin from '@/module/components/spin/spin'
  import mNoData from '@/module/components/noData/noData'
  import themeData from '@/module/echarts/themeData.json'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'servers-mysql',
    data () {
      return {
        isLoading: false,
        mysqlList: [],
        color: themeData.color
      }
    },
    props: {},
    methods: {
      ...mapActions('monitor', ['getDatabaseData'])
    },
    watch: {},
    created () {
      this.isLoading = true
      this.getDatabaseData().then(res => {
        this.mysqlList = res
        this.isLoading = false
      }).catch(() => {
        this.isLoading = false
      })
    },
    mounted () {
    },
    components: { mList, mListConstruction, mSpin, mNoData }
  }
</script>
<style lang="scss" rel="stylesheet/scss">
  @import "./servers";
</style>