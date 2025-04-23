<template>
<div class="home-container">
    <div class="home-header">
        <div class="header-title">ROBOTMAP</div>
        <div class="header-nav">
            <div class="upload" @click=" dialogCreateMaps=true">Create Map</div>
            <div class="maps"  @click="dialogShowMaps=true;getnameWork();">Your Maps</div>
            <div class="upload" >Create Route</div>
        </div>
    </div>
    <div class="home-main">
        <div id="cesiumContainer" ref="cesiumContainer">      
        </div>
    </div>

    <!-- 上传地图信息-->
    <el-dialog v-model="dialogCreateMaps" title="Create layer" width="600" >
        <el-form :model="form" :rules="rules" ref="formRef" v-loading="loading" @submit.prevent>
      <el-form-item label="layer name" label-width="140" prop="layerName">
        <el-input v-model="form.layerName" autocomplete="off" />
      </el-form-item>
      <el-form-item label="Upload File" label-width="140" ><el-upload ref="upload"
    v-model:file-list="fileList"
    class="upload-demo"
    :action="uploadUrl"
    :auto-upload="false"
    :on-exceed="handleExceed"
    :on-error="uploadFileError"
    multiple
    accept=".tif,.tiff"
  >
  
    <el-button type="primary">Click to upload</el-button>
    <template #tip>
      <div class="el-upload__tip">
        Only TIF files are allowed, no size limit.
      </div>
    </template>
  </el-upload>
      </el-form-item>
  <el-form-item>
      <el-button type="primary" @click="handleSuccess()">确认提交</el-button>
      <el-button @click="resetForm">重置</el-button>
    </el-form-item>
    </el-form>
  </el-dialog>



        <!-- 展示地图图层信息-->
    <el-dialog v-model="dialogShowMaps" title="Show layers" width="600">
      <el-table :data="tableMaps"  style="width: 100% ;height: 90vh;" 
      @current-change="handleSelecedLayer"
      highlight-current-row>
    <el-table-column prop="date" label="Date" />
  </el-table>
  <el-button color="#082631" style="margin-top: 20px;" @click="loadLayers">确认</el-button>
  <el-pagination size="small" layout="prev, pager, next" :total="5" />
    </el-dialog>

</div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { ElMessage, ElMessageBox } from 'element-plus'


import {getLayerName,getWorkName,uploadLayer,getWorkLocation} from '@/api/getLayer.js'
import { onMounted ,ref,reactive,watch,toRefs,nextTick} from 'vue';
import {handleExceed,uploadFileError} from '@/utils/fileUpload/fileUpload.js';

import{ getBaseURL} from '@/utils/request.js'

const baseUrl=getBaseURL();
//显示地图创建对话框
const dialogCreateMaps = ref(false);
const dialogShowMaps = ref(false);
const viewer=ref(null)
//所填写的表格
const form = reactive({
    layerName: '',
    storename:'',
    workspace:'',

})
const tableMaps=ref([]);
const { layerName, storename, workspace} = toRefs(form);
const showLayers=ref([])
const fileList = ref([]);

onMounted(() => {
  nextTick(()=>{
    console.log(import.meta.env.VITE_APP_BASE_URL,"基地址");
  })
  getnameWork();
  console.log(getBaseURL());
  // 初始化 Cesium 视图器
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI3ZDY2YzAxMS02Nzc4LTQyZGMtYjU5MC0xNDIyNDc2MzZlODYiLCJpZCI6MjkxMjcxLCJpYXQiOjE3NDM5MjYxNDl9.U0mgwiqQE22npBjqbrCYhAkVJIRzmw9eXgFndNNnJSQ'

  viewer.value = new Cesium.Viewer('cesiumContainer', {
    // 调整基础设置隐藏组件
    animation: false, // 动画小部件
    baseLayerPicker: false, // 地图图层组件
    fullscreenButton: false, // 全屏组件
    geocoder: false, // 地理编码搜索组件
    homeButton: false, // 首页组件
    infoBox: false, // 信息框
    sceneModePicker: false, // 场景模式
    selectionIndicator: false, // 选取指示器组件
    timeline: false, // 时间轴
    navigationHelpButton: false, // 帮助按钮
    navigationInstructionsInitiallyVisible: false,
     baseLayerPicker: false,
  })
})
const selectedLayer=ref("")
const handleSelecedLayer=(rowDate)=>{
  
  selectedLayer.value=rowDate.date;
  console.log("被选中行的数据",selectedLayer.value);

}
// 加载图层并跳转到指定坐标
// const loadLayers = () => {
//   dialogShowMaps.value=false
//   if (!viewer.value) {
//     ElMessage.error('Cesium 视图器未初始化')
//     return
//   }

//   // 跳转到指定坐标
//   // viewer.value.camera.flyTo({
//   //   destination: Cesium.Cartesian3.fromDegrees(114.3550, 30.4796, 800.0),
//   // })
//   viewer.value.imageryLayers.removeAll();
//   // 创建 imageryProvider
//     // 跳转到指定坐标
//   viewer.value.camera.flyTo({
//     destination: Cesium.Cartesian3.fromDegrees(22.5561, 110.2025, 1800.0),
//   })
//   var imageryProvider = new Cesium.WebMapServiceImageryProvider({
//     // 这里是你的 geoserver服务点击查看图层的 url
//     url: `http://202.140.140.215:9999/geoserver/${selectedLayer.value}/wms?`,
//     // customTags: {
//     //                 level: function(imageryProvider, x, y, z) {
//     //                     return z+1
//     //                 }
//     //             },

//     // 这里是自定义的图层名称
//     layers: `${selectedLayer.value}:${selectedLayer.value}`,
//     parameters: {
//       service: 'WMS',
//       format: 'image/png',
//       crs: 'EPSG:4326',
//       bbox:' 110.2479632005,22.461699393800004,110.89326136749999,23.1277073488',
//       // request: 'GetMap',
//       transparent: true,
//     },
//   })


//   // 图层添加
//   viewer.value.imageryLayers.addImageryProvider(imageryProvider)
//   console.log("加载");


// }
const loadLayers = () => {
  dialogShowMaps.value = false;

  if (!viewer.value) {
    ElMessage.error("Cesium 视图器未初始化");
    return;
  }

  const layerName = `${selectedLayer.value}:${selectedLayer.value}`;
  // const layerUrl = `http://202.140.140.215:9999/geoserver/${selectedLayer.value}/wms?`;
  const layerUrl = `http://47.122.123.251:8233/geoserver/${selectedLayer.value}/wms?`;
  // const bbox = "114.348977902349,30.528903163361175,114.35036065924551,30.530418382267538"; 
  // **检查是否已存在相同的图层，避免重复加载**
  const layers = viewer.value.imageryLayers;
  for (let i = 0; i < layers.length; i++) {
    const existingProvider = layers.get(i).imageryProvider;
    if (existingProvider.url === layerUrl && existingProvider.layers === layerName) {
      console.log("图层已存在，无需重复加载");
      return;
    }
  }

  // **创建 imageryProvider**
  const imageryProvider = new Cesium.WebMapServiceImageryProvider({
    url: layerUrl,
    layers: layerName,
    parameters: {
      service: "WMS",
      format: "image/png",
      srs: "EPSG:4326",  // 确保 GeoServer 兼容
      transparent: true,
      styles: "",
      version: "1.1.1",  // 确保 WMS 兼容性
      // bbox:bbox,
    },
    tileWidth: 256,  // 优化瓦片加载
    tileHeight: 256,
  });

  // **添加图层**
  layers.addImageryProvider(imageryProvider);
  console.log("新图层加载成功");

  // **仅在首次加载时调整视角**
  if (!viewer.value.camera.flyToExecuted) {
    // viewer.value.camera.flyTo({
    //   // destination: Cesium.Cartesian3.fromDegrees(110.35036065924551, 22.53041838226753, 50000.0),
    //   destination: Cesium.Cartesian3.fromDegrees(114.35036065924551, 30.53041838226753, 50000.0),
    // });//114 30
    viewer.value.camera.flyTo({
  destination: Cesium.Rectangle.fromDegrees(
    114.348977902349,
    30.528903163361175,
    114.35036065924551,
    30.530418382267538
  )
});

    viewer.value.camera.flyToExecuted = true; // 标记已执行
  }
  // selectedLayer.value=""
};

// const getLocation=(name)=>{
  
// }

// const get =async(form)=>{
//   const res =await uploadLayer(form);

//     if(res.code==200){
//       console.log("操作成功");
//       ElMessage.success("操作成功")
//     }
//     else {
//       ElMessage.error(res.msg);
//     }
    
// };
const loading = ref(false)
const getnameWork =async ()=>{
  const res =await getWorkName();
    // tableMaps.value=res.data
    if(res.code==200){
      console.log("tableMaps",tableMaps.value);
      ElMessage.success("操作成功")
      tableMaps.value=res.data.map((item)=>{
        return {
          date:item
        }
      })

      console.log("处理后的地图表",tableMaps.value);
    }
    else if(res.code==400){
      ElMessage.error(res.msg);
    }
    
};

// **表单验证规则**
const rules = {
  layerName: [
    { required: true, message: 'Layer Name 不能为空', trigger: 'blur' },
    
    { pattern: /^[a-zA-Z0-9]+$/, message: 'Layer Name 只能包含英文字符和数字', trigger: 'blur' }
  ],
};
// **重置表单**
const resetForm = () => {
  formRef.value.resetFields();
  console.log("表单重置成功");
};

//文件处理
const formRef = ref(null); 

  // 上传实例
  const upload = ref(null);
  // 文件上传
  const uploadUrl = ref('');
// const handleSuccess = (response, file, fileList) => {
//   console.log(fileList,"handleSuccess");
//   // fileLIst = fileList; // 确保fileList实时更新
//   // ElMessage.success(`${file.name} 上传成功！`);
//   // formRef.value.validateField('fileList'); // 手动触发 fileList 校验
// };

// let tempFiles=[];

  // 文件创建
const  createMap=()=>{
    formRef.value.validate(valid => {
      if(fileList.value.length==0){
        ElMessage.error("请同时提交地图文件")
      }
      else {
        if (valid) {
  ElMessage.warning("正在创建")
  console.log(import.meta.env.VITE_APP_BASE_URL);

        uploadUrl.value =
          `${baseUrl}/geoserver/postFile`;
        nextTick(async () => {
          // tableLoading.value = false;
          await upload.value.submit();
          
          // getList();
        });
      }
      }

    });
  }
// const submitForm = () => {
//   console.log(fileList,"提交表单1 "); 


//   nextTick(()=>{
//   console.log(fileList,"提交表单2"); 


//   //   formRef.value.validate((valid) => {
//   //   if (valid &&fileList.value.length!=0) {
//   //     console.log(fileList.value.length);
//   //     ElMessageBox.confirm('确定提交表单？', '提示', {
//   //       confirmButtonText: '确定',
//   //       cancelButtonText: '取消',
//   //       type: 'warning'
//   //     })
//   //       .then(() => {
//   //         homeUploadLayer();

//   //         console.log('提交的表单数据:', fileList);
//   //         ElMessage.success('提交成功！');

//   //       })
//   //       .catch(() => {
//   //         ElMessage.info('取消提交');
//   //       });
//   //   } else if(fileList.value.length==0){
//   //     console.log("sokacio");
//   //   console.log(fileList.value.length,"sokacio");

//   //     // console.log();
//   //     // ElMessage.error('请检查表单输入');
//   //   }
//   // });
//   })



// };

const  handleSuccess=async(response)=> {
    // const file= response.url;
    const file = '/var/www/uploads/feidian/test.tif';
    upload.value.clearFiles();
    loading.value = true;
  try{
    const res=await uploadLayer({ ...form, file });
    loading.value = false;
    ElMessage.success('地图创建成功');
    resetForm();
  }
  catch(e){
    console.log(e,'上传出错');
  }

      
    
    // if(res.code==200){
    //   console.log("操作成功");
    //   ElMessage.success("操作成功")
    // }
    // else if(res.code==400){
    //   ElMessage.error(res.msg);
    // }
    // getList();
  }

watch(layerName,()=>{
  workspace.value=layerName.value;
  storename.value=layerName.value;
  console.log(storename.value);
  // handleSuccess();
});


</script>

<style lang="scss" scoped>
//删除cesium的版权信息
:deep(.cesium-widget-credits){
        display: none !important;
      }


#cesiumContainer{
    height: 90vh;
}
.home-header{
    background-color: #082631;
    // width: 100vh;\
    height: 10vh;
    display: flex;
    justify-content:space-between;

    // overflow-y: hidden !important;
.header-title{
    // flex: 1;
    font-size: 3rem;
    font-style: italic;
    color: #fff;
    margin-left: 2vw;
    text-shadow: 3px 3px 8px rgba(255, 255, 255);
}
.header-nav{
    font-size: 1.8rem;
    font-family: Avenir, Helvetica, Arial, sans-serif;
    // font-size: 2rem;
    text-align: center;
    color: #fff;
    display: flex;
    .maps,
    .upload{
        width: 200px;
        // height: 10px;
        height: 100%;
        display: grid;
    place-items: center;
    margin-left: 10px;
    cursor: pointer;
    &:hover{
        // background-color: pink;
        color: #9BC5EF;

    }
    }
    
}
}
.home-main{
    background-color: pink;
    height: 2000px;
}


</style>