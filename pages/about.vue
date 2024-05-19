<template>
  <el-form :model="ruleForm" :rules="rules" label-width="auto" class="demo-ruleForm" status-icon :inline="true" style="margin: 30px 0px">
    <el-form-item label="地區" prop="city">
      <el-select v-model="selectedCity" placeholder="Select" size="large" style="width: 240px" filterable>
        <el-option v-for="item in cityOptions" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="日期" prop="Date">
      <el-select v-model="filterDate" multiple filterable size="large" placeholder="Select" style="width: 240px">
        <el-option v-for="item in dateOptions" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>

    <!-- <el-button type="primary" @click="submitData('ruleFormRef')">送出</el-button> -->
  </el-form>
  <div class="resultNotChunkList">
    <el-table :data="showData" style="width: 100%">
      <el-table-column prop="CountyName" label="行政區" width="auto" />
      <el-table-column prop="Date" label="日期" width="auto" />
      <el-table-column prop="SunRiseTime" label="日出時間" width="auto" />
      <el-table-column prop="SunSetTime" label="日落時間" width="auto" />
    </el-table>
    <el-pagination layout="prev, pager, next" :total="totalDataCount" :page-size="pageSize"
      @current-change="handleCurrentPage" />
  </div>
</template>

<script>
import { ref, reactive, onMounted, watch } from 'vue';

export default {
  setup() {
    const selectedCity = ref('臺北市');
    const cityOptions = [
      { value: '宜蘭縣', label: '宜蘭縣' },
      { value: '臺北市', label: '臺北市' },
      { value: '臺中市', label: '臺中市' },
      { value: '花蓮縣', label: '花蓮縣' },
      { value: '臺東縣', label: '臺東縣' },
      { value: '澎湖縣', label: '澎湖縣' },
      { value: '金門縣', label: '金門縣' },
      { value: '連江縣', label: '連江縣' },
      { value: '新北市', label: '新北市' },
      { value: '桃園市', label: '桃園市' },
      { value: '臺南市', label: '臺南市' },
      { value: '高雄市', label: '高雄市' },
      { value: '基隆市', label: '基隆市' },
      { value: '新竹縣', label: '新竹縣' },
      { value: '新竹市', label: '新竹市' },
      { value: '苗栗縣', label: '苗栗縣' },
      { value: '彰化縣', label: '彰化縣' },
      { value: '南投縣', label: '南投縣' },
      { value: '雲林縣', label: '雲林縣' },
      { value: '嘉義縣', label: '嘉義縣' },
      { value: '嘉義市', label: '嘉義市' },
      { value: '屏東縣', label: '屏東縣' },
    ];

    const filterDate = ref('');
    const dateOptions = ref(null);
    const APIResponse = ref(null)
    
    const fetchDateOptions = async () => {
      try {
        const response = await fetch(`https://opendata.cwa.gov.tw/api/v1/rest/datastore/A-B0062-001?Authorization=CWA-F8E54806-1CEA-489F-A531-83972F4E5DA8&limit=30000`);
        const data = await response.json();
        APIResponse.value = data.records.locations.location;
        const d  = APIResponse.value[0].time.map(timeObject=> {
          return {
            value: timeObject.Date,
            label: timeObject.Date
          }
        })
        dateOptions.value = d;
        totalDataCount.value = d.length;
      } catch (error) {
        console.error('Error fetching date options:', error);
      }
    };

    const alreadyChunkList = ref(null);

    const ruleForm = reactive({
      city: '',
    });

    const rules = reactive({
      city: [
        { required: true, message: '請選擇城市', trigger: 'change' }
      ]
    });

    const tableData = ref([]);
    const resultNotChunkList = ref([]);
    const showData = ref([]);
    const currentPage = ref(1);
    const pageSize = ref(20);
    const totalDataCount = ref(0);


    const handleCurrentPage = (page) => {
      currentPage.value = page;
      const pageIndex = page - 1
      showData.value =  alreadyChunkList.value[pageIndex]
    };

    const ruleFormRef = ref(null);
    const filterHandler = (value, row) =>  {
      console.log(value, row,'www')
      return row.name === value;
    }
    onMounted(() => {
      fetchDateOptions();
    });

    watch(selectedCity, () => {
      const resultItem = APIResponse.value.find(location => location.CountyName === selectedCity.value);
      const result = resultItem.time.map(item => { 
        return {
          ...item,
          CountyName:resultItem.CountyName
        } 
      })
      resultNotChunkList.value = result;
      alreadyChunkList.value = chunkArray(result, 20);
      showData.value = alreadyChunkList.value[0];
    });
    watch(filterDate, () => {
      const result = resultNotChunkList.value.filter(data => {
        let isMatch = false;
        if(filterDate.value.includes(data.Date)) {
          isMatch = true
        }
        return isMatch
      });
      alreadyChunkList.value = chunkArray(result, 20);
      showData.value = alreadyChunkList.value[0];
    });

    const chunkArray = (array, chunkSize) => {
        if (chunkSize <= 0) {
            throw new Error("chunkSize should be greater than 0");
        }
        
        const result = [];
        for (let i = 0; i < array.length; i += chunkSize) {
            result.push(array.slice(i, i + chunkSize));
        }
        return result;
    };


    return {
      filterHandler,
      filterDate,
      alreadyChunkList,
      selectedCity,
      cityOptions,
      dateOptions,
      ruleForm,
      rules,
      tableData,
      resultNotChunkList,
      showData,
      currentPage,
      pageSize,
      totalDataCount,
      handleCurrentPage,
      ruleFormRef,
    };
  },
};
</script>

<style scoped>
.location {
  display: flex;
  text-align: center;
  margin: 20px auto 50px auto;
}
</style>