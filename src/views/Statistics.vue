<template>
  <Layout>
    <Tabs :dataSource="typeList" :value.sync="type" class-prefix="type"></Tabs>
    <ol>
      <li v-for="(group,index) in groupedList" :key="index">
        <h3 class="title">{{beautify(group.title)}} <span><b>{{group.total}}</b></span></h3>
        <ol>
          <li v-for="item in group.items" :key="item.id"
              class="record">
<!--            <span>标签：{{tagString(item.tags)}}</span>-->
            <span class="staticsNotes">备注：{{item.notes}}</span>
            <span>金额：￥{{item.amount}} </span>
          </li>
        </ol>
      </li>
    </ol>
  </Layout>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component} from 'vue-property-decorator';
  import Tabs from '@/views/Tabs.vue';
  import typeList from '@/constants/recordTypeList';
  import recordTypeList from '@/constants/intervalList';
  import dayjs from 'dayjs';

  console.log(dayjs());
  @Component({
    components: {Tabs}
  })
  export default class Statistics extends Vue {
    type = '-';
    interval = 'day';
    typeList = typeList;
    recordTypeList = recordTypeList;

    beforeCreate() {
      return this.$store.commit('fetchRecords');
    }

    get recordList() {
      return (this.$store.state as RootState).recordList;
    }
    get localTypeList (){
      return (this.$store.state as RootState).tagList
    }

    get groupedList() {
      const {recordList} = this;
      if(recordList.length === 0){return []}
      const newList = recordList.filter(r => r.type === this.type).sort((a,b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf())
      type Result = {title: string,total?: number,items: RecordItem[]}[]
      const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'),items:[newList[0]]}]
      for(let i=1;i<newList.length;i++){
        const current =  newList[i]
        const last = result[result.length -1]
       if(dayjs(last.title).isSame(dayjs(current.createdAt),'day')){
         last.items.push(current)
       } else{
         result.push({title: dayjs(current.createdAt).format('YYYY-MM-DD'), items: [current]})
       }

      }
      result.map(group => {
        group.total = group.items.reduce((sum,item) => sum + item.amount,0)
      })
      return result
    }

    tagString(tags: Tag[]) {
      const tagList = [];
      for (let i = 0; i < tags.length; i++) {
        tagList.push(tags[i].name);
      }
      return tags.length === 0 ? '无' : tagList.join(',');
    }

    beautify(time: string) {
      const day = dayjs(time);
      const now = dayjs();
      if (day.isSame(now, 'day')) {
        return '今天';
      } else if (day.isSame(now.subtract(1, 'day'), 'day')) {
        return '昨天';
      } else if (day.isSame(now.subtract(2, 'day'), 'day')) {
        return '前天';
      } else {
        return time;
      }
    }
  }
</script>

<style scoped lang="scss">
  ::v-deep {
    .type-tabs-item {
      background: #3EB575;
      height: 48px;
      color:#555;
      &.selected {
        color: white;

        &::after {
          display: none;
        }
      }
    }
  }

  %item {
    padding: 8px 16px;
    line-height: 24px;
    display: flex;
    justify-content: space-between;
    align-content: center;

  }

  .title {
    @extend %item;
    background: lighten(#F5F5F5,1.5%);
    font-weight: 400;
    color:darken(#54BD85,15%);
  }

  .record {
    background: lighten(#F5F5F5,1.5%);
    @extend %item;

    .staticsNotes {
      margin-right: auto;
      margin-left: 16px;
      color: #999;
    }
  }

</style>