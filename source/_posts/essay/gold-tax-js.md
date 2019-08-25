---
title: 航天信息金税盘接口 js 调用
author: Joden_He
tags: 
  - JavaScript
  - ActiveX 
  - 金税
categories: 
  - 随笔
description: 最近项目要求与单机版的金税盘接口进行对接，在这里简单记录一下自己的开发经验，希望可以帮助到有需要的人

---

### 背景

> 最近项目要求与单机版的金税盘接口进行对接，在这里简单记录一下自己的开发经验，希望可以帮助到有需要的人
>
> PS：接口使用 `js` 对接，仅支持 `ie` 浏览器。

### 前置条件

在进行开发时候需要有一些前置条件

1.  `ie` 浏览器

   - 开启 `activeX` 控件

     设置 --> Internet 选项 --> 安全 --> 自定义级别

     把 `activeX` 相关设置勾上

     可参考: 

   - 管理员身份运行调试

     如果开票软件安装到本地 `C盘` 需要使用管理员打开 `ie` 浏览器，然后再进行调试

2. 金税盘安装

   - 要确保金税盘安装准确安装了`TaxCardX.dll`

### 通用 `js`代码

```javascript
/** 航信金税盘相关function */
var goldTax = {
	/**
	 * 使用前先判断是否是ie浏览器
	 * @return true/false
	 */
	isIE: function () {
        if(!!window.ActiveXObject || "ActiveXObject" in window)
            return true;
        else
            return false;
    },
	/**
	 * 开启金税盘
	 * @param certPassword 单机版为证书口令  服务器版为地址 留空则读取开票BIN文件下cert.txt
	 * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx', // 错误信息
	 *        'data': {} // 接口返回相关数据
	 *     }
	 * </pre>
	 */
	openCard: function (certPassword) {
		var result = {
			'success': false,
			'code': -1,
			'msg': '',
			'data': {}
		};
        try {
            goldTax.card = new ActiveXObject("TaxCardX.GoldTax");
            // 单机版为证书口令  服务器版为地址 留空则读取开票BIN文件下cert.txt
            if(typeof certPassword != 'undefined') {
            	goldTax.card.CertPassword = certPassword;
            }
            // 开启金税盘
            goldTax.card.OpenCard();
            result.code = goldTax.card.RetCode;
            result.msg = goldTax.card.RetMsg;
            // 1011 开启成功
            if (result.code == 1011) {
            	result.success = true;
            	result.data = {
            		RetCode: goldTax.card.RetCode, /* RetCode - 状态码， 1011 开启成功 */
            		RetMsg: goldTax.card.RetMsg, /* RetMsg 状态信息 */
            		InvLimit: goldTax.card.InvLimit, /* InvLimit – 开票限额, 金税卡发票开具价税合计限额 */
            		TaxCode: goldTax.card.TaxCode, /* TaxCode 本单位税号 */
            		TaxClock: goldTax.card.TaxClock, /* TaxClock – 金税卡时钟 */
            		MachineNo: goldTax.card.MachineNo, /* MachineNo – 开票机号码，主开票机为 0 */
            		IsInvEmpty: goldTax.card.IsInvEmpty, /* IsInvEmpty – 有票标志，0为金税卡中无可开发票，1为有票 */
            		IsRepReached: goldTax.card.IsRepReached, /* IsRepReached – 抄税标志，0为未到抄税期，1为已到抄税期 */
            		IsLockReached: goldTax.card.IsLockReached, /* IsLockReached – 锁死标志，0为未到锁死期，1为已到锁死期 */
            	};
            } else {
            	result.success = false;
            }
        } catch (e) {
            result.success = false;
            result.code = -1;
            result.msg = 'ActiveX Error, ' + e.description;
        }
        return result;
    },
    /**
     * 关闭金税盘
     * @return
     * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx' // 错误信息
	 *     }
	 * </pre>
     */
    closeCard: function() {
    	var result = {
			'success': false,
			'code': -1,
			'msg': ''
		};
    	if (typeof goldTax.card != 'undefined') {
    		try {
    			goldTax.card.closeCard();
    			result.code = goldTax.card.RetCode;
	            result.msg = goldTax.card.RetMsg;
    			// RetCode 9000 调用成功，其他失败
    			if (result.code == 9000) {
    				result.success = true;
    			} else {
    				result.success = false;
    			}
    		} catch (e) {
    			result.success = false;
	            result.code = -1;
	            result.msg = 'ActiveX Error, ' + e.description;
    		}
    		return result;
    	}
    },
    /**
     * 查询库存发票
     * @param invKind 发票种类
 	 * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx', // 错误信息
	 *        'data': {} // 接口返回相关数据
	 *     }
	 * </pre>
     */
    invoiceInventory: function(invKind) {
    	var result = {
			'success': false,
			'code': -1,
			'msg': '',
			'data': {}
		};
    	try {
    		if (typeof goldTax.card != 'undefined') {
    			goldTax.card.InfoKind = invKind;
    			goldTax.card.GetInfo();
    			result.code = goldTax.card.RetCode;
	            result.msg = goldTax.card.RetMsg;
    			// RetCode 3011 读取成功，其他失败
    			if (result.code == 3011) {
    				result.success = true;
    				result.data = {
    					RetCode: goldTax.card.RetCode, /* RetCode - 状态码， 3011 读取成功，其他失败 */
            			RetMsg: goldTax.card.RetMsg, /* RetMsg 状态信息 */
            			InfoTypeCode: goldTax.card.InfoTypeCode, /* 要开具发票的十位代码。为空或全为0时，表示无可用发票 */
            			InfoNumber: goldTax.card.InfoNumber, /* 要开具发票的号码 */
            			InvStock: goldTax.card.InvStock, /* 剩余的可用发票份数 */
            			TaxClock: goldTax.card.TaxClock /* 金税盘时钟 */
    				};
    			} else {
    				result.success = false;
    			}
    		} else {
    			result.success = false;
	            result.code = -1;
	            result.msg = 'Please Open Card First';
    		}
		} catch (e) {
			result.success = false;
            result.code = -1;
            result.msg = 'ActiveX Error, ' + e.description;
		}
		return result;
    },
    /**
     * 发票校验
     * @param o 传入发票信息
     * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx', // 错误信息
	 *        'data': {} // 接口返回相关数据
	 *     }
	 * </pre>
  	 */
    invoiceVeriferify: function(o) {
    	return goldTax.inner.invoice(1, o);
    },
    /**
     * 发票开具
     * @param o 传入发票信息
     * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx', // 错误信息
	 *        'data': {} // 接口返回相关数据
	 *     }
	 * </pre>
  	 */
    invoicing: function(o) {
    	return goldTax.inner.invoice(0, o);
    },
    /**
     * 发票打印
     * @param o 传入发票信息
     * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx' // 错误信息
	 *     }
	 * </pre>
  	 */
    printInv: function(o) {
    	var result = {
			'success': false,
			'code': -1,
			'msg': ''
		};
    	try {
    		if (typeof goldTax.card != 'undefined') {
    			goldTax.card.InfoKind = o.InfoKind; /* 发票种类（0：专用发票  2：普通发票  11：货物运输业增值税专用发票  12：机动车销售统一发票） */
    			goldTax.card.InfoTypeCode = o.InfoTypeCode; /* 要打印发票的十位代码 */
                goldTax.card.InfoNumber = o.InfoNumber; /* 要打印发票的号码 */
                goldTax.card.GoodsListFlag = o.GoodsListFlag; /* 销货清单标志，0 – 打印发票，1 – 打印销货清单 */
                goldTax.card.InfoShowPrtDlg = o.InfoShowPrtDlg; /* 打印时是否显示边距确认对话框，0 – 不出现，1 – 出现 */
                // 调用接口
    			goldTax.card.PrintInv();
    			result.code = goldTax.card.RetCode;
	            result.msg = goldTax.card.RetMsg;
    			// RetCode 5011 打印成功，5001 – 未找到发票或清单，5012 – 未打印，5013 – 打印失败
    			if (result.code == 5011) {
    				result.success = true;
    			} else {
    				result.success = false;
    			}
    		} else {
    			result.success = false;
	            result.code = -1;
	            result.msg = 'Please Open Card First';
    		}
		}  catch (e) {
            result.success = false;
            result.code = -1;
            result.msg = 'ActiveX Error, ' + e.description;
        }
		return result;
    },
    /**
     * 已开发票作废
     * @param o 传入发票信息
     * @return 
	 * <pre>
	 *     {
	 *        'success': true/false, // 成功或者失败
	 *        'code': -1/其他, // 错误码，-1 为系统错误
	 *        'msg': 'xxx' // 错误信息
	 *     }
	 * </pre>
  	 */
    cancelInv: function(o) {
    	var result = {
			'success': false,
			'code': -1,
			'msg': ''
		};
    	try {
    		if (typeof goldTax.card != 'undefined') {
    			goldTax.card.InfoKind = o.InfoKind; /* 发票种类（0：专用发票  2：普通发票  11：货物运输业增值税专用发票  12：机动车销售统一发票） */
    			goldTax.card.InfoTypeCode = o.InfoTypeCode; /* 要作废发票的十位或十二位代码 */
                goldTax.card.InfoNumber = o.InfoNumber; /* 要作废发票的号码 */
                // 调用接口
    			goldTax.card.CancelInv();
    			result.code = goldTax.card.RetCode;
	            result.msg = goldTax.card.RetMsg;
    			// RetCode 6011 打印成功，6001 – 当月发票库未找到该发票，6002 – 该发票已经作废，6012 – 作废失败，6013 – 作废失败(异常)
    			if (result.code == 6011) {
    				result.success = true;
    			} else {
    				result.success = false;
    			}
    		} else {
    			result.success = false;
	            result.code = -1;
	            result.msg = 'Please Open Card First';
    		}
		}  catch (e) {
            result.success = false;
            result.code = -1;
            result.msg = 'ActiveX Error, ' + e.description;
        }
		return result;
    },
    // 内部方法
    inner: {
    	/**
	     * invoice 接口
	     * @param checkEWM 默认为0（0: 发票开具, 1: 发票校验, 2: 空白作废)
	     * @param o 传入发票信息
	     * @return 
		 * <pre>
		 *     {
		 *        'success': true/false, // 成功或者失败
		 *        'code': -1/其他, // 错误码，-1 为系统错误
		 *        'msg': 'xxx', // 错误信息
		 *        'data': {} // 接口返回相关数据
		 *     }
		 * </pre>
	  	 */
    	invoice: function(checkEWM, o) {
    		var result = {
				'success': false,
				'code': -1,
				'msg': '',
				'data': {}
			};
	    	try {
	    		if (typeof goldTax.card != 'undefined') {
	    			// CheckEWM值默认为0（为1时用于发票校验。
	    			// 注意：一旦CheckEWM值置1用于发票校验之后，
	    			//如果要再进行发票开具必须手动将CheckEWM值置为0，否则Invoice()方法的功能将一直处于发票校验状态）
	    			goldTax.card.CheckEWM = checkEWM; 
	    			goldTax.card.InvInfoInit(); /* 初始化发票抬头信息 */
	    			goldTax.card.InfoKind = o.InfoKind; /* 增值税普通发票2 专票0 */
	    			goldTax.card.InfoClientName = o.InfoClientName; /* 购方名称 */
	                goldTax.card.InfoClientTaxCode = o.InfoClientTaxCode; /* 购方税号 */
	                goldTax.card.InfoClientBankAccount = o.InfoClientBankAccount; /* 购方开户行及账号 */
	                goldTax.card.InfoClientAddressPhone = o.InfoClientAddressPhone; /* 购方地址电话 */
	                goldTax.card.InfoSellerBankAccount = o.InfoSellerBankAccount; /* 销方开户行及账号 */
	                goldTax.card.InfoSellerAddressPhone = o.InfoSellerAddressPhone; /* 销方地址电话 */
	                // 如果是多商品多税率 税率应该放到下面商品信息循环里
	                if (typeof o.InfoTaxRate != 'undefined' && o.InfoTaxRate) { 
	                	goldTax.card.InfoTaxRate = o.InfoTaxRate; /* 税率,(已授权的税率， 17% 传17) */
	                }
	                goldTax.card.InfoNotes = o.InfoNotes; /* 备注 */
	                goldTax.card.InfoInvoicer = o.InfoInvoicer; /* 开票人 */
	                // 复核人，可为空
	                if (typeof o.InfoChecker != 'undefined' && o.InfoChecker) {
	                	goldTax.card.InfoChecker = o.InfoChecker;
	                }
	                // 收款人，可为空
	                if (typeof o.InfoCashier != 'undefined' && o.InfoCashier) {
	                	goldTax.card.InfoCashier = o.InfoCashier;
	                }
	                // 如不为空，则开具销货清单，此为发票上商品名称栏的清单信息，应为“(详见销货清单)”字样
	                if (typeof o.InfoListName != 'undefined' && o.InfoListName) { 
	                	goldTax.card.InfoListName = o.InfoListName;
	                }
	                // 清空商品明细列表
	                goldTax.card.ClearInvList(); 
	                // 遍历行
	                $.each(o.InvList, function(i, v) {
	                	goldTax.card.InvListInit(); /* 初始化发票明细信息各项属性 */
	                	goldTax.card.ListGoodsName = v.ListGoodsName; /* 商品或劳务名称 */
		                goldTax.card.ListTaxItem = v.ListTaxItem; /* 税目，4位数字，商品所属类别 */
		                goldTax.card.ListStandard = v.ListStandard; /* 规格型号 */
		                // 计量单位，如计量单位为空，则忽略数量和单价
		                if (typeof v.ListUnit != 'undefined' && v.ListUnit) {
		                	goldTax.card.ListUnit = v.ListUnit;
		                }
		                // 建议传入数量和含税单价或含税金额 由接口计算带小数的税额 规避误差
		                goldTax.card.ListNumber = v.ListNumber; // 数量
		                goldTax.card.ListPrice = v.ListPrice; // 单价
		                // 金额，可以不传(为0)，由接口软件计算，如传入则应符合计算关系
		                if (typeof v.ListAmount != 'undefined' && v.ListAmount ) {
		                	goldTax.card.ListAmount = v.ListAmount;
		                }
		                goldTax.card.ListPriceKind = v.ListPriceKind; /* 含税价标志，单价和金额的种类， 0为不含税价，1为含税价 */
		                // 税额可以不传(为0)，由接口软件计算，如传入则应符合计算关系
		                if (typeof v.ListTaxAmount != 'undefined' && v.ListTaxAmount) {
		                	goldTax.card.ListTaxAmount = v.ListTaxAmount;
		                }
		                // 添加一行
		                goldTax.card.AddInvList();
	                });
	                // 调用接口
	    			goldTax.card.Invoice();
	    			result.code = goldTax.card.RetCode;
		            result.msg = goldTax.card.RetMsg;
	    			// RetCode 4011 开票成功，其他失败
	    			if (result.code == 4011) {
	    				result.success = true;
	    				result.data = {
	    					RetCode: goldTax.card.RetCode, /* RetCode - 状态码， 3011 读取成功，其他失败 */
	            			RetMsg: goldTax.card.RetMsg, /* RetMsg 状态信息 */
	            			InfoAmount: goldTax.card.InfoAmount , /* 合计不含税金额 */
	            			InfoTaxAmount: goldTax.card.InfoTaxAmount, /* 合计税额 */
	            			InfoInvDate: goldTax.card.InfoInvDate, /* 开票日期 */
	            			InfMonth: goldTax.card.InfMonth, /* 所属月份 */
	            			InfoTypeCode: goldTax.card.InfoTypeCode, /* 发票十位代码 */
	            			InfoNumber: goldTax.card.InfoNumber, /* 发票号码 */
	            			GoodsListFlag: goldTax.card.GoodsListFlag /* 销货清单标志，0 – 无销货清单，1 – 有销货清单 */
	    				};
	    			} else {
	    				result.success = false;
	    			}
	    		} else {
	    			result.success = false;
		            result.code = -1;
		            result.msg = 'Please Open Card First';
	    		}
			} catch (e) {
	            result.success = false;
	            result.code = -1;
	            result.msg = 'ActiveX Error, ' + e.description;
	        }
			return result;
    	}
    }
};
```

### 相关资料

`CSDN` 需要积分下载 [金税 防伪税控 组件接口 开发文档 代码案例](https://download.csdn.net/download/qq_36894469/9891113)

没有积分的 [戳这里](https://pan.baidu.com/s/11djfBHQMfhxpsLi3BqOOSA)
