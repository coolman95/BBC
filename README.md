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

