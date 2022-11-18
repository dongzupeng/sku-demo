<template>
  <modal v-if="show" :title="`${model.id ?'编辑' :'新增'}商品`" :show.sync="show" width="1200px" @ok="submit">
    <el-form ref="form" :model="model" label-width="120px">
      <el-form-item label="名称" prop="name" required>
        <el-input v-model="model.name"></el-input>
      </el-form-item>
      <el-form-item label="描述">
        <tinymce-editor v-model="model.description"></tinymce-editor>
      </el-form-item>
      <el-row>
        <el-col :span="8">
          <el-form-item label="类型" prop="type" required>
            <constant-select :constant="$constant.CoinMarketGoodsType" v-model="model.type" :disabled="!!model.id"></constant-select>
          </el-form-item>
        </el-col>
        <el-col :span="8" v-if="this.model.type === $constant.CoinMarketGoodsType.REALITY">
          <el-form-item label="多规格商品">
            <el-switch v-model="model.multipleSpecs"></el-switch>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="会员专属商品">
            <el-switch v-model="model.belongToCplus"></el-switch>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8" v-if="model.type === $constant.CoinMarketGoodsType.DOWNLOAD_TIMES">
          <el-form-item label="下载次数" prop="downloadTimes" required>
            <el-input-number :controls="false" :min="1" v-model="model.downloadTimes"></el-input-number>
          </el-form-item>
        </el-col>
        <el-col :span="8" v-if="model.type === $constant.CoinMarketGoodsType.HEADWEAR">
          <el-form-item label="挂件" prop="headwearId" required>
            <remote-select url="/resource/headwear/select" v-model="model.headwearId">
              <div class="headwear-select-item" slot-scope="{item}">
                <div class="name">{{ item.name }}</div>
                <img :src="$img(item.image,200,200) | fullurl"/>
              </div>
            </remote-select>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <app-form-item prop="coinPrice" required tips="兑换需要消耗的C币数量" label="兑换C币量">
            <el-input-number :controls="false" :min="1" v-model="model.coinPrice"></el-input-number>
          </app-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8">
          <el-form-item label="是否启用" prop="enabled">
            <el-switch v-model="model.enabled"></el-switch>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <app-form-item label="最少抵扣钻石量" label-width="140px" prop="minDeductDiamondCount"
                         tips="用户假如需要使用钻石抵扣C币，那么最少需要多少钻石抵扣，为0代表不可以用钻石抵扣">
            <el-input-number v-model="model.minDeductDiamondCount" :min="0" :controls="false"></el-input-number>
          </app-form-item>
        </el-col>
        <el-col :span="8">
          <app-form-item label="钻石抵扣汇率" prop="diamondDeductRate" tips="1钻石可以抵扣这个商品多少C币">
            <el-input-number v-model="model.diamondDeductRate" :min="1" :controls="false"></el-input-number>
          </app-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8">
          <app-form-item prop="cost" required tips="兑换需要支付的平台成本" label="财务成本">
            <el-input v-model="model.cost">
              <template slot="append">元</template>
            </el-input>
          </app-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="库存数量" prop="stockCount" required>
            <el-input-number :controls="false" :min="0" v-model="model.stockCount"></el-input-number>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item prop="exchangeLimitCount" required>
            <span slot="label">
              兑换限制
              <el-tooltip content="限制单个用户最多允许兑换次数，为0则不限制（挂件每个用户仅可以兑换一次）"><i class="el-icon-question"></i></el-tooltip>
            </span>
            <el-input-number :controls="false" :min="0" v-model="model.exchangeLimitCount"></el-input-number>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item prop="sortNum">
            <span slot="label">
              排序号
              <el-tooltip content="商城展现的时候会根据排序号从大到小排列"><i class="el-icon-question"></i></el-tooltip>
            </span>
            <el-input-number :controls="false" :min="0" v-model="model.sortNum"></el-input-number>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="商品图" prop="imageUrls" required>
        <multi-image-upload v-model="model.imageUrls" token-path="/file/image/resource_token"></multi-image-upload>
      </el-form-item>
      <el-form-item label="商品规格" required v-if="this.model.multipleSpecs">
        <el-table
          border
          :data="spec"
          style="width: 90%"
          max-height="250">
          <el-table-column
            type="index"
            label="序列"
            width="50">
          </el-table-column>
          <el-table-column
            label="规格名称"
          >
            <template slot-scope="{ row }">
               <el-input v-model="row.specName" placeholder="请输入"></el-input>
            </template>
          </el-table-column>
          <el-table-column
            v-for="i in 10"
            :key="i"
            :label="`规格值${i}`"
          >
            <template slot-scope="{ row }">
               <el-input v-model="row.specList[i-1]" @input="onSpecInput(row.specList[i-1],i-1,row.specList)" placeholder="请输入"></el-input>
            </template>
          </el-table-column>
          <el-table-column
            fixed="right"
            label="操作"
          >
            <template slot-scope="{ $index }">
               <el-button type="danger" @click="delspec($index)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="add-spec">
          <el-button type="primary" @click="addspec()">添加规格</el-button>
        </div>
        <el-table
          border
          :data="specData"
          style="width: 90%"
          max-height="500"
        >
          <el-table-column
            type="index"
            label="序列"
            width="50">
          </el-table-column>
          <el-table-column
            v-for="(item, index) in spec"
            :key="`${item.specName}_${index}`"
            :label="item.specName || '#'"
          >
            <template slot-scope="{ row }">
               {{getSpecValue(row, item.specName)}}
            </template>
          </el-table-column>

          <el-table-column
            label="兑换C币量"
          >
            <template slot-scope="{ row }">
               <el-input v-model="row.coinPrice" placeholder="请输入"></el-input>
            </template>
          </el-table-column>
          <el-table-column
            label="财务成本（元）"
          >
            <template slot-scope="{ row }">
               <el-input v-model="row.cost" placeholder="请输入"></el-input>
            </template>
          </el-table-column>
          <el-table-column
            label="库存数量"
          >
            <template slot-scope="{ row }">
               <el-input v-model="row.stockCount" placeholder="请输入"></el-input>
            </template>
          </el-table-column>
          <el-table-column
            label="规格商品图"
          >
            <template slot-scope="{ row }">
              <image-upload v-model="row.imageUrl"></image-upload>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
    </el-form>
  </modal>
</template>

<script>

import TinymceEditor from "../../components/common/TinymceEditor";

export default {
  name: 'GoodsEditor',
  components: {TinymceEditor},
  data() {
    return {
      model: null,
      show: false,
      spec: [],
      specData: []
    }
  },
  watch: {
    spec: {
      deep: true,
      handler(){
        this.generatespecData()
      }
    }
  },
  mounted() {
    this.generatespec()
  },
  methods: {
    async start(id) {
      if (id) {
        const {data} = await this.$rest.post('/coin/coinmarketgoods/get', {id})
        this.model = data
        this.specData = data.skuList
        if (this.specData) {
          this.spec = []
          let specNameArr = []
          this.specData.forEach(itm => {
            itm.specList.forEach(it => {
              if (specNameArr.indexOf(it.specName) === -1) {
                specNameArr.push(it.specName)
              }
            })
          })
          specNameArr.forEach(it => {
            let specValueArr = []
            this.specData.forEach(i => {
              i.specList.forEach(y => {
                if (it === y.specName && specValueArr.indexOf(y.specValue) === -1) {
                  specValueArr.push(y.specValue)
                }
              })
            })
            let spec = {specName: it, specList:specValueArr}
            this.spec.push(spec)
          });
        }else {
          this.specData = []
          this.spec = []
        }
      } else {
        this.model = {
          name: '',
          coinPrice: 100,
          stockCount: 0,
          exchangeLimitCount: 0,
          downloadTimes: 1,
          headwearId: null,
          enabled: true,
          description: '',
          type: null,
          sortNum: 0,
          cost: 0,
          minDeductDiamondCount: 0,
          diamondDeductRate: 2,
          imageUrls: [],
          multipleSpecs: false,
          belongToCplus: false
        }
      }
      this.show = true
    },
    async submit() {
      await this.$refs.form.validate()
      if (!this.model.imageUrls.length) {
        return this.$msg('必须至少上传一个商品图')
      }
      this.model.skuList = this.specData
      await this.$cf('是否确认提交')
      await this.$rest.post('/coin/coinmarketgoods/createOrUpdate', this.model)
      this.$successMsg()
      this.$emit('success')
      this.show = false
    },
    getSpecValue(row, name) {
      const i = row.specList.find(i=> i.specName === name)
      return i ? i.specValue : ''
    },
    generatespec() {
      let arr = [];
      this.spec = this.specData.reduce((arr, cur) => {
        for (let i = 0; i < cur.specList.length; i++) {
          const item = cur.specList[i]

          const has = arr.find(i=>i.specName === item.specName)
          if(!has) {
            arr.push({
              specName: item.specName,
              specList: [item.specValue]
            })
            return arr
          }
          if(!has.specList.includes(item.specValue)) {
            has.specList.push(item.specValue)
          }
        }
        return arr
      }, arr)
    },
    generatespecData() {
      if(!this.spec.length) {
        return
      }
      const check = () => {
        const specNameArr = this.spec.filter(i=>i.specName).map(i=>i.specName)
        let str = specNameArr.join(",") + ",";
        for(let i=0; i<specNameArr.length; i++){
            if(str.replace(specNameArr[i]+",", "").indexOf(specNameArr[i]+",") > -1){
                this.$msg(`规格名称${specNameArr[i]}已存在`)
                return false
            }
        }
        return true
      }

      if(!check()) return

      const arr = this.spec.reduce((t, i)=>{
        let specList = i.specList.filter(i=>i)
        if(!specList.length) {
          return t
        }
        const list = specList.map(v=>{
          return {
            specName: i.specName,
            specValue: v
          }
        })
        t.push(list)
        return t
      }, [])

      const res = this.handleCartesianProduct(arr)
      for (let i = 0; i < res.length; i++) {
        // 直接覆盖
        if(this.specData[i]) {
          this.specData[i].specList = res[i]
        } else {
          // 新增数据
          this.specData.push({
            cost: '',
            stockCount: '',
            coinPrice: '',
            imageUrl: '',
            specList: res[i]
          })
        }
      }
      // 保持数据长度一致
      this.specData = this.specData.splice(0, res.length)
    },
    handleCartesianProduct(array = []) {
      if(!array.length) return []
      if (Object.prototype.toString.call(array) === '[object Array]' && array.length < 2) return [...array[0].map(el => [el])];
      return array.reduce((col, row) => {
        const res = [];
        col.forEach((c) => {
          row.forEach((r) => {
            const t = [].concat(Array.isArray(c) ? c : [c]);
            t.push(r);
            res.push(t);
          });
        });
        return res;
      });
    },
    delspec(index) {
      this.spec.splice(index, 1)
      if (this.spec.length <= 0) {
        this.specData = []
      }
    },
    addspec() {
      this.spec.push({
        specName: '',
        specList: [],
      })
    },
    onSpecInput(val, index, specList) {
      let specValueArr = Object.assign([], specList)
      for (let i = 0; i < specValueArr.length; i++) {
        if (val === specValueArr[i] && i !== index){
          this.$msg("规格值"+val+"重复")
        }
      }
    }
  }
}
</script>

<style lang="scss">
.headwear-select-item {
  display: flex;
  align-items: center;
  height: 100%;

  .name {
    flex: 1;
  }

  img {
    height: 100%;
  }
}
.add-spec {
  text-align: center;
  margin: 30px 0;
}
</style>
