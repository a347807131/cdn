#相关说明

## 1.JdbScanCodeService#bankCardPay(借贷宝银行卡支付)

- 改动部分
> 改为sigChargRedoService.sigChargeRedo 快捷支付交易实现外发银行扣款

- 上游系统

`web层`

## 2.AccountRefundService.processRefundZhilian(直连退款)

- 改动部分
> 改为unifiedRefundService.unifiedRedound 直连统一退款

- 上游系统

`AccountRefundApproveService`

`AsyncRefundService`

`OrderRetryJobService`

`RefundService`


## 3.RpmtRefundTaskService.tryCallZhilianRefund 

- 改动部分
> 改动与2相同，调直连统一退款

## 4.InnerPayService.acountCancel 撤销记账

- 改动部分
> 改动与2基本相同，不过增加了查询退款流水以确定退款类型

- 上游系统

`SecurityPayService.securityPayCancel`



