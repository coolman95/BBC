# BBC 二次开发实记

## 为客户关联价目表

> 客户列表SQL

```
  select s.FID id,sl.FNAME secChannelName,s.FNUMBER,s.FTELPHONE,s.FCONTACT, s.FDEFRECADDRESS defAddress  
      from T_RD_SecSupplier s 
      inner join T_RD_SecSupplier_L sl on s.FID  = sl.FID and sl.FLOCALEID= :localeid
      inner join T_BD_CUSTOMER c on c.FCUSTID=s.FBELONGCUST
      inner join T_ESS_CHANNEL ch on c.FMASTERID = ch.FCUSTOMERID 
      where s.FDOCUMENTSTATUS='C' and s.FFORBIDSTATUS='A'
```

主要关联 `T_ESS_CHANNEL` FCUSTOMERID 


> 添加二次开发之 XML 文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE pageview SYSTEM "http://xd.k3cloud.kingdee.com/dtd/JFS.dtd" >
<pageview type="billView" name="新增客户" plugin="kingdee.k3.rd.webapp.web.MyCustomerEditPluginRd" bo="secSupplier" ao="ChannelStaff" editview="myCustomerEditView" webeditview="myCustomerEdit" mainchannel="FCHANNELID">
	<view>
		<toolbar id="toolbar">
			<menubutton id="refresh" name="刷新" icon="refresh"  iservice="refresh"/>
			<menubutton id="add" name="新增"  icon="add" iservice="add"/>
			<menubutton id="save" name="保存" icon="save"  iservice="save"/>			
		</toolbar>
		<tabgroup>
			<tab name="基本信息">
				<latticerowpanel labelwidth="140px">
					<mltext id="FNAME" name="客户名称" orm="name" required="true"/>
					<code name="编码" rule="CR-SYS-CUSTOMER" id="FNUMBER" orm="customerno"/>
					<text id="FCONTACT" name="联系人" orm="contact" required="true"/>
					<text id="FTELPHONE" name="联系方式" orm="telphone" required="true"/>
					<text id="FADDRESS" name="客户地址" orm="address" required="true"/>
					<selectdata id="FMYPRICEID" name="适用价目表" superselect="mypricelist" linkdatamap="FMYPRICEID:id, FGPRICETITLE" orm="mypriceid" required="true"/>
					<datetime id="FCREATEDATE" name="创建时间" orm="createdate" disabled="true"/>
					<mltext id="FREMARK" name="备注" orm="remark"/>
					<text id="priceId" hidden="true"/>
					<text id="priceName" hidden="true"/>
				</latticerowpanel>
			</tab>
		</tabgroup>
		
		<html id="html"><![CDATA[
					<script>
					// 不想用定时器做的，试了很多方法 superslelect 没有自动填充值，只能先这样凑合，后面有时间再研究 superslelect 原理
					$(window).bind("load", function () {
						setTimeout(function(){ 
							setPrice();
						}, 300);
						
					});
					// 点击save
					$('a[id^="save_"]').bind("click",function(){
						setTimeout(function(){
							setPrice();
						}, 100);
					});
					// 点击刷新
					$('a[id^="refresh_"]').bind("click",function(){
						setTimeout(function(){
							setPrice();
						}, 100);
					});
					// 填充值实现方法
					function setPrice(){
						let id = $('input[id^="priceId_"]').val();
						let title = $('input[id^="priceName_"]').val();
						
						console.log(id,title)
						let obj = $('[id^="FMYPRICEID_"]');
						for(let i=0; i<obj.length; i++) {
							let arr = obj[i].id.split("_");
							if (arr.length===3 && arr[2]=="v") {
								$('[id^="FMYPRICEID_"]').eq(i).val(title)
							}
							if (arr.length===2) {
								$('[id^="FMYPRICEID_"]').eq(i).attr("skey", id);
								$('[id^="FMYPRICEID_"]').eq(i).attr("svalue", title);
							}
						}
					}
					</script>
		]]></html>
		
	</view>
</pageview>

```

> 添加控制器 java 文件

```
package kingdee.k3.rd.webapp.web;

import kingdee.k3.o2o.bos.dynamic.form.plugin.BillViewPlugin;

import java.util.Date;

import kingdee.k3.o2o.bos.base.dynamic.orm.DynamicObject;
import kingdee.k3.o2o.bos.base.object.DaoParam;
import kingdee.k3.o2o.bos.base.util.SpringContextUtil;
import kingdee.k3.o2o.bos.web.context.WebContext;
import kingdee.k3.o2o.bos.dynamic.form.BillView;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.LoadDataEvent;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.ToolbarClientEvent;
import kingdee.k3.rd.RdServiceFactory;

public class MyCustomerEditPluginRd extends BillViewPlugin{
	RdServiceFactory rdServiceFactory = (RdServiceFactory) SpringContextUtil.getBean("rdServiceFactory");
	
	protected DynamicObject beforeBindData(WebContext webCtx, BillView view, LoadDataEvent event, DynamicObject dataObject)
	{
		view.updateValue("FCREATEDATE", new Date().getTime());
		
		String pkValue = webCtx.getForm().getString("pkValue");
		if (pkValue != null && !"".equals(pkValue)) {
			DaoParam param = new DaoParam();
			String from = "T_RD_SecSupplier s left join T_RD_SecSupplier_L sl ON s.FID=sl.FID AND sl.FLocaleID=2052 WHERE s.FID = "+pkValue;
			DynamicObject supplier = rdServiceFactory.getRdChannelService().getRow(webCtx, "s.*,sl.*", from, param);
			Long priceId = supplier.getLong("FMYPRICEID");
			if (priceId != 0) {
				DynamicObject priceName = rdServiceFactory.getRdChannelService().getRow(webCtx, "FGPRICETITLE", "T_ESS_PRICE_L WHERE FGPRICEID="+priceId, param);
				view.updateValue("priceId", priceId);
				view.updateValue("priceName", priceName.getString("FGPRICETITLE"));
			}
			System.out.println(supplier.toString());
			return supplier;
		}
		
		return super.beforeBindData(webCtx, view, event, dataObject);
	  
	}
	
	
	protected boolean beforeMetadataSave(WebContext webCtx, BillView view, ToolbarClientEvent event, DynamicObject beforeData, DynamicObject afterData) {
		long channelId = webCtx.getLoginState().getChannelId();
		String pkValue = webCtx.getForm().getString("pkValue");
		Long userId = webCtx.getLoginState().getUserId();
		
		if (pkValue != null && !"".equals(pkValue)) {
			afterData.put("modifydate", new Date().getTime());
		} else {
			afterData.put("modifydate", afterData.getString("createdate"));
			afterData.put("masterid", "0");
			afterData.put("documentstatus", "C");
			afterData.put("forbidstatus", "A");
			afterData.put("useorgid", "1");
			afterData.put("createorgid", "1");
			afterData.put("creatorid", userId);
			DaoParam param = new DaoParam();
			DynamicObject channel = rdServiceFactory.getRdChannelService().getRow(webCtx, "FCUSTOMERID,FSALEORGID", "T_ESS_CHANNEL WHERE FCHANNELID = "+channelId, param);
			afterData.put("belongcust", channel.getLong("FCUSTOMERID"));
			afterData.put("defsaleorgid", channel.getLong("FSALEORGID"));
		}
		afterData.put("modifierid", userId);
		afterData.put("defrecaddress", afterData.getString("address"));
		return true;
	}

}


```

> 逻辑代码编译调试
