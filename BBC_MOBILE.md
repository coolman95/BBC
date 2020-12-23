# 手机端页面页面实现一键创建订单，提交，审核，发货操作

## 添加表单页面

> 页面实现axios请求,分页加载，弹窗提示，弹窗列表，选择数据，排版，页面初始化等...
```
<template>
    <div class="o1">
        <div v-show="loading1" style="position: fixed;left: 47%;top: 38%;">
            <van-loading type="spinner" color="#1989fa" />
        </div>
        <div style="line-height: 56px;background-color: #fff; font-size: 16px;font-weight: 1000">
            <van-icon name="arrow-left" style="padding: 20px;float: left;position: absolute;" @click="go"/>
            <div style="text-align: center">创建发货订单</div>
        </div>
        <div class="details-box">
            <van-cell-group>
                <van-field v-model="customer.FCONTACT" label="客户" right-icon="warning-o" placeholder="请选择客户" @click="showPopup()"  required/>
                <van-field v-model="customer.FTELPHONE" label="电话" type="tel" placeholder="" readonly />
                <van-field v-model="customer.defAddress" label="地址" placeholder="" readonly />
                <van-action-sheet v-model="cusShow" :actions="supplierList" @select="onSelect" />
            </van-cell-group>
            <div class="uploader" v-show="0">
                <van-cell title="上传附件" value=""/>
                <van-uploader v-model="attachment" multiple />
            </div>
            <div class="details-list">
            </div>
        </div>
        <van-action-sheet v-model="goodsShow" title="">
            <div class="searchHeader">
                <van-search v-model="params.keyword" @search="onLoad(1)" placeholder="请输入搜索关键词" />
                <van-radio-group v-model="isChecked">
                    <van-button type="info" @click="checkAll(0)" size="mini" style="float: left;margin: 3px 12px 12px 12px;">反选</van-button>
                    <van-button type="info" @click="selectGoods" size="small" style="float: right;margin: 0px 12px 12px 0px;">选择</van-button>
                </van-radio-group>
            </div>
            <van-divider />
            <van-list v-model="loading" @load="onLoad(0)" :finished="finished" finished-text="没有更多了" error-text="请求失败，点击重新加载">
                <van-checkbox-group v-model="goods" ref="checkboxGroup">
                    <van-checkbox v-for="(item,i) in goodsList" :key="item.goodsId" :name="item.goodsId" :value="item.goodsId" style="padding: 0 12px">
                        <p :style="{'padding-top':(i == 0 ? '90px' : '')}">
                        <van-cell title="产品名称" :value="item.goodsName"/>
                        <div class="first-line">
                            <div class="attr">编号：{{item.goodsNumber}}</div>
                            <div class="attr">规格：{{item.specification}}</div>
                        </div>
                        </p>
                        <van-divider />
                    </van-checkbox>
                </van-checkbox-group>
            </van-list>
        </van-action-sheet>
        <div class="details-list">
            <div class="details-list-1" v-for="(list,index) in product_lists" :key="index">
                <van-cell title="产品名称" :value="list.goodsName"/>
                <p>
                <div class="first-line">
                    <div class="attr">编号：{{list.goodsNumber}}</div>
                    <div class="attr">规格：{{list.specification}}</div>
                </div>
                <div class="first-line">
                    <div class="attr" style="line-height: 35px;">订单数量：<van-stepper v-model="list.value" style="float: right"/> </div>
                    <div class="attr" style="line-height: 35px;">库存：{{parseFloat(list.stockqty)}}</div>
                </div>
                </p>
            </div>

            <div v-show="0" class="details-list-2">
                <div class="details-list-2-2">
                    <p>
                        <span class="span-1">订单编号：</span>
                        <span class="span-2"></span>
                    </p>
                    <p>
                        <span class="span-1">单据状态：</span>
                        <span class="span-2"></span>
                    </p>
                    <p>
                        <span class="span-1">订单备注：</span>
                        <span class="span-2"></span>
                    </p>
                </div>
            </div>
        </div>
        <div class="submit-bar">
            <div class="leftBar" style="float: left;padding: 0.1rem;">
                <van-button type="info" @click="addGoods" >添加商品</van-button>
            </div>
            <div class="bar">
                <van-button @click="confirmSign" type="info">提交订单</van-button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";
import OrderHeader from "../common/header-sign";
import wx from 'weixin-js-sdk';
import Vue from 'vue';
import { Toast } from "mint-ui";
import { Dialog } from 'vant';
export default {
  data() {
    return {
        cusShow: false,
        goodsShow: false,
        product_lists: [],
        supplierList: [],
        attachment: [],
        customer: {id: "",FCONTACT: "", FTELPHONE: "", defAddress: ""},
        count: 0,
        loading: false,
        finished: false,
        goodsList: [],
        params:{keyword: "", page:1, size:10},
        goods: [],
        isChecked: false,
        discountGoods: [],
        word : "",
        loading1 : false,
    };
  },
  // components: {
  //   OrderHeader
  // },
  created() {
  },
  mounted()  {
      let _this = this;
      // _this.customer = _this.supplierList[0];
      // this.getSupplierList();
  },
    methods:{
        onLoad(tag) {
            // console.log(this.params, this.finished);
            if (tag === 1) {
                this.finished = false;
                this.params.page = 1;
                this.goodsList = [];
            }
            let _this = this;
            _this.loading1 = true;
            axios.get(window.urls.getGoodsList,{params: this.params}).then(function(res) {
                _this.loading = false;
                _this.loading1 = false;
                if (parseInt(res.data.errno) === 0) {
                    if (parseInt(res.data.data.count) === 0) {
                        _this.params.keyword = "";
                        Dialog({ message: '未找到数据!' });
                        return ;
                    }
                    ++_this.params.page;
                    res.data.data.list.forEach(function(item, index) {
                        item.name = item.FCONTACT;
                        _this.goodsList.push(item);
                    });
                    if(_this.goodsList.length >= res.data.data.count) {
                        _this.finished = true
                    }
                }
            });
            // console.log(this.params, this.finished);
        },
        confirmSign(){
            if (!this.customer.id){
                Dialog({ message: '请选择客户信息!' });
                return ;
            }
            if (this.product_lists.length === 0){
                Dialog({ message: '请添加商品信息!' });
                return ;
            }
            let goodsInfo = [];
            this.product_lists.forEach(function (item, index) {
                // goodsInfo.push({goodsId: item.goodsId, price: item.salePrice, num: item.value});
                goodsInfo.push(item);
            });
            let cus = this.customer.FCONTACT + " " + this.customer.FTELPHONE + " " + this.customer.defAddress;
            let postParam = {customerId: this.customer.id, customer: cus, cusobj: this.customer, goods: goodsInfo};
            let _this = this;
            _this.loading1 = true;
            axios.post(window.urls.consignmenOrder, postParam).then(function(res) {
                _this.loading1 = false;
                console.log(res);
                if (parseInt(res.data.errno) === 0) {
                    Dialog({ message: '操作成功' }).then(() => {
                        location.reload();
                    });
                } else {
                    Dialog({ message: res.data.message }).then(() => {
                        location.reload();
                    });
                }
            });
        },
        scan(){
        },
        getSupplierList(){
            let _this = this;
            _this.loading1 = true;
            axios.get(window.urls.getSupplierList,{params:{page:1, size:100}}).then(function(res) {
                _this.loading1 = false;
                if (parseInt(res.data.errno) === 0) {
                    res.data.data.list.forEach(function(item, index) {
                        item.name = item.FCONTACT;
                        _this.supplierList.push(item);
                    });
                    // _this.count = res.data.data.count;
                }
            });
        },
        showPopup() {
            this.cusShow = true;
            this.getSupplierList();
        },
        onSelect(item){
            this.customer = item;
            this.cusShow = false;
            let _this = this;
            _this.loading1 = true;
            axios.get(window.urls.getSuplierGoods,{params:{suplierId: item.id}}).then(function(res) {
                _this.loading1 = false;
                if (parseInt(res.data.errno) === 0) {
                    _this.discountGoods = res.data.data.list;
                } else {
                    Dialog({ message: '获取该客户折扣商品信息失败!' });
                }
            });
            console.log(item,_this.discountGoods);
        },
        addGoods(){
            this.goodsShow = true;
            // this.onLoad();
        },
        checkAll(n) {
            this.isChecked = !this.isChecked;
            this.$refs.checkboxGroup.toggleAll(n);
        },
        selectGoods() {
            let _this = this;
            // _this.product_lists = [];
            this.goodsList.forEach(function (item, index) {
                if (_this.goods.indexOf(item.goodsId) >= 0) {
                    _this.product_lists.push(item);
                }
            });
            this.goodsShow = false;
            let arr = [];
            let newGoodList = [];
            this.product_lists.forEach(function (item, index) {
                if (arr.indexOf(item.goodsId) === -1) {
                    arr.push(item.goodsId);
                    item.value = parseInt(item.value) > 1 ? item.value : 1;
                    newGoodList.push(item)
                }
            });
            _this.product_lists = newGoodList;
            this.findDiscount();
        },
        findDiscount(){
            // 为商品寻找商品折扣
            let _this = this;
            this.product_lists.forEach(function (item, index) {
                for (let i = 0 ; i<_this.discountGoods.length; i++) {
                    _this.product_lists[index].discountRate = (_this.discountGoods[i].FGOODID === item.goodsId) ? _this.discountGoods[i].FDISCOUNT : 100;
                }
                if (_this.discountGoods.length === 0) {
                    _this.product_lists[index].discountRate = 100;
                }
            });
        },
        go(){this.$router.go(-1)}
    },
};
</script>
<style lang="stylus" scoped>
    .header
        width 100%
        background white
        position fixed
        z-index 999
        display flex
        align-items center
        .barcode
            width 90%
        .header-left
            width 10%
            height 100%
            float left

        .header-in
            width: 80%;
            float: left;
            text-align: center;
            font-size: 0.45rem;
            line-height: 1.45rem;
        .header-rigth
            width 10%
            height 100%
            float left
            line-height:1.45rem

    .barcode >>> input{
        color:red;
    }
    .van-uploader{
        padding-left: 0.3rem;
    }
    .sign-number >>> input{
        border:0.02rem solid #26a2ff !important;
    }
    .barcode{
        margin-top: 0.1rem;
    }
    .uploader{
        background-color: white;
        margin-top: 0.1rem;
    }
    .submit-bar{
        position: fixed;
        bottom: 0rem;
        width: 100%;
        background-color: white;
        border-top: 0.01rem solid #f2f3f5;
    }
    .bar{
        float: right;
        padding: 0.1rem;
    }
    .attr-bottom{
        display: flex;
        align-items: center;
        padding-left:0.5rem;
    }
    .first-line{
        padding-top: 0.2rem;
        display: flex;
    }
    .attr{
        font-size:0.35rem;
        width: 50%;
        padding-left: 0.5rem;
        padding-bottom: 0.2rem;;
    }
.details-box {
    padding-top: 0.1rem;
}

.details-success {
    width: 100%;
    height: 2.4rem;
    background: #69c1ff;

    p {
        color: #fff;
        font-size: 0.46rem;
        line-height: 2.4rem;
        float: left;
        padding-left: 0.4rem;
    }

    img {
        width: 2.7rem;
        margin: 0.5rem;
        float: right;
    }
}

.details-list {
    width: 100%;
    padding-bottom: 0.3rem;
    .details-list-1 {
        width: 100%;
        /*height: 3rem;*/
        background: #ffffff;
        margin-top: 5px;

        img {
            width: 2.5rem;
            margin: 0.28rem;
            float: left;
        }

        p {
            display: flex;
            flex-direction: column;

            .name {
                font-size: 0.4rem;
                margin-top: 0.3rem;
                height: 0.6rem;
            }

            .price {
                color: red;
                font-size: 0.35rem;
                font-weight: 500;
                height: 0.6rem;
            }

            span {
                p {
                    float: right;
                    margin-right: 0.5rem;
                }
            }
        }
    }
    .details-list-2 {
        width: 100%;
        background: #ffffff;

        div {
            width: 90%;
            margin: auto;
            padding-top: 0.3rem;
            padding-bottom: 0.3rem;
            font-size: 0.36rem;
            border-top: 1px solid #e8e8e8;

            span {
                color: #888;
                float: left;
            }

            .span-1 {
                width: 2.8rem;
                display: block;
            }
        }

        .details-list-2-1 {
            height: 2rem;

            p {
                height: 0.76rem;
                line-height: 0.76rem;
            }
        }

        .details-list-2-2 {
            height: 3.5rem;
            p {
                height: 0.76rem;
                line-height: 0.76rem;
            }
            .red {
                color: red;
            }
        }
        .details-list-2-3 {
            height: 4rem;

            p {
                height: 0.76rem;
                line-height: 0.76rem;
            }
        }
    }

}
   /deep/  .van-checkbox__label {
        width: 90%;
    }
   /deep/  .searchHeader{
        position: fixed;
        width: 100%;
        z-index: 999;
       background-color: #fff;
    }
    /deep/ .van-checkbox:nth-child(1) .van-checkbox__icon{
        margin-top: 100px;
    }
</style>


```

> 底层 pdo 连接实现事务操作（原方法不支持事务）

```
    public function beginTransaction(){
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        $this->pdo->beginTransaction();
    }

    public function rollBack(){
        $this->pdo->rollBack();
    }

    public function commit(){
        $this->pdo->commit();
    }
    
```

### server层数据库操作，数据库字段维护

> 1. 需要三个数据列表

```PHP
    public function getSupplierList($custcode, $page=1, $size=10){
        if(empty($custcode)){
            return [];
        }
        $offset = ($page - 1) * $size;
        $sql = "select s.FID id,sl.FNAME secChannelName,s.FNUMBER,s.FTELPHONE,s.FCONTACT, s.FDEFRECADDRESS defAddress 
			from T_RD_SecSupplier s 
			inner join T_RD_SecSupplier_L sl on s.FID  = sl.FID and sl.FLOCALEID= 2052 
			inner join T_BD_CUSTOMER c on c.FCUSTID=s.FBELONGCUST 
            inner join T_ESS_CHANNEL ch on c.FMASTERID = ch.FCUSTOMERID 
            where s.FDOCUMENTSTATUS='C' and s.FFORBIDSTATUS='A' and ch.FNUMBER='{$custcode}' ";
        $this->db->query($sql . "ORDER BY id offset {$offset} row fetch next {$size} row only");
        $data = $this->db->fetchAll();
        // 获取总数
        $this->db->query("select count(1) as num from ({$sql}) as count");
        $count = $this->db->fetchColumn();
        return [$data, $count];
    }

// 获取关联客户折扣商品信息
    public function getPriceGoods($superlierId){
        if (empty($superlierId)) {
            return [];
        }
        $sql = "SELECT s.FID, pg.* FROM T_RD_SecSupplier s INNER JOIN T_ESS_PRICE_GOODS pg ON s.FMYPRICEID = pg.FGPRICEID WHERE s.FID = {$superlierId}";
        $this->db->query($sql);
        return $this->db->fetchAll();
    }

    // 获取客户商品信息
    public function getGoodsList($custcode, $page=1, $size=10, $keyword=""){
        if(empty($custcode)){
            return [];
        }
        $offset = ($page - 1) * $size;
        $date = date('Y-m-d');
        $sql = "select distinct gp.FCHANNEL,gp.FSTOCKQTY stockqty,g.FGOODSID goodsId,price.salePrice,gl.FNAME goodsName,g.FNUMBER goodsNumber,g.FDEFAULTPICADDR defaultPicAddr,g.FSTOCKUNITID stockUnitId,gl.FSPECIFICATION specification,g.FBASEUNITID baseUnitId,g.FBASEUNITID realUnit,bul.FNAME baseUnitName,g.FBASEUNITID unitId,bul.FNAME unitName,ol.FORGID orgId,ol.FNAME orgName,m.FMATERIALID materialId 
		    FROM (SELECT goods.FGOODSID, goods.FSERIESCODE, goods.FTYPEID FSTOCKID,(case when chagp.FCHANNELID is null then goods.FCHANNELID else chagp.FCHANNELID end) FCHANNEL, chagp.FSTOCKQTY
            FROM T_ESS_GOODS goods left join T_ESS_INVENTORY chagp on goods.FGOODSID = chagp.FGOODSID) gp
            left join T_ESS_GOODS g on g.FGOODSID = gp.FGOODSID
			left join T_ESS_GOODS_L gl on g.FGOODSID = gl.FGOODSID and gl.FLOCALEID = 2052
			left join T_BD_UNIT bu on g.FBASEUNITID = bu.FUNITID
			left join T_BD_UNIT_L bul on bul.FUNITID = bu.FUNITID and bul.FLOCALEID = 2052
			left join T_BD_MATERIAL m on g.FMATERIALID = m.FMATERIALID
			LEFT JOIN T_ESS_CHANNELSTOCK_L T3 ON T3.FID = gp.FSTOCKID and T3.FLOCALEID = 2052
            LEFT JOIN T_ESS_CHANNELSTOCK T6 ON T6.FID = gp.FSTOCKID
			left join T_ESS_GOODS_PUBLISH p on p.FGOODSID = g.FGOODSID and p.FCHANNEL = gp.FCHANNEL
			left join T_ESS_CHANNEL c on p.FCHANNEL = c.FCHANNELID
			left join T_ORG_ORGANIZATIONS o on p.FORGID = o.FORGID
			left join T_ORG_ORGANIZATIONS_L ol on o.FORGID = ol.FORGID and ol.FLOCALEID = 2052 
			left join ( select p.FPRICE salePrice, g.FGOODSID  from  T_SAL_PRICELISTENTRY p  
            left join T_SAL_APPLYCUSTOMER a on p.FID  = a.FID
            INNER JOIN T_BD_CUSTOMER CU ON CU.FCUSTID = a.FCUSTID 
            INNER JOIN T_BD_MATERIAL M ON M.FMATERIALID  = p.FMATERIALID 
            inner join t_BD_MaterialSale ms on  M.FMATERIALID = ms.FMATERIALID 
            INNER JOIN T_ESS_GOODS g ON M.FMASTERID = g.FMATERIALID 
            INNER JOIN T_ESS_CHANNEL d ON CU.FMASTERID = d.FCUSTOMERID 
            where '{$date}'<=p.FEXPRIYDATE and '{$date}'>=p.FEFFECTIVEDATE 
            and p.FFORBIDSTATUS = 'A' and d.FNUMBER='{$custcode}') price ON price.FGOODSID = g.FGOODSID WHERE c.FNUMBER='{$custcode}' " ;
        $sql .= empty($keyword) ? "" : " and (gl.FNAME like '%{$keyword}%' or g.FNUMBER like '%{$keyword}%' or gl.FSPECIFICATION like '%{$keyword}%') ";

        $this->db->query($sql . " ORDER BY goodsId offset {$offset} row fetch next {$size} row only");
        $data = $this->db->fetchAll();

        // 获取总数
        $this->db->query("select count(1) as num from ({$sql}) as count");
        $count = $this->db->fetchColumn();

        return [$data, $count];
    }
    
```

> 2. BBC 数据库自增类和编码类

```

/**
     * 获取数据表字段最大ID
     * @param $tablename
     * @param string $feildID
     * @return mixed
     */
    private function getMaxID($tablename)
    {
        $keyTable = 'Z_'.substr($tablename, 2, strlen($tablename) - 2);
        $this->db->insert($keyTable, array('column1' => 0));

        return $this->db->lastInsertId2();
    }

    /**
     * 删除主键表最大ID数据
     * @param $tablename
     */
    private function clearMaxIdData($tablename){
        $keyTable = 'Z_'.substr($tablename, 2, strlen($tablename) - 2);
        $deleteSql = "delete from {$keyTable} ;";
        $this->db->exec($deleteSql, true);
    }

    /**
     * 备份数据表
     * @param $tablename
     * @return mixed
     */
    private function backupTable($tablename)
    {
        $backflag = '_BAK' . date('YmdHis');
        $newtable = $tablename . $backflag;
        $sql = "select * into {$newtable} from {$tablename}";

        return $this->db->exec($sql, true);
    }

    /**
     * 根据单据类型生成单号
     * @param $sheetType
     * @return string
     */
    private function getSheetNo($sheetType){
        // 获取单据编码生成规则
        $sql = "SELECT FENTRYID,FSEQ,FLENGTH,FPROJECTTYPE,FFORMATETEXT,FPROJECTVALUE,FSEED,
                FINCREMENT,FCURRENTSERIALNUM,FSERIALTYPE,FSERIALVALUE
                FROM T_ESS_CODERULEENTRY
                WHERE FID = (SELECT FID FROM T_ESS_CODERULE WHERE FNUMBER = '{$sheetType}')
                 ORDER BY FSEQ ASC
        ";
        $this->db->query($sql, true);

        $sheetRules = $this->db->fetchAll();
        if(empty($sheetRules)){
            return '';
        }
        // 拼装单号
        $sheetNo = '';
        foreach ($sheetRules as $sheetRule) {
            $sheetNo .= $this->makeSheetString($sheetRule);
        }

        return $sheetNo;
    }

    /**
     * 根据单据规则生成单据字符串
     * @param $sheetRule
     */
    protected function makeSheetString($sheetRule){
        if(empty($sheetRule) || !is_array($sheetRule)){
            return '';
        }
        $sheetno = '';
        // 根据规则生成代码
        switch($sheetRule['FPROJECTTYPE']){
            case 1: // 固定前缀
                $sheetno = $sheetRule['FPROJECTVALUE'];
                break;
            case 2: // 日期格式
                $sheetno = date('Ymd');
                break;
            case 3: // 流水
                $lastFlowId = $sheetRule['FCURRENTSERIALNUM'] + $sheetRule['FINCREMENT'];
                if (($sheetRule['FSERIALTYPE']  == 2 && $sheetRule['FSERIALVALUE'] != date('Y-m-d')) || (strlen($lastFlowId) > $sheetRule['FLENGTH']) ){
                    $lastFlowId = $sheetRule['FSEED'] + $sheetRule['FINCREMENT'];
                }
                $sheetno = str_pad($lastFlowId, $sheetRule['FLENGTH'], '0', STR_PAD_LEFT);
                $this->updateSheetLastFlowID($sheetRule, $lastFlowId);
                break;
            default:
                break;
        }
        return $sheetno;
    }

    /**
     * 更新单据最后流水ID
     * @param $sheetRule
     * @param $lastFlowId
     */
    protected function updateSheetLastFlowID($sheetRule, $lastFlowId){
        // 更新单据流水最后ID
        $data['FCURRENTSERIALNUM'] = $lastFlowId;
        $data['FSERIALVALUE'] = $sheetRule['FPROJECTTYPE'] == 3 && $sheetRule['FSERIALTYPE'] == 2 ? date('Y-m-d') : '';
        return $this->db->update('T_ESS_CODERULEENTRY', $data, " and FENTRYID = {$sheetRule['FENTRYID']}");
    }
```

> 3. 四步逻辑

```
    // 订单处理
    public function consignmenOrder($orgcode, $data){
        if(empty($orgcode)){
            return null;
        }
        $sql1 = "select FCHANNELID from T_ESS_CHANNEL where FNUMBER = '{$orgcode}' and FENABLE = 1";
        $this->db->query($sql1);
        $custid = $this->db->fetchColumn();
        if(empty($custid)){
            return null;
        }

        try{
//            $fid = $this->consignmenSaveOrder($custid, $data);
//            $this->consignmenSubmitOrder($fid);
//            $this->consignmenVerifyOrder($fid);
//            $this->consignmenDeliveryOrder($custid, $fid, $data);
        }catch (Exception $e) {
            return [false, $e->getMessage()];
        }
        return [true, "ok"];
    }

    // 保存
    private function consignmenSaveOrder($custid, $data){
        $col = array_column($data['goods'], 'value');
        $sumNum = array_sum($col);
        $arr = [];
        foreach ($data['goods'] as $val) {
            array_push($arr, $val['salePrice'] * $val['discountRate'] * $val['value'] / 100);
        }
        $sumPrice = array_sum($arr);
        $this->db->query("SELECT FUSERID FROM T_ESS_USER_CHANNEL WHERE FCHANNELID={$custid}");
        $userId = $this->db->fetchColumn();
        $fid = $this->getMaxID('T_ESS_SALESDATA');
        try{
            $this->db->beginTransaction();
            $oData = array(
                'FID' => $fid,
                'FBILLNO' => $this->getSheetNo("CR-SYS-RETAILORDER"),
                'FDATE' => date('Y-m-d H:i:s'),
                'FSALEORG' => 1,                                // 销售商id 销售组织
                'FTYPEID' => 20,                                // FTYPEID  单据类型
                'FBUSINESSTYPE' => 58,                          // FBUSINESSTYPE 业务类型
                'FORDERCHANNEL' => $custid,
                'FSTAFF' => 0,
                'FUSERID' => $userId,
                'FCREATEDATE' => date('Y-m-d H:i:s'),
                'FSTATUS' => 'A',
                'FCURRENCYID' => 1,                         // 币别
                'FGOODSCOST' => $sumPrice,
                'FGOODSNUMS' => $sumNum,
                'FDELIVERYTYPE' => 4,                       // 交货方式
                'FEXPECTDATE' => date('Y-m-d H:i:s'),
                'FPROVIDE' => $custid,
                'FRECEIVEADDRID' => 0,
                'FNOTETYPE' => 2,
                'FRECEIVECHANNEL' => $custid,
                'FBALANCECHANNEL' => $custid,
                'FPAYCHANNEL' => $custid,
                'FPAYTYPE' => 2,
                'FSTAYSTOCKTIME' => date('Y-m-d H:i:s'),
                'FPROVIDETYPE' => 2,
                'FTERMINALTYPE' => 0,
                'FRECEIVEDMONEY' => $sumPrice,
                'FPAYMENT' => 1,
                'FPAYSTATUS' => 2,
                'FSIGNSTATUS' => 'E',
                'FENTERPISESTAFF' => 0,
                'FSHOPGUIDER' => 0,
                'FSECCUSTOMER' => $data['customerId'],
                'FADDRESS' => $data['customer'],
                'FPRINTCUSTOM' => 0
            );
            $res = $this->db->insert("T_ESS_SALESDATA", $oData, 0, 1);
            if (!$res) {throw new Exception("保存数据失败");}
            foreach($data['goods'] as $val) {
                $gData = [
                    'FENTRYID' => $this->getMaxID('T_ESS_SALESDATA_DETAIL'),
                    'FGOODSID' => $val['goodsId'],
                    'FSKUID' => 0,
                    'FMATERIALID' => $val['materialId'],
                    'FUNITID' => $val['unitId'],
                    'FBASEUNITID' => $val['realUnit'],
                    'FQTY' => $val['value'],
                    'FAPPROVENUMS' => $val['value'],
                    'FBASEQTY' => $val['value'],
                    'FAPPROVEBASENUMS' => $val['value'],
                    'FSTDPRICE' => $val['salePrice'],
                    'FPRICE' => $val['salePrice'] * $val['discountRate'] / 100,
                    'FAMOUNT' => $val['salePrice'] * $val['discountRate'] * $val['value'] / 100,
                    'FSALLERID' => 0,
                    'FROWTYPE' => 0,
                    'FDISCOUNTAMOUNT' => $val['salePrice'] * (100 - $val['discountRate']) * $val['value'] / 100,
                    'FDISCOUNT' => $val['salePrice'] * (100 - $val['discountRate']) * $val['value'] / 100,
                    'FISPRESENT' => 0,             // 不是赠品
                    'FIDENTITYID' => 0,
                    'FERPFLEXSITEMDETAILID' => 0,
                    'F_RDJT_DISCOUNTRATE' => $val['discountRate'],
                    'FORDERTYPE' => 3,              // 默认为其它
                    'FID' => $fid
                ];
                $res = $this->db->insert("T_ESS_SALESDATA_DETAIL", $gData, 0, 1);
                if (!$res) {throw new Exception("保存商品数据失败");}
            }
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollBack();
            $mess = $e->getMessage();
        }

        if (isset($mess)) {
            throw new Exception($mess);
        } else {
            $this->clearMaxIdData('T_ESS_SALESDATA');
            $this->clearMaxIdData('T_ESS_SALESDATA_DETAIL');
        }

        return $fid;
    }
  // 提交
    public function consignmenSubmitOrder($fid){
        $sql1 = "UPDATE T_ESS_SALESDATA SET FSTATUS = 'B', FDOCUMENTSTATUS = 'B' WHERE FID = {$fid}";
        $res1 = $this->db->exec($sql1, true);
        $sql2 = "UPDATE T_ESS_SALESDATA_DETAIL SET FORDERSTATUS = 'B' WHERE FID = {$fid}";
        $res2 = $this->db->exec($sql2, true);
        return $res1 && $res2;
    }

    // 审核 确认
    private function consignmenVerifyOrder($fid){
        $sql1 = "UPDATE T_ESS_SALESDATA SET FSTATUS = 'C', FDOCUMENTSTATUS = 'C' WHERE (FSTATUS = 'B' AND FID = {$fid}) ";
        $res1 = $this->db->exec($sql1, true);
        $sql2 = "UPDATE T_ESS_SALESDATA_DETAIL SET FORDERSTATUS = 'C' WHERE FID = {$fid}";
        $res2 = $this->db->exec($sql2, true);
        return $res1 && $res2;
    }

    // 发货
    private function consignmenDeliveryOrder($custid, $orderId, $data){
        $fid = $this->getMaxID('T_ESS_OUTSTOCK');
        // 判断库存
        foreach ($data['goods'] as $val) {
            if ($val['stockqty'] < $val['value']) {
                throw new Exception($val['goodsName'] . " 库存不足!");
            }
        }
        try {
            $this->db->beginTransaction();
            $sdata = [
                'FID' => $fid,
                'FBILLNO' => $this->getSheetNo('CR-SYS-OUTSTOCK'),
                'FSALEORGID' => 1,
                'FTYPE' => 11,
                'FBUSINESSTYPE' => 44,
                'FINCHANNELID' => $custid,
                'FOUTCHANNELID' => $custid,
                'FDATE' => date('Y-m-d H:i:s'),
                'FSTATUS' => 'C',
                'FMEMBERID' => 0,
                'FREVIEWEDUSER' => $data['customerId'],
                'FCHANNELSTAFFID' => 0,
                'FRECEIVER' => $data['cusobj']['secChannelName'],
                'FTEL' => $data['cusobj']['FTELPHONE'],
                'FRECEIVEADDR' => $data['customer'],
                'FRELPARTMENT' => '',
                'FACOUNTTAXNO' => '',
                'FCURRENCYID' => 1,
            ];
            // 出库单
            $res = $this->db->insert("T_ESS_OUTSTOCK", $sdata, 0, 1);
            if (!$res) {throw new Exception("生成出库单失败!");}
            // 获取订单详情
            $this->db->query("SELECT d.*,s.FBILLNO FROM T_ESS_SALESDATA_DETAIL d LEFT JOIN T_ESS_SALESDATA s ON d.FID = s.FID WHERE d.FID = {$orderId}");
            $goods = $this->db->fetchAll();
            // 出库详情单
            foreach ($goods as $val) {
                $gData = [
                    'FENTRYID' => $this->getMaxID('T_ESS_OUTSTOCKENTRY'),
                    'FSOURCEFORMID' => "SalesData",
                    'FORDERID' => $orderId,
                    'FORDERDETAILID' => $val['FENTRYID'],
                    'FGOODSID' => $val['FGOODSID'],
                    'FSKUID' => $val['FSKUID'],
                    'FMATERIALID' => $val['FMATERIALID'],
                    'FBASEUNITID' => $val['FBASEUNITID'],
                    'FUNITID' => $val['FUNITID'],
                    'FSTOCKID' => $custid,
                    'FBASEQTY' => $val['FQTY'],
                    'FQTY' => $val['FQTY'],
                    'FPRICE' => $val['FSTDPRICE'],
                    'FDISCOUNTPRICE' => $val['FPRICE'],
                    'FTAXAMOUNT' => $val['FAMOUNT'],
                    'FSOURCEBILLNO' => $val['FBILLNO'],
                    'FSOURCEBILLTYPEID' => 20,
                    'FSOURCEBILLID' => $orderId,
                    'FSOURCEBILLENTRYID' => $val['FENTRYID'],
                    'FISPRESENT' => 0,
                    'FSHOULDQTY' => $val['FQTY'],
                    'FID' => $fid,
                    'FMODIFYDATE' => date('Y-m-d H:i:s'),
                    'FCREATEDATE' => date('Y-m-d H:i:s'),
                    'FREVIEWEDTIME' => date('Y-m-d H:i:s'),
                    'FCREATORID' => $custid,
                    'FMODIFIERID' => $custid
                ];
                $res = $this->db->insert("T_ESS_OUTSTOCKENTRY", $gData, 0, 1);
                if (!$res) {throw new Exception("生成出库详情单失败!");}
                // 库存更新
                $date = date('Y-m-d H:i:s');
                $sql1 = "UPDATE T_ESS_INVENTORY SET FSTOCKQTY = FSTOCKQTY - {$val['FQTY']},FBASEUNITQTY = FBASEUNITQTY - {$val['FQTY']}, FMODIFYDATE = '{$date}' WHERE FGOODSID = {$val['FGOODSID']} AND FCHANNELID = {$custid}";
                $res1 = $this->db->exec($sql1, true);
                if (!$res1) {throw new Exception("更新库存失败!");}
            }

            // 更新状态
            $sql1 = "UPDATE T_ESS_SALESDATA SET FSTATUS = 'G', FDOCUMENTSTATUS = 'A' WHERE  FID = {$orderId} ";
            $res1 = $this->db->exec($sql1, true);
            $sql2 = "UPDATE T_ESS_SALESDATA_DETAIL SET FORDERSTATUS = 'G' WHERE FID = {$orderId}";
            $res2 = $this->db->exec($sql2, true);
            if (!$res1 || !$res2) {throw new Exception("更新状态失败!");}
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollBack();
            $mess = $e->getMessage();
        }

        if (isset($mess)) {
            throw new Exception($mess);
        } else {
            $this->clearMaxIdData('T_ESS_OUTSTOCK');
            $this->clearMaxIdData('T_ESS_OUTSTOCKENTRY');
        }
        return true;
    }
    
```
