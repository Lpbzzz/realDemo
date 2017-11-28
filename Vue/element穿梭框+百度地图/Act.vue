<template>
    
    <el-row>
        <el-col :span="9">
            <el-button type="primary" id="btn_main" @click="submit_result">提交结果</el-button>
            <div class="grid-content bg-purple-light">
                <!-- 貌似做不到 穿梭到右面之后 按照穿梭顺序排序，留待研究 -->
                <el-transfer 
                    v-model="value1" 
                    filterable 
                    :titles="['未选中的道路', '已选道路']"
                    :button-texts="['移除', '添加']" 
                    :format="{
                        noChecked: '${total}',
                        hasChecked: '${checked}/${total}'
                        }"
                    :props="{
                        key: 'value',
                        label: 'desc'
                    }"
                    @change="handleChange" :data="data">
                    <!-- <el-button class="transfer-footer" slot="left-footer" size="small">操作</el-button>
                    <el-button class="transfer-footer" slot="right-footer" size="small">操作</el-button> -->
                </el-transfer>
            </div>
        </el-col>
        <!-- <el-col :span="3">
             <ul v-for="(value ,key ,index) in items_right">
                <li class="lis">{{value.desc}}</li>
            </ul>
             <el-table :data="items_right" style="width: 100%">
                <el-table-column
                    prop="desc"
                    label="选中道路列表"
                    width="200">
                </el-table-column>
                
           </el-table>
        </el-col> -->
        <el-col :span="15">
            <div class="grid-content bg-purple-light">
                <baidu-map class="map" :center="{lng: 116.450, lat: 39.935}" :zoom="14" :scroll-wheel-zoom="true">
                    <!-- <bm-label content="我爱北京天安门" :position="{lng: 116.404, lat: 39.915}" :labelStyle="{color: 'red', fontSize : '12px' ,paddingRight:'150px'}" title="Hover me"/> -->
                    <div v-for="(val,key,index) in items_right ">
                        <bm-polyline :path="val.polylinePath" stroke-color="red" :stroke-opacity="1" :stroke-weight="3" :editing="false" @click="alerta"></bm-polyline>
                        <!-- <bm-circle :center="circlePath.center" :radius="circlePath.radius" stroke-color="blue" :stroke-opacity="0.5" :stroke-weight="2" @lineupdate="updateCirclePath" :editing="true"></bm-circle> -->
                        
                    </div>
                </baidu-map>
            </div>
        </el-col>
    </el-row>
</template>


<!-- <script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=hyQFaoLvOPWBThrUIVB2VjjD10Rq6b32"></script> -->

<script>
// import { MP } from './map'
export default {
  data() {
    // const assign_data = () =>{
    //     let data_new = [];
    //     Object.assign(data_new,this.data);
    //     return data_new;
    // },
    return {
      center: {
        lng: 0,
        lat: 0
      },
      zoom: 3,
      a: 0.01,
      circlePath: {
        center: {
          lng: 116.414,
          lat: 39.915
        },
        radius: 500
      },
      data: [
        {
          value: 111,
          desc: "阜成门南大街1",
          polylinePath: [
            {
              lng: 116.404,
              lat: 39.915
            },
            {
              lng: 116.405,
              lat: 39.92
            }
          ]
        },
        {
          value: 222,
          desc: "阜成门西大街2222", 
          polylinePath: [
            {
              lng: 116.414,
              lat: 39.925
            },
            {
              lng: 116.415,
              lat: 39.93
            }
          ]
        },
        {
          value: 333,
          desc: "阜成门北大街3",
          polylinePath: [
            {
              lng: 116.424,
              lat: 39.935
            },
            {
              lng: 116.425,
              lat: 39.94
            }
          ]
        },
        {
          value: 444,
          desc: "阜成门东大街4",
          polylinePath: [
            {
              lng: 116.434,
              lat: 39.945
            },
            {
              lng: 116.435,
              lat: 39.95
            }
          ]
        },
        {
          value: 555,
          desc: "月坛5",
          polylinePath: [
            {
              lng: 116.444,
              lat: 39.955
            },
            {
              lng: 116.445,
              lat: 39.96
            }
          ]
        }
      ],
      value1: [],
      items_right: [],
      items_left: [],
      tableData: [
        
      ]
    };
  },
  mounted() {
    this.changeStyle();
  },
  methods: {
    handleChange(value, direction, movedKeys) {
      this.items_right = [];
      let arr = [];
      this.value1.forEach(e => {
        this.data
          .filter(i => {
            return i.value == e;
          })
          .forEach(j => {
            this.items_right.push(j);
          });
      });
      value.forEach(e => {
        this.data = this.del(this.data, e);
      });
    },
    alerta(val) {
      console.log(val);
    },
    updateCirclePath(e) {
      this.circlePath.center = e.target.getCenter();
      this.circlePath.radius = e.target.getRadius();
    },
    del(arr1, piovt) {
      let arrTarget = [];
      let arrLeft = [];
      let arrRight = [];
      for (let j = 0; j < arr1.length; j++) {
        // console.log(arr1[j].value)
        if (arr1[j].value == piovt) {
          arrLeft.push(arr1[j]);
        } else {
          arrRight.push(arr1[j]);
        }
      }
      arrTarget = arrRight.concat(arrLeft);
      return arrTarget;
    },
    changeStyle() {
      $(".el-transfer__buttons :button")
        .eq(1)
        .insertBefore($(".el-transfer__buttons :button").eq(0));
    },
    submit_result(){
        console.log(this.items_right)
    }
  }
};
</script>

<style>
.transfer-footer {
  margin-left: 20px;
  padding: 6px 5px;
}

.el-row {
  margin-bottom: 20px;
}

.el-col {
  border-radius: 4px;
  height: 700px;
}

.bg-purple-light {
  height: 100%;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

.row-bg {
  padding: 10px 0;
}

.el-transfer-panel {
  height: 80%;
}

.map {
  width: inherit;
  height: inherit;
}

.el-transfer {
  height: inherit;
}

.el-transfer-panel {
  height: 100%;
  margin-right: 20px;
}

.el-transfer-panel__list.is-filterable {
  height: 90%;
}

.el-transfer-panel__body {
  height: inherit;
}

.el-transfer__buttons {
  margin-right: 20px;
}
.lis {
  list-style: none;
}
.el-col-3 {
  margin-right: 60px;
  margin-left: -20px;
}
#btn_main{
    position: relative;
    top: 454px;
    left: 216px
}
</style>