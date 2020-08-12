## 一、接口规范

在往来接口中，使用UTF-8编码、json格式进行数据传递，接口均为POST请求,

可在请求时加上 content-type为application/json;charset=utf-8头部信息。
```
//入参示例
{
    "partner_token": "用户token",
    "encode_data": "aes加密数据"
}
//返回示例
{
	"code":"状态码，0表示成功",
	"msg":"状态码的说明，成功为“success”，失败返回具体的错误信息",
	"data":{"返回的详细数据内容，当无需返回的时候可为空"}
}
```

## 二、数据加密

接口使用aes-128-cbc对称加密保护通讯数据，先json格式化入参，再做加密处理
```
//示例，php
const AES_KEY = 'Loan@apis#123456@';
const AES_IV = 'loan#654321@apis';
const AES_METHOD = 'AES-128-CBC';

public function encrypt($json_data)
{
    //加密 
    return openssl_encrypt($json_data, static::AES_METHOD, static::AES_KEY, 0, static::AES_IV);
}

public function decrypt($json_data)
{
    //解密  
    return openssl_decrypt($json_data, static::AES_METHOD, static::AES_KEY, 0, static::AES_IV);
}
```

## 三、接口列表

### 01. 申请合作者验签
接口地址：开发人员对接分配
```
//贷超提供
{
    partner_token:"合作者token",
    aes_key:"加密key",
    aes_iv:"加密iv"
}
//合作者提供
{
    product_id:"产品id，双方约定",
    product_name:"产品应用名",
    product_icon:"产品icon文件或者url",
    product_desc:"产品描述",
    amount_min:"可借额度下线",
    amount_max:"可借额度上线",
    interest_rate:"利率(日或月利率)",
    gp_link:"谷歌应用商店地址，有请填写，便于贷超运营丰富产品描述，提高转化"
}
```

### 02.  预检接口
接口地址：机构提供/get-loan-precheck
```
//入参示例
{
	"product_id": "产品id", 
	"user_mobile": "用户手机号码  (8开头的9-13位数字)", 
	"user_idcard": "用户身份证", 
	"user_name": "用户身份证姓名(大小写字母+空格)",  #非必传 
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
        "is_reloan":0,
        "loan_range":[
            {
                "amount":"可借金额",
                "term":["可借周期"]
            },
            {
                "amount":"可借金额",
                "term":["可借周期"]
            }
        ],
        "term_unit":"借款单位(1-天,  2-月)"
    }
}

错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 03.  获取用户可借款金额和期限
接口地址：机构提供/get-loan-amount
```
//入参示例
{
	"product_id": "产品id", 
	"user_mobile": "用户手机号码", 
	"user_idcard": "用户身份证",  #非必传 
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
	    "loan_range":[
            {
                "amount":"可借金额",
                "term":["可借周期"]
            },
            {
                "amount":"可借金额",
                "term":["可借周期"]
            }
        ],
	    "term_unit":"借款单位(1-天,  2-月)"
	    "default_amount":"默认可借金额",
	    "default_term":"默认可借周期"
    }
}

错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 04.  利息试算
接口地址：机构提供/get-loan-interest
```
//入参示例
{
	"product_id": "产品id", 
	"user_mobile": "用户手机号码", 
	"user_idcard": "用户身份证",  #非必传 
	"application_amount": "借款金额（最小值100000）", 
	"application_term": "借款期限（最小值7）", 
	"term_unit": "借款单位(1-天,  2-月)", 
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
	    "interest_amount":"利息",
	    "admin_amount":"管理费",
	    "actual_amount":"实际到账金额",
	    "repay_amount":"还款金额"，
	    "expiry_time":"产品试算时的贷款应还日期(以当前时间点算的，有误差。真正以放款时确认的为准)"
    }
}

错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 05.  基础信息推送接口(一推)
接口地址：机构提供/push-basic-info
```
//入参示例
{
	"product_id": "产品id", 
	"device_ip": "用户来源ip", 
	"basic_info": "用户申请信息", 
	"order_info": "订单信息", 
	"is_reloan": "是否复贷用户(0-否, 1-是)", 
}

//请求示例
{
    "basic_info":{
        "user_type":"用户类型 1-已工作 2-学生 3-其他",
        "idcard_image_front":"身份证正面照url",
        "idcard_image_hand":"手持身份证url",
        "education":"教育程度 1-未受过教育 2-小学 3-中学 4-高中 5-大学本科 6-研究生",
        "loan_remark":"贷款用途 1-生活、工作使用 2-消费使用 3-教育支出 4-旅游使用 5-结婚使用 6-医疗支出 7-装修 8-宗教旅行 9-生育支出 10-多用途使用 11-其他",
        "month_income":月收入(5~9位首位非0数字),
        "religion":"宗教 1-穆斯林 2-基督教 3-天主教徒 4-印度教 5-佛教 6-儒教/孔教"
    },
    "device_ip":"用户来源ip",
    "is_reloan":是否复贷用户(0-否, 1-是),
    "order_info":{
        "order_no":"订单号",
        "application_amount":借款金额,
        "application_term":借款期限,
        "term_unit":借款单位1-天, 2-月,
        "product_id":产品id,
        "user_name":"姓名",
        "user_mobile":"手机号(8开头的9-13位数字)",
        "user_idcard":"身份证号"
    },
    "product_id":产品id,
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{}
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 06.  补充信息推送(二推)
接口地址：机构提供/push-supplement-info
```
//入参示例
{
	"product_id": "产品id", 
	"device_ip": "用户来源ip", 
	"supplement_info": "用户信息", 
	"order_info": "订单信息", 
}

//请求示例
{
    "supplement_info":{
        "is_charge":2,
        "birthday":"生日",
        "marital_status":"婚姻状态 1-单身 2-已婚 3-离婚 4-丧偶",
        "sex":"性别 1-男 2-女",
        "face_photo_image":[
            "活体照片url数组, 可能有多张"
        ],
        "pass_face_photo":活体是否通过， 1-是,
        "same_person":人脸比对结果,是否同一人， 1-是,
        "confidence_score":"人脸相似度",
        "work_type":"工作类型 1-企业家 2-员工（私人） 3-员工（公共） 4-专业人士 5-公民 6-学生 7-无工作 8-自雇 9-工薪阶层 10-其他",
        "company":"公司名称",
        "area":{
            "province":"省",
            "city":"市",
            "district":"区",
            "village":"村镇",
            "detail_address":"详细地址"
        },
        "emergs":[
            {
                "emergs_name":"姓名",
                "emergs_phone":"电话",
                "emergs_relation":"关系"
            },
            {
                "emergs_name":"姓名",
                "emergs_phone":"电话",
                "emergs_relation":"1-夫妻/配偶 2-父母 3-子女/孩子 4-兄弟/姐妹 5-亲戚 6-朋友 7-同事 8-其他"
            }
        ], //紧急联系人数据, 至少2个
        "device_info":{
            //设备信息
        }
    },
    "order_info":{
        "order_no":"订单号",
        "application_amount":借款金额,
        "application_term":借款期限,
        "term_unit":借款单位1-天, 2-月,
        "user_mobile":"手机号",
    },
    "product_id":产品id,
    "device_ip":"用户来源ip"
}
​

//返回示例

​正常返回：{
    "code":0,
	"message":"success",
	"data":{}
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 07.  获取银行卡列表/还款方式列表
接口地址：机构提供/get-bank-list
```
//入参示例
{
	"action_type": "1-绑卡银行列表, 2-还款方式", 
	"product_id": "产品id", 
}

//返回示例

action_type=1,绑卡支持的银行卡列表:
{
	"code":0,
	"message":"success",
	"data":{
        "banklist":[
            {
                "bank_code":"BUKOPIN",
                "bank_name":"BankBukopin"
            },
            {
                "bank_code":"CIMB",
                "bank_name":"BankCIMBNiaga"
            }
        ]
    }
}

action_type=2,还款支持的还款方式:
{
	"code": 0,
	"message": "success",
	"data": {
		"banklist":[
            {
                "bank_code":"BNI",
                "method_code":"1001",
                "repay_method":1
            },
            {
                "bank_code":"MANDIRI",
                "method_code":"1003",
                "repay_method":1
            },
            {
                "bank_code":"Alfamart",
                "method_code":"2001",
                "repay_method":2
            }
        ]
	}
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 08.  绑定银行卡
接口地址：机构提供/push-bind-bank
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
	"bank_card": "银行卡号 (9-16位数字)", 
	"user_name": "开户人姓名(与下单姓名一致   大小写字母+空格)", 
	"bank_code": "开户银行code",  
}

//返回示例

​正常返回：{
	"code":0,
	"message":"success",
	"data":{}
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,
	telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 09.  获取绑定的银行卡
接口地址：机构提供/get-bind-bank
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：{
	"code":0,
	"msg":"success",
	"data":{
	    "order_no":"111",
	    "card_list":[
	        {
	            "bank_code":"ICBC",
	            "bank_card":"6212262308004359191",
	            "extra_info":""
	        }
	    ]
    }
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 10.  获取合同url
接口地址：机构提供/get-loan-contract
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​{
	"code":0,
	"msg":"success",
	"data":{
		"contract_url":"https://www.xxx.com/api/contact/xxxx?aa=bb&cc=dd",
		"contract_desc":"xxx pinjiaman contract",
    }
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 11.  确认借款
接口地址：机构提供/push-order-confirm
```
//入参示例
{
	"order_no": "订单号", 
	"product_id": "产品id", 
	"application_amount": "借款金额", 
	"application_term": "借款期限", 
	"term_unit": "借款单位1-天, 2-月", 
}

//返回示例

​正常返回：{
	"code":0,
	"msg":"success",
	"data":{}
}

​错误返回：{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}


```
### 12.  获取订单状态
接口地址：机构提供/get-order-status
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
		"order_no":"订单号",
		"order_status":"订单状态，参考枚举值",
		"update_time":最后更新时间,
	}
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 13.  获取审批结论
接口地址：机构提供/get-approval-result
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：
审批中：
{
	"code":0,
	"message":"success",
	"data":{
        "order_no":"订单编号",
        "order_status":订单状态,
        "order_remark":"等待审批"
    }
}

​审批拒绝：
{
    "code":0,
    "message":"success",
    "data":{
        "order_no":"订单编号",
        "order_status":订单状态,
        "order_remark":"拒绝原因"
    }
}

审批通过:
{
    "code":0,
    "message":"success",
    "data":{
        "order_no":"订单编号",
        "order_status":订单状态,
        "approval_time":"贷款时间",
        "loan_time":"审批时间",
        "approval_amount":"审批金额",
        "approval_term":"审批周期",
        "term_unit":"周期单位:1-天 2-月",
        "interest_amount":"总利息",
        "interest_rate":"每日或每月利率",
        "actual_amount":"实际到账金额",
        "admin_amount":"总管理费",
        "repay_amount":"总还款额",
        "order_remark":"本金1600000.00，利息0，还款金额1600000"
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 14.  拉取还款计划
接口地址：机构提供/get-repay-plan
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
        "order_no":"订单号",
        "order_status":"订单状态",
        "loan_time":"贷款时间",
        "loan_amount":"贷款金额",
        "loan_term":"贷款周期",
        "term_unit":"周期单位:1-天 2-月",
        "interest_amount":"应还利息",
        "interest_rate":"每天或每月利率",
        "expiry_time":"到期时间",
        "expiry_amount":"逾期利息",
        "is_overdue":"是否逾期 0-未逾期 1-已逾期",
        "overdue_days":"逾期天数",
        "admin_amount":"管理费，部分机构收取",
        "repay_amount":"还款总额",
        "url_repay":"是否支持URL还款，星合",
        "url_info":"url还款链接",
        "delay_repay":"是否支持展期 ,0不支持, 1支持",
        "delay_status":"展期支付状态 0 展期中, 1 展期支付成功, 2 展期支付失败, null 未发起过展期",
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}


```
### 15.  获取还款详情/发起还款/发起展期
接口地址：机构提供/get-repay-detail
```
//入参示例
{
	"order_no": "订单编号", 
	"product_id": "产品id", 
	"repay_method": "还款类型 1银行，2便利店", 
	"method_code": "支付方式code", 
	"repay_type": "还款类型 1-立即还款 2-展期还款", 
	"delay_term": "延期周期，默认传7就可以", 
}

//返回示例

​正常返回：
{
    "code":0,
    "message":"success",
    "data":{
        "order_no":"订单编号",
        "repay_amount":"当前所需还款额",
        "repay_code":"还款码",
        "expire_time":"还款码失效时间",
        "bank_code":"支付对象code",
        "method_code":"支付方式code",
        "merchant_name":"便利店还款商户名",
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 16.  获取展期记录
接口地址：机构提供/get-delay-list
```
//入参示例
{
    "order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：
{
	"code":0,
	"msg":"success",
	"data":{
        "order_no":"订单编号",
        "delay_times":"当前延期成功的数目",
        "delay_list":[
            {
                "loan_amount":"贷款金额",
                "delay_amount":"展期当前所需还款额",
                "delay_term":"展期周期",
                "term_unit":"展期期限单位:1-天 2-月",
                "delay_time":"发起展期时间",
                "expiry_time":"展期后应还到期时间"
            }
        ]
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 17.  展期试算
接口地址：机构提供/get-delay-interest
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
	"delay_term": "展期周期", 
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
	    "order_no":"订单编号",
	    "loan_amount":"贷款金额",
	    "interest_amount":"应还利息",
	    "expiry_amount":"逾期利息",
	    "admin_amount":"管理费，部分机构收取",
	    "delay_amount":"展期当前所需还款额",
	    "repay_amount":"还款总额",
	    "expiry_time":"到期时间",
	    "start_time":"展期费用产生的开始时间",
	    "end_time":"展期费用产生的结束时间",
	    "delay_times":"当前延期成功的数目",
	    "limit_times":"限制总延期数",
	    "delay_term"::["展期支持的周期列表"], 
	    "default_value":"默认展期周期"
	    "term_unit":"周期单位:1-天 2-月",
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}

```
### 18.  展期详情
接口地址：机构提供/get-delay-detail
```
//入参示例
{
	"order_no": "订单号(10-32位)", 
	"product_id": "产品id",
}

//返回示例

​正常返回：
{
	"code":0,
	"message":"success",
	"data":{
		"order_no":"订单编号",
	    "repay_amount":"当前需还金额",
	    "delay_amount":"展期还款金额",
	    "delay_term":"展期还款天数(对应借款的申请期限)",
	    "term_unit":"展期期限单位:1-天 2-月",
	    "expiry_time":"展期到期还款日",
	    "delay_times":"当前延期成功的数目",
	    "limit_times":"限制总延期数",
    }
}

​错误返回：
{
	"code":11007,
	"message":"tidakbisameminjam,telahditolakdalam30hariterakhir",
	"data":{}
}
```

***

### 19.  推送订单状态
接口地址：机构提供/callback-order-status
```
//入参示例
{
	"order_no": "订单编号", 
	"order_status": "订单状态", 
	"update_time": "更新时间戳"
}

//返回示例

​正常返回：
{
    "code":0,
    "message":"success",
    "data":{}
}

```

### 20.  推送审批结果
接口地址：机构提供/callback-approval-result
```
//入参示例
{
	"order_no":"订单编号",
    "order_status":订单状态,
    "approval_time":"贷款时间",
    "loan_time":"审批时间",
    "approval_amount":"审批金额",
    "approval_term":"审批周期",
    "term_unit":"周期单位:1-天 2-月",
    "interest_amount":"总利息",
    "interest_rate":"每日或每月利率",
    "actual_amount":"实际到账金额",
    "admin_amount":"总管理费",
    "repay_amount":"总还款额",
    "order_remark":"本金1600000.00，利息0，还款金额1600000"
}

//返回示例
​
​正常返回：
{
    "code":0,
    "message":"success",
    "data":{}
}

```


### 21.  推送还款计划
接口地址：机构提供/callback-repay-plan
```
//入参示例
{
	"order_no":"订单号",
    "order_status":"订单状态",
    "loan_time":"贷款时间",
    "loan_amount":"贷款金额",
    "loan_term":"贷款周期",
    "term_unit":"周期单位:1-天 2-月",
    "interest_amount":"应还利息",
    "interest_rate":"每天或每月利率",
    "expiry_time":"到期时间",
    "expiry_amount":"逾期利息",
    "is_overdue":"是否逾期 0-未逾期 1-已逾期",
    "overdue_days":"逾期天数",
    "admin_amount":"管理费，部分机构收取",
    "repay_amount":"还款总额",
    "url_repay":"是否支持URL还款，星合",
    "url_info":"url还款链接",
    "delay_repay":"是否支持展期 ,0不支持, 1支持",
    "delay_status":"展期支付状态 0 展期中, 1 展期支付成功, 2 展期支付失败, null 未发起过展期",
}

//返回示例

​正常返回：
{
    "code":0,
    "message":"success",
    "data":{}
}


```

## 四、字段枚举
### 01.  订单状态，order_status
| 枚举 | 说明 | 翻译 | 
| --- | --- | --- |
| 80  | 新订单 |    |
| 90  | 审批中 |    |
| 100  | 审批成功/放款处理中 |    |
| 110  | 审批拒绝 |    |
| 161  | 贷款取消 |    |
| 169  | 放款失败 |    |
| 170  | 放款成功/正常还款中 |    |
| 180  | 已逾期 |    |
| 200  | 还款结清 |    |


### 02.  用户类型，user_type
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 已工作 |  Bekerja   |
| 2    | 学生   |  Pelajar   |
| 3    | 其他   |  Lainnya   |

### 03.  教育程度，education
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 未受过教育 |  S3   |
| 2    | 小学 | S2    |
| 3    | 中学 |  S1   |
| 4    | 高中 |  SLTA   |
| 5    | 大学本科 | DIPLOMA_II    |
| 6    | 研究生 |  DIPLOMA_I   |

### 04.  贷款用途，loan_remark
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 生活、工作使用 |  Keperluan sehari-hari   |
| 2    | 消费使用 |   Belanja konsumtif  |
| 3    | 教育支出 |  Pendidikan   |
| 4    | 旅游使用 |   Liburan  |
| 5    | 结婚使用 |   Menikah  |
| 6    | 医疗支出 |   Pengobatan  |
| 7    | 装修 |   Renovasi rumah  |
| 8    | 宗教旅行 |  Perjalanan Religius   |
| 9    | 生育支出 |  Lahiran   |
| 10    | 多用途使用 |  Lainnya   |
| 11   | 其他 |   LAIN LAIN  |

### 05.  宗教，religion
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 穆斯林 |   Muslim  |
| 2    | 基督教 | Kristen    |
| 3    | 天主教徒 |   Katolik  |
| 4    | 印度教 | Hindu    |
| 5    | 佛教 | Buddha    |
| 6    | 儒教/孔教 |  Konghucu   |

### 06.  婚姻状态，marital_status
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 单身 |  SINGLE   |
| 2    | 已婚 | MARRIED    |
| 3    | 离婚 |  DIVORCED   |
| 4    | 丧偶 |  WIDOWED   |

### 07.  工作类型，work_type
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 企业家 |  ENTREPRENEUR   |
| 2    | 员工（私人） |  Karyawan (swasta)   |
| 3    | 员工（公共） |  Karyawan (BUMN)   |
| 4    | 专业人士 |  Ahli profesi   |
| 5    | 公民 |     |
| 6    | 学生 |  Pelajar   |
| 7    | 无工作 |  Pengangguran   |
| 8    | 自雇 |     |
| 9    | 工薪阶层 |     |
| 10    | 其他 |   OTHER  |

### 08.  紧急联系人关系，emergs_relation
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | 夫妻/配偶 |  Suami / Istri   |
| 2    | 父母 |  PARENTS   |
| 3    | 子女/孩子 |  Anak   |
| 4    | 兄弟/姐妹 |  Adik / Kakak Kandung   |
| 5    | 亲戚 |     |
| 6    | 朋友 |  Teman   |
| 7    | 同事 |  Rekan Kerja   |
| 8    | 其他 |     |

### 09.  还款账户类型，repay_account_list
| 枚举 |  说明  | 翻译 |
| --- | ----- | --- |
| 1    | ATM |     |
| 2    | 线上 |     |
| 3    | 手机银行 |     |
| 4    | 便利店 |     |
| 5    | 其他atm |     |
| 6    | 其他线上银行 |     |
| 7    | 其他手机银行 |     |







