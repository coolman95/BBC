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
<pageview type="billView" name="新增客户" plugin="kingdee.k3.rd.webapp.web.MyCustomerEditPluginRd" bo="Customer" ao="ChannelStaff" editview="myCustomerEditView" webeditview="myCustomerEdit" mainchannel="FCHANNELID">
	<view>
		<toolbar id="toolbar">
			<menubutton id="refresh" name="刷新" icon="refresh"  iservice="refresh"/>
			<menubutton id="add" name="新增"  icon="add" iservice="add"/>
			<menubutton id="save" name="保存" icon="save"  iservice="save"/>			
		</toolbar>
		<tabgroup>
			<tab name="基本信息">		
        
      </tab>
		</tabgroup>
	</view>
</pageview>

```

> 添加控制器 java 文件

```
package kingdee.k3.rd.webapp.web;

import kingdee.k3.o2o.bos.dynamic.form.plugin.BillViewPlugin;

import java.util.Date;
import java.util.List;
import java.util.*;

import kingdee.k3.o2o.bos.base.dynamic.orm.DynamicObject;
import kingdee.k3.o2o.bos.web.context.WebContext;
import kingdee.k3.o2o.bos.dynamic.form.BillView;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.ClickEvent;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.DataChangeEvent;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.LoadDataEvent;
import kingdee.k3.o2o.bos.dynamic.form.plugin.event.ToolbarClientEvent;


public class MyCustomerEditPluginRd extends BillViewPlugin{

}

```

> 逻辑代码编译调试
