
## <a id="research"></a>RQData 


### <a id="research-version"></a> 变更履历

版本 | 发布日期 | 新功能/API | Bug Fix | 改善
--|--|--|--|--
rqdatac-0.2.2 | 2016-12-14 | 1. 增加报错提示，更加友好 2. 增加对Python2的支持 | | 
rqdatac-0.1.29 | 2016-12-16 | 增加index_weights | | 
rqdatac-0.1.26 | 2016-11-30 | 增加get_financials | | 
rqdatac-0.1.17 | 2016-11-04 | 增加get_dominant_future, get_turnover_rate, xueqiu.history |对期货夜盘分钟数据进行了清洗，对股票历史数据源进行了替换 |get_price增加对前后复权数据以及跳过停牌期的支持，get_fundamentals函数增加了report_quarter关键字

### <a id="research-intro"></a>RQData简介

Ricequant数据SDK - RQdata是一个面向机构的商用版Python金融数据工具包。它安装在本地，为量化工程师们提供了丰富整齐的历史数据以及简单高效的API接口，最大限度地免除了您进行数据搜索、清洗的烦恼。在RQData的基础上，您可以可以方便地搭建本地运行环境并使用自己的工具链进行策略研究开发。

Ricequant为您提供的每日更新的数据包括：

 - 中国A股、ETF，中国期货（股指、国债、商品期货），美股、ETF的所有基本信息
 - 中国A股、ETF、期货过去10多年以来每日市场数据
 - 中国A股、ETF2005年以来的分钟线数据
 - 美股、ETF过去20多年以来的所有市场数据
 - 中国A股上市以来的所有财务数据
 - 中国期货2010年以来的分钟线数据
 - 舆情大数据

### <a id="research-init"></a>RQData初始化
以下的API的范例皆默认先操作了引入rqdata的模块。

互联网端调用数据(rqdatac), 需要在init的时候做用户名、密码的登录验证才可以, 如想试用的话可以联系我们的销售（email: lk@ricequant.com, qq: 285962740）获得户名和密码就可以通过互联网调用数据量化接口SDK了:

```
import rqdatacfrom rqdatac import *rqdatac.init(username, password)
```

本地部署的服务器无需做身份验证的初始化:

```
import rqdata
from rqdata import *
rqdata.init()
```
### <a id="research-owncsv-save"></a>保存自己的数据到csv文件

只需要使用以下简单的代码调用`pandas`的`pandas.DataFrame`的`to_csv`函数即可保存到csv：

```
import pandas as pd
df = pd.DataFrame(xxxx)
df.to_csv('xxxx.csv')
```
### <a id="research-notice"></a>一些注意事项



#### 期货连续合约

需要注意，由于期货合约存续的特殊性，针对每一品种的期货合约，系统中都增加了 **主力连续合约**以及**指数连续合约**两个人工合成的合约来满足使用需求。其中，主力连续合约是由该品种期货不同时期主力合约接续而成，代码以88结尾，例如IF88；指数连续合约是由当前品种全部可交易合约以累计持仓量为权重加权平均得到，代码以99结尾，例如IF99。

主力合约的判断标准：合约首次上市时，以当日收盘同品种持仓量最大者作为从第二个交易日开始的主力合约。当同品种其他合约持仓量在收盘后超过当前主力合约1.1倍时，从第二个交易日开始进行主力合约的切换。日内不会进行主力合约的切换。


#### 郑商所相关
郑商所的order_book_id我们统一做了补齐，例如CF701补齐为CF1701。

郑商所的分钟线成交额'TotalTurnover'为0，但成交量'TotalVolumeTrade'正常。

一些商品期货交易品种存在合约代码修改的问题。因为变动前后它们的合约乘数产生了变化，所以目前我们将修改前后的期货合约**当做两种合约**来处理。而实际上，它们交易的又是同一品种，价格水平具备连续性。所以在主力连续与指数连续合约的处理上，我们做了区分处理。举例来说，强麦的主力连续合约我们有'强麦主力连续（旧）'和'强麦主力连续'，分别对应'WS88'和'WH88'的orderbook_id. 一些变动的期货代码列举如下：

品种 | 品种代码（变动前）| 品种代码（变动后）
 -- | -- | --
强麦 | WS（1305及以前） | WH
普麦 | WT (1211及以前) | PM
菜籽油 | RO（1305及以前） | OI
早籼稻 | ER（1305及以前） | RI
甲醇 | ME（1505及以前） | MA
动力煤 | TC（1604及以前）| ZC




---

## <a id="research-API"></a>API文档



### <a id="research-API-all_instruments"></a>all_instruments - 获取所有合约基础信息

```
all_instruments(type='None', country='cn')
```


获取某个国家市场的所有合约信息。使用者可以通过这一方法很快地对合约信息有一个快速了解，目前仅支持中国市场。

#### <a id="research-API-all_instruments-params"></a>参数


> 参数 | 类型 | 说明
> --  | -- | --
> type | *str* | 需要查询合约类型，例如：type='CS'代表股票。默认是所有类型
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


其中type参数传入的合约类型和对应的解释如下：


> 合约类型 | 说明
> -- | --
> CS | Common Stock, 即股票
> ETF | Exchange Traded Fund, 即交易所交易基金
> LOF | Listed Open-Ended Fund，即上市型开放式基金
> FenjiMu | Fenji Mu Fund, 即分级母基金
> FenjiA | Fenji A Fund, 即分级A类基金
> FenjiB | Fenji B Funds, 即分级B类基金
> INDX | Index, 即指数
> Future | Futures，即期货，包含股指、国债和商品期货

#### <a id="research-API-all_instruments-return"></a>返回
 *pandas DataFrame* - 所有合约的基本信息。

#### <a id="research-API-all_instruments-example"></a>范例

- 获取中国市场所有合约的基础信息：

```
[In]all_instruments()
[Out]
    abbrev_symbol	order_book_id  sector_code	symbol
0	XJDQ	000400.XSHE	  Industrials	    许继电气
1	HXN	    002582.XSHE	  ConsumerStaples	好想你
2	NFGF	300004.XSHE	  Industrials	    南风股份
3	FLYY	002357.XSHE	  Industrials	    富临运业
...
```

- 获取中国市场所有分级基金的基础信息：

```
[In]all_instruments(type='FenjiA')
[Out]
    abbrev_symbol	order_book_id	product	sector_code  symbol
0	CYGA	150303.XSHE	null	null	华安创业板50A
1	JY500A	150088.XSHE	null	null	金鹰500A
2	TD500A	150053.XSHE	null	null	泰达稳健
3	HS500A	150110.XSHE	null	null	华商500A
4	QSAJ	150235.XSHE	null	null	鹏华证券A
...
```

- 获取中国市场所有期货的基础信息：

```
[In]all_instruments(type='Future')
[Out]
	abbrev_symbol	order_book_id	product	sector_code	symbol
0	MH0610	CF0610	Commodity	null	棉花0610
1	LD0209	GN0209	Commodity	null	绿豆0209
...
3615	HS1301	IF1301	Index	null	沪深1301
...
```
---


### <a id="research-API-instruments"></a>instruments - 获取合约详细信息

```
instruments(order_book_id, country='cn')
```

获取某个国家市场内一个或多个合约的详细信息。目前仅支持中国市场。

#### <a id="research-API-instruments-params"></a>参数
> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* OR *str list* | 合约代码或合约代码列表，可传入order_book_id, order_book_id list, symbol, symbol list。中国市场的order_book_id通常类似'000001.XSHE'。需要注意，国内股票、ETF、指数合约代码分别应当以'.XSHG'或'.XSHE'结尾，前者代表上证，后者代表深证。期货则无此要求
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="research-API-instruments-return"></a>返回
一个instrument对象，或一个instrument list。



目前系统**并不支持**跨国家市场的同时调用。传入 order_book_id list必须属于同一国家市场，不能混合着中美两个国家市场的order_book_id。

+ **股票，ETF，指数Instrument对象**


参数 | 类型 | 说明
-- | -- | --
order_book_id | *str* | 证券代码，证券的独特的标识符。应以'.XSHG'或'.XSHE'结尾，前者代表上证，后者代表深证
symbol | *str* | 证券的简称，例如'平安银行'
abbrev_symbol | *str* | 证券的名称缩写，在中国A股就是股票的拼音缩写。例如：'PAYH'就是平安银行股票的证券名缩写
round_lot | *int* | 一手对应多少股，中国A股一手是100股
sector_code | *str* | 板块缩写代码，全球通用标准定义
sector_code_name | *str* | 以当地语言为标准的板块代码名
shenwan_industry_code | *str* | 申万的行业分类代码
shenwan_industry_name | *str* | 申万的行业分类命名
industry_code | *str* | 国民经济行业分类代码，具体可参考下方“Industry列表”
industry_name | *str* | 国民经济行业分类名称
listed_date | *str* |该证券上市日期
de_listed_date | *str* | 退市日期
type | *str* | 合约类型，目前支持的类型有: 'CS', 'INDX', 'LOF', 'ETF', 'FenjiMu', 'FenjiA', 'FenjiB', 'Future'
concept_names | *str* | 概念股分类，例如：'铁路基建'，'基金重仓'等
exchange | *str* | 交易所，'XSHE' - 深交所, 'XSHG' - 上交所
board_type | *str* | 板块类别，'MainBoard' - 主板,'GEM' - 创业板
status | *str* | 合约状态。'Active' - 正常上市, 'Delisted' - 终止上市, 'TemporarySuspended' - 暂停上市, 'PreIPO' - 发行配售期间, 'FailIPO' - 发行失败
special_type | *str* | 特别处理状态。'Normal' - 正常上市, 'ST' - ST处理, 'StarST' - *ST代表该股票正在接受退市警告, 'PT' - 代表该股票连续3年收入为负，将被暂停交易, 'Other' - 其他





+ **期货Instrument对象**

参数 | 类型 | 说明
-- | -- | --
order_book_id | *str* | 期货代码，期货的独特的标识符（郑商所期货合约数字部分进行了补齐。例如原有代码'ZC609'补齐之后变为'ZC1609'）。主力连续合约UnderlyingSymbol+88，例如'IF88' ；指数连续合约命名规则为UnderlyingSymbol+99
symbol | *str* | 期货的简称，例如'沪深1005'
abbrev_symbol | *str* | 期货的名称缩写，例如'HS1005'。主力连续合约与指数连续合约都为'null'
round_lot | *float* | 期货全部为1.0
listed_date | *str* | 期货的上市日期。主力连续合约与指数连续合约都为'0000-00-00'
type | *str* | 合约类型，'Future'
contract_multiplier | *float* | 合约乘数，例如沪深300股指期货的乘数为300.0
underlying_order_book_id | *str* | 合约标的代码，目前除股指期货(IH, IF, IC)之外的期货合约，这一字段全部为'null'
underlying_symbol | *str* | 合约标的名称，例如IF1005的合约标的名称为'IF'
maturity_date | *str* | 期货到期日。主力连续合约与指数连续合约都为'0000-00-00'
settlement_method | *str* | 交割方式，'CashSettlementRequired' - 现金交割, 'PhysicalSettlementRequired' - 实物交割
product | *str* | 产品类型，'Index' - 股指期货, 'Commodity' - 商品期货, 'Government' - 国债期货
exchange | *str* | 交易所，'DCE' - 大连商品交易所, 'SHFE' - 上海期货交易所，'CFFEX' - 中国金融期货交易所, 'CZCE'- 郑州商品交易所


Instrument对象也支持如下方法：

+ 合约已上市天数。

```
days_from_listed(date=None)
```

默认返回合约上市距离当前日期的天数。date支持str, 
如果合约首次上市交易，天数为0；如果合约尚未上市或已经退市，则天数值为-1

+ 合约距离到期天数。

```
days_to_expire(date=None)
```

如果策略已经退市，则天数值为-1


#### <a id="research-API-instruments-example"></a>范例

- 获取单一股票合约的详细信息：

```
[In]instruments('000001.XSHE')
[Out]
Instrument(round_lot=100.0, special_type='Normal', sector_code_name='金融', board_type='MainBoard', industry_code='J66', shenwan_industry_name='银行', order_book_id='000001.XSHE', sector_code='Financials', abbrev_symbol='PAYH', industry_name='货币金融服务', exchange='XSHE', concept_names='外资背景|本月解禁|融资融券|社保重仓|券商重仓|基金重仓|保险重仓|深圳本地', listed_date='1991-04-03', de_listed_date='0000-00-00', status='Active', shenwan_industry_code='801780.INDX', symbol='平安银行', type='CS')
```


- 获取多个股票合约的详细信息：

```
[In]instruments(['000001.XSHE', '000024.XSHE'])
[Out]
[Instrument(round_lot=100.0, special_type='Normal', sector_code_name='金融', board_type='MainBoard', industry_code='J66', shenwan_industry_name='银行', order_book_id='000001.XSHE', sector_code='Financials', abbrev_symbol='PAYH', industry_name='货币金融服务', exchange='XSHE', concept_names='外资背景|本月解禁|融资融券|社保重仓|券商重仓|基金重仓|保险重仓|深圳本地', listed_date='1991-04-03', de_listed_date='0000-00-00', status='Active', shenwan_industry_code='801780.INDX', symbol='平安银行', type='CS'),
 Instrument(round_lot=100.0, special_type='Other', sector_code_name='金融', board_type='MainBoard', industry_code='K70', order_book_id='000024.XSHE', sector_code='Financials', abbrev_symbol='ZSDC', industry_name='房地产业', exchange='XSHE', concept_names='null', listed_date='1993-06-07', de_listed_date='2015-12-30', status='Delisted', symbol='招商地产', type='CS')]
```

- 获取合约已上市天数：

```
[In]instruments('000001.XSHE').days_from_listed('20160801')
[Out]
9252
```


- 获取合约距离到期天数：

```
[In]instruments('IF1608').days_to_expire('20160801')
[Out]
18
```

---

### <a id="research-API-get_price"></a>get_price - 获取合约历史数据
```
get_price(order_book_id, start_date='2013-01-04', end_date='2014-01-04', frequency='1d', fields=None, adjust_type='pre', skip_suspended =False, country='cn')
```

#### <a id="research-API-get_price-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* OR *str list* | 合约代码，可传入order_book_id, order_book_id list, symbol, symbol list
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期，默认为'2013-01-04'。交易使用时，用户必须指定
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 结束日期，默认为'2014-01-04'。交易使用时，默认为策略当前日期前一天
> frequency | *str*| 历史数据的频率。 现在支持**日/分钟级别**的历史数据，默认为'1d'。使用者可自由选取不同频率，例如'5m'代表5分钟线
> fields | *str* OR *str list* | 返回字段名称
> adjust_type | *str* | 权息修复方案。前复权 - `pre`，后复权 - `post`，不复权 - `none`，回测使用 - `internal` 需要注意，`internal`数据与回测所使用数据保持一致，**仅就拆分事件**对价格以及成交量进行了前复权处理，并未考虑分红派息对于股价的影响。所以在分红前后，价格会出现跳跃
> skip_suspended | *bool* | 是否跳过停牌数据。默认为False，不跳过，用停牌前数据进行补齐。True则为跳过停牌期。**注意**，当设置为True时，函数order_book_id只支持单个合约传入
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场



#### <a id="research-API-get_price-return"></a>返回

+ 传入一个order_book_id，多个fields，函数会返回一个*pandas DataFrame*
+ 传入一个order_book_id，一个field，函数会返回*pandas Series*
+ 传入多个order_book_id，一个field，函数会返回一个*pandas DataFrame*
+ 传入多个order_book_id，函数会返回一个*pandas Panel* 



> 参数 | 类型 | 说明
> -- | -- | --
>  open | *float* | 开盘价
>  close | *float* | 收盘价
>  high | *float* | 最高价
>  low | *float* | 最低价
>  limit_up | *float* | 涨停价
>  limit_down | *float* | 跌停价
>  total_turnover | *float* | 总成交额
> volume | *float* | 总成交量
>  acc_net_value  | *float* | 累计净值（仅限基金日线数据）
>  unit_net_value   | *float* | 单位净值（仅限基金日线数据）
>  discount_rate   | *float* | 折价率（仅限基金日线数据）
>  settlement | *float* | 结算价 （仅限期货日线数据）
>  prev_settlement | *float* | 昨日结算价（仅限期货日线数据）
>  open_interest | *float* | 累计持仓量（期货专用）
> basis_spread | *float* | 基差点数（股指期货专用，股指期货收盘价-标的指数收盘价）
> trading_date | *pandasTimeStamp* | 交易日期（仅限期货分钟线数据），对应期货夜盘的情况


#### <a id="research-API-get_price-example"></a>范例


+ 获取单一股票历史日线行情（返回*pandas DataFrame*）:

```
[In]get_price('000001.XSHE', start_date='2015-04-01', end_date='2015-04-12')
[Out]
open	close	high	low	total_turnover	volume	limit_up	limit_down
2015-04-01	10.7300	10.8249	10.9470	10.5469	2.608977e+09	236637563.0	11.7542	9.6177
2015-04-02	10.9131	10.7164	10.9470	10.5943	2.222671e+09	202440588.0	11.9102	9.7397
2015-04-03	10.6486	10.7503	10.8114	10.5876	2.262844e+09	206631550.0	11.7881	9.6448
2015-04-07	10.9538	11.4015	11.5032	10.9538	4.898119e+09	426308008.0	11.8288	9.6787
2015-04-08	11.4829	12.1543	12.2628	11.2929	5.784459e+09	485517069.0	12.5409	10.2620
2015-04-09	12.1747	12.2086	12.9208	12.0255	5.794632e+09	456921108.0	13.3684	10.9403
2015-04-10	12.2086	13.4294	13.4294	12.1069	6.339649e+09	480990210.0	13.4294	10.9877
```

+ 获取单一股票历史日线行情（**未经复权处理的原始数据**，返回*pandas DataFrame*）:

```
[In]get_price('000001.XSHE', start_date='2015-04-01', end_date='2015-04-03', adjust_type='none')
[Out]
open	close	high	low	total_turnover	volume	limit_up	limit_down
2015-04-01	15.82	15.96	16.14	15.55	2.608977e+09	164331641.0	17.33	14.18
2015-04-02	16.09	15.80	16.14	15.62	2.222671e+09	140583742.0	17.56	14.36
2015-04-03	15.70	15.85	15.94	15.61	2.262844e+09	143494132.0	17.38	14.22```

+ 获取单一股票历史分钟线收盘价（返回*pandas Series*）:

```
[In]get_price('000001.XSHE', start_date='2015-04-01', end_date='2015-04-12', fields='close')
[Out]
2015-04-01    10.8249
2015-04-02    10.7164
2015-04-03    10.7503
2015-04-07    11.4015
2015-04-08    12.1543
2015-04-09    12.2086
2015-04-10    13.4294
Name: close, dtype: float64
```

+ 获取单一股票历史**15分钟线**行情（返回*pandas DataFrame*）:

```
[In]get_price('000001.XSHE', start_date='2015-04-01', end_date='2015-04-01', frequency='15m')
[Out]
	total_turnover	close	low	open	high	volume
index						
2015-04-01 09:45:00	348982890.0	10.6215	10.6215	10.7300	10.7843	31848628.0
2015-04-01 10:00:00	234445219.0	10.5808	10.5469	10.6283	10.6350	21618354.0
2015-04-01 10:15:00	125882985.0	10.6757	10.5808	10.5808	10.7368	11540023.0
2015-04-01 10:30:00	144396901.0	10.6622	10.6215	10.6757	10.7503	13190460.0
2015-04-01 10:45:00	169238918.0	10.7368	10.6554	10.6622	10.7775	15409772.0
2015-04-01 11:00:00	131464598.0	10.7232	10.6961	10.7436	10.7503	11977847.0
2015-04-01 11:15:00	83455348.0	10.7368	10.7164	10.7300	10.7639	7589260.0
...
```


+ 获取股票列表历史日线收盘价（返回*pandas DataFrame*）:

```
[In]get_price(['000024.XSHE', '000001.XSHE', '000002.XSHE'], start_date='2015-04-01', end_date='2015-04-12', fields='close')
[Out]
000024.XSHE	000001.XSHE	000002.XSHE
2015-04-01	32.1251	10.8249	12.7398
2015-04-02	31.6400	10.7164	12.6191
2015-04-03	31.6400	10.7503	12.4891
2015-04-07	31.6400	11.4015	12.7398
2015-04-08	31.6400	12.1543	12.8327
2015-04-09	31.6400	12.2086	13.5941
2015-04-10	31.6400	13.4294	13.2969
```

+ 获取股票列表历史日线行情（返回*pandas DataPanel*）:

```
[In]get_price(['000024.XSHE', '000001.XSHE', '000002.XSHE'], start_date='2015-04-01', end_date='2015-04-12')
[Out]
<class 'rqcommons.pandas_patch.HybridDataPanel'>
Dimensions: 8 (items) x 7 (major_axis) x 3 (minor_axis)
Items axis: open to limit_down
Major_axis axis: 2015-04-01 00:00:00 to 2015-04-10 00:00:00
Minor_axis axis: 000024.XSHE to 000002.XSHE
```

---

### <a id="research-API-get_dominant_future"></a>get_dominant_future - 获取主力合约列表
```
get_dominant_future(underlying_symbol, start_date=None, end_date=None)
```
获取某一期货品种一段时间的主力合约列表。合约首次上市时，以当日收盘同品种持仓量最大者作为从第二个交易日开始的主力合约。当同品种其他合约持仓量在收盘后超过当前主力合约1.1倍时，从第二个交易日开始进行主力合约的切换。日内不会进行主力合约的切换。

#### <a id="research-API-get_dominant_future-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> underlying_symbol | *str* | 期货合约品种，例如沪深300股指期货为'IF'
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |开始日期，默认为期货品种最早上市日期后一交易日
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |结束日期，默认为当前日期

#### <a id="research-API-get_dominant_future-return"></a>返回
*Pandas.Series* - 主力合约代码列表

#### <a id="research-API-get_dominant_future-example"></a>范例

- 获取某一天的主力合约代码
```
[In]
get_dominant_future('IF', '20160801')
[Out] 
date
20160801    IF1608
```
- 获取从上市到某天之间的主力合约代码
```
[In]
get_dominant_future('IC', end_date='20150501')
[Out]
date
20150417    IC1505
20150420    IC1505
20150421    IC1505
20150422    IC1505
20150423    IC1505
20150424    IC1505
20150427    IC1505
20150428    IC1505
20150429    IC1505
20150430    IC1505
20150501    IC1505
```



---

### <a id="research-API-get_fundamentals"></a>get_fundamentals - 查询财务数据

```
get_fundamentals(query, entry_date, interval=None, report_quarter=False)
```

获取历史财务数据表格。目前支持中国市场超过400个指标，具体请参考<a href="/fundamentals" target="_blank"> 财务数据文档 </a>。目前仅支持中国市场。我们特别为该函数进行了优化，读取内存的操作会极大地提升数据的获取速度。需要注意，在ricequant上查询基本面数据时，我们是以所有年报的发布日期（announcement date）为准，因为只有财报发布后才成为市场上公开可以获取的数据。比如某公司第三季度的财报于11月10号发布，那么如果从查询日期为10月5号，也就是早于发布日期，那么返回的只是第二季度的财报数据。

#### <a id="research-API-get_fundamentals-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> query | *SQLAlchemyQueryObject* | SQLAlchmey的Query对象。其中可在'query'内填写需要查询的指标，'filter'内填写数据过滤条件。具体可参考 <a href="http://docs.sqlalchemy.org/en/rel_1_0/orm/tutorial.html#querying" target="_blank">sqlalchemy's query documentation </a>学习使用更多的方便的查询语句。从数据科学家的观点来看，sqlalchemy的使用比sql更加简单和强大
> entry_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 查询财务数据的基准开始日期
> interval | *str* | 查询财务数据的间隔。例如，填写'5y'，则代表从entry_date开始（包括entry_date）回溯5年，返回数据时间以**年**为间隔。'd' - 天，'w' - 周，'m' - 月， 'q' - 季，'y' - 年
> report_quarter | *bool* | 是否显示报告期，默认为不显示。'Q1' - 一季报，'Q2' - 半年报，'Q3' - 三季报，'Q4' - 年报

#### <a id="research-API-get_fundamentals-return"></a>返回
*pandas DataPanel* - 财务数据查询结果。


#### <a id="research-API-get_fundamentals-example"></a>范例

fundamentals是一个重要的对象，其中包括了股指指标表（eod_derivative_indicator），财务指标表（financial_indicator），利润表（income_statement），资产负债表（balance_sheet），现金流量表（cash_flow_statement）以及股票列表（stock_code）等内容。结合SQLAlchemy的查找方式，能够满足用户多种查找需求。

+ 获取某只股票2016年8月1日及以前4个季度的市盈率（pe_ratio）

```
[In]
dp = get_fundamentals(query(fundamentals.eod_derivative_indicator.pe_ratio).filter(fundamentals.stockcode == '000001.XSHE'), '2016-08-01','4q' ,report_quarter = True)
[In]
dp.minor_xs('000001.XSHE')
[Out]
		report_quarter	pe_ratio
2016-08-01	2016-Q1 	7.0768
2016-04-29	2016-Q1		6.7755
2016-01-29	2015-Q3		6.5492
2015-10-29	2015-Q3		7.3809
```

+ 获取某几只股票2015年1月10日及以前5年的营业收入（revenue）以及营业成本（cost_of_good_sold）

```
[In] 
dp = get_fundamentals(query(fundamentals.income_statement.revenue, fundamentals.income_statement.cost_of_goods_sold).filter(fundamentals.income_statement.stockcode.in_(['002478.XSHE', '000151.XSHE'])), '2015-01-10', '5y')

[Out]
<class 'pandas.core.panel.Panel'>
Dimensions: 2 (items) x 5 (major_axis) x 2 (minor_axis)
Items axis: revenue to cost_of_goods_sold
Major_axis axis: 2015-01-09 to 2011-01-10
Minor_axis axis: 002478.XSHE to 000151.XSHE

[In]
dp['revenue']
[Out]
	        002478.XSHE	    000151.XSHE
2015-01-09	2.937843e+09	1.733703e+09
2014-01-10	2.926316e+09	8.839355e+08
2013-01-10	2.616532e+09	9.488980e+08
2012-01-10	2.681016e+09	6.205934e+08
2011-01-10	2.034147e+09	4.998120e+08
```


---

### <a id="research-API-get_financials"></a>get_financials - 查询季度财务信息


```
get_financials(query, quarter, interval=None)
```

以给定一个报告期回溯的方式获取季度基础财务数据（三大表）。financials是在查询中会使用到的重要对象，功能与上述fundamentals类似。但因为get_financials为季度财务信息查询，所以financials中支持的财务表格包括利润表（income_statement），资产负债表（balance_sheet），现金流量表（cash_flow_statement），以及财务指标（financial_indicator）。除此之外，financials中还包括了股票列表（stock_code）以及公布日期（announce_date）两个指标。

#### <a id="research-API-get_financials-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> query | *SQLAlchemyQueryObject* | SQLAlchmey的Query对象。其中可在'query'内填写需要查询的指标，'filter'内填写数据过滤条件
> quarter | *str* | 财报回溯查询的起始报告期，例如'2015q2', '2015q4'分别代表2015年半年报以及年报。默认只获取当前报告期财务信息
> interval | *str* | 查询财务数据的间隔。例如，填写'5y'，则代表从报告期开始回溯5年，每年为相同报告期数据；'3q'则代表从报告期开始向前回溯3个季度

#### <a id="research-API-get_financials-return"></a>返回
*pandas DataPanel* 
*pandas DataFrame*
*pandas Series
视指定查询指标、合约数量以及回溯间隔自动做数据降维处理。

#### <a id="research-API-get_financials-example"></a>范例


+ 获取单只股票过去两个报告期的净利润

```
[In]
q = query(financials.income_statement.net_profit, financials.announce_date).filter(financials.stockcode.in_(['000002.XSHE']))
[In]
get_financials(q, '2016q3', '2q')
[Out]
         net_profit announce_date
2016q3  1.12903e+10      20161028
2016q2  7.09463e+09      20160822

```

+ 获取多只股票单一报告期的净利润

```
[In]
q = query(financials.income_statement.net_profit, financials.announce_date).filter(financials.stockcode.in_(['000002.XSHE', '601988.XSHG']))
[In]
get_financials(q, '2016q3')
[Out]
		net_profit    announce_date
601988.XSHG  1.51558e+11      20161027
000002.XSHE  1.12903e+10      20161028

```

+ 获取某只股票相同季度的历史财务信息

```
[In]
q = query(financials.income_statement.net_profit).filter(financials.stockcode.in_(['000002.XSHE']))
[In]
get_financials(q, '2016q3', '3y')

[Out]
2016q3    1.12903e+10
2015q3    9.53862e+09
2014q3      7.605e+09
```

+ 获取多只股票多个报告期的多个财务指标

```
[In]
q = query(financials.income_statement.net_profit, financials.balance_sheet.prepayment).filter(financials.stockcode.in_(['000002.XSHE', '601988.XSHG']))
[In]
get_financials(q, '2016q3', '3y')
[Out]
<class 'pandas.core.panel.Panel'>
Dimensions: 2 (items) x 3 (major_axis) x 2 (minor_axis)
Items axis: net_profit to prepayment
Major_axis axis: 2016q3 to 2014q3
Minor_axis axis: 601988.XSHG to 000002.XSHE
```

---


### <a id="research-API-sector"></a>sector - 获取某板块股票列表

```
sector(code, country='cn')
```

获得属于某一板块的所有股票列表。
#### <a id="research-API-sector-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> code| *str* OR *sector_code items* | 板块名称或板块代码。例如，能源板块可填写'Energy'、'能源'或sector_code.Energy
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场。

#### <a id="research-API-sector-return"></a>返回
属于该板块的股票order_book_id或order_book_id list.

#### <a id="research-API-sector-example"></a>范例

```
[In]sector('Energy')
[Out]
['300023.XSHE', '000571.XSHE', '600997.XSHG', '601798.XSHG', '603568.XSHG', .....]
[In]sector(sector_code.Energy)
[Out]
['300023.XSHE', '000571.XSHE', '600997.XSHG', '601798.XSHG', '603568.XSHG', .....]
```

#### <a id="research-API-sector-category"></a>板块分类列表

目前支持的板块分类如下，其取值参考自MSCI发布的<a href="https://en.wikipedia.org/wiki/Global_Industry_Classification_Standard" target="_blank">全球行业标准分类</a>:

板块代码 | 中文板块名称 | 英文板块名称
-- | -- | --
Energy | 能源 | energy
Materials | 原材料 | materials
ConsumerDiscretionary | 非必需消费品 | consumer discretionary
ConsumerStaples | 必需消费品 | consumer staples
HealthCare | 医疗保健 | health care
Financials | 金融 | financials
InformationTechnology | 信息技术 | information technology
TelecommunicationServices | 电信服务 | telecommunication services
Utilities | 公共服务 | utilities
Industrials | 工业 | industrials




---

### <a id="research-API-shenwan-industry"></a>shenwan_industry - 获取申万行业股票列表

```
shenwan_industry(shenwan_industry_code/shenwan_industry_name, date)
```

通过shenwan_industry_code或shenwan_indstry_name传入，拿到某个日期的申万宏源的行业分类股票列表

#### <a id="research-API-shenwan-industry-params"></a>参数
> 参数 | 类型 | 说明
> -- | -- | --
> shenwan_industry_code/shenwan_industry_name | *str*  | 申万行业分类的代码或名称，比如shenwan_industry_code = "801010.INDX"，shenwan_industry_name="农林牧渔"
> date | *str*, *datetime.date*, *datetime.datetime*, *pandas Timestamp* | 查询日期，默认为当前最新日期

#### <a id="research-API-shenwan-industry-return"></a>返回

所属目标行业的order_book_id list

#### <a id="research-API-shenwan-industry-sampels"></a>范例

- 得到当前今天最新银行申万行业分类的股票列表：

```
[In]
shenwan_industry('银行')
[Out]
['000001.XSHE', '600000.XSHG', '600015.XSHG', '600016.XSHG', '600036.XSHG', '601988.XSHG', '601398.XSHG', '601166.XSHG', '601998.XSHG', '601328.XSHG', '002142.XSHE', '601009.XSHG', '601169.XSHG', '601939.XSHG', '601288.XSHG', '601818.XSHG']
```

- 得到'2015-01-04'的银行申万行业分类的股票列表：

```
[In]
shenwan_industry('银行', date='2015-01-04')
[Out]
['000001.XSHE', '600000.XSHG', '600015.XSHG', '600016.XSHG', '600036.XSHG', '601988.XSHG', '601398.XSHG', '601166.XSHG', '601998.XSHG', '601328.XSHG', '002142.XSHE', '601009.XSHG', '601169.XSHG', '601939.XSHG', '601288.XSHG', '601818.XSHG']
```

- 得到'2015-01-04'的'000001.XSHE'的申万行业分类

```
[In]
ins = instruments('000001.XSHE')

[In]
ins.shenwan_industry('2015-01-04')

[Out]
{'name': '银行', 'code': '801780.INDX'}

[In]
ins.shenwan_industry_name
[Out]
'银行'
```

#### <a id="research-API-shenwan-industry-table"></a>申万行业分类列表

**注意：** 申万在2014年初做过一次重大调整，因此有些行业是在那个时候新加入的

<a href="http://www.swsindex.com/pdf/swhyflsm.pdf" target="_blank"> 申银万国行业分类标准 </a>

申万行业指数代码 | 申万行业指数名
-- | --
801010.INDX | 农林牧渔
801020.INDX | 采掘
801030.INDX | 化工
801040,INDX | 钢铁
801050.INDX | 有色金属
801080.INDX | 电子
801110.INDX | 家用电器
801120.INDX | 食品饮料
801130.INDX | 纺织服装
801140.INDX | 轻工制造
801150.INDX | 医药生物
801160.INDX | 公用事业
801170.INDX | 交通运输
801180.INDX | 房地产
801200.INDX | 商业贸易
801210.INDX | 休闲服务
801230.INDX | 综合
801710,INDX | 建筑材料
801720.INDX | 建筑装饰
801730.INDX | 电气设备
801740.INDX | 国防军工
801750.INDX | 计算机
801760.INDX | 传媒
801770.INDX | 通信
801780.INDX | 银行
801790.INDX | 非银金融
801880.INDX | 汽车
801890.INDX | 机械设备


---

### <a id="research-API-industry"></a>industry - 获取某行业股票列表

```
industry(code, country='cn')
```

获得属于某一行业的所有股票列表。
#### <a id="research-API-industry-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> code| *str* OR *industry_code items* | 行业名称或行业代码。例如，农业可填写industry_code.A01 或 'A01'
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="research-API-industry-return"></a>返回
属于该行业的股票order_book_id或order_book_id list.

#### <a id="research-API-industry-example"></a>范例


```
[In]
industry('A01')
[Out]
['600540.XSHG', '600371.XSHG', '600359.XSHG', '600506.XSHG',...]
[In]
industry(industry_code.A01)
[Out]
['600540.XSHG', '600371.XSHG', '600359.XSHG', '600506.XSHG',...]
```

#### <a id="research-API-industry-category"></a>行业分类列表

我们目前使用的行业分类来自于中国国家统计局的<a href="http://www.stats.gov.cn/tjsj/tjbz/hyflbz/" target="_blank">国民经济行业分类</a>，可以使用这里的任何一个行业代码来调用行业的股票列表：

行业代码 | 行业名称
-- | --
A01 | 农业
A02 | 林业
A03 | 畜牧业
A04 | 渔业
A05 | 农、林、牧、渔服务业
B06 | 煤炭开采和洗选业
B07 | 石油和天然气开采业
B08 | 黑色金属矿采选业
B09 | 有色金属矿采选业
B10 | 非金属矿采选业
B11 | 开采辅助活动
B12 | 其他采矿业
C13 | 农副食品加工业
C14 | 食品制造业
C15 | 酒、饮料和精制茶制造业
C16 | 烟草制品业
C17 | 纺织业
C18 | 纺织服装、服饰业
C19 | 皮革、毛皮、羽毛及其制品和制鞋业
C20 | 木材加工及木、竹、藤、棕、草制品业
C21 | 家具制造业
C22 | 造纸及纸制品业
C23 | 印刷和记录媒介复制业
C24 | 文教、工美、体育和娱乐用品制造业
C25 | 石油加工、炼焦及核燃料加工业
C26 | 化学原料及化学制品制造业
C27 | 医药制造业
C28 | 化学纤维制造业
C29 | 橡胶和塑料制品业
C30 | 非金属矿物制品业
C31 | 黑色金属冶炼及压延加工业
C32 | 有色金属冶炼和压延加工业
C33 | 金属制品业
C34 | 通用设备制造业
C35 | 专用设备制造业
C36 | 汽车制造业
C37 | 铁路、船舶、航空航天和其它运输设备制造业
C38 | 电气机械及器材制造业
C39 | 计算机、通信和其他电子设备制造业
C40 | 仪器仪表制造业
C41 | 其他制造业
C42 | 废弃资源综合利用业
C43 | 金属制品、机械和设备修理业
D44 | 电力、热力生产和供应业
D45 | 燃气生产和供应业
D46 | 水的生产和供应业
E47 | 房屋建筑业
E48 | 土木工程建筑业
E49 | 建筑安装业
E50 | 建筑装饰和其他建筑业
F51 | 批发业
F52 | 零售业
G53 | 铁路运输业
G54 | 道路运输业
G55 | 水上运输业
G56 | 航空运输业
G57 | 管道运输业
G58 | 装卸搬运和运输代理业
G59 | 仓储业
G60 | 邮政业
H61 | 住宿业
H62 | 餐饮业
I63 | 电信、广播电视和卫星传输服务
I64 | 互联网和相关服务
I65 | 软件和信息技术服务业
J66 | 货币金融服务
J67 | 资本市场服务
J68 | 保险业
J69 | 其他金融业
K70 | 房地产业
L71 | 租赁业
L72 | 商务服务业
M73 | 研究和试验发展
M74 | 专业技术服务业
M75 | 科技推广和应用服务业
N76 | 水利管理业
N77 | 生态保护和环境治理业
N78 | 公共设施管理业
O79 | 居民服务业
O80 | 机动车、电子产品和日用产品修理业
O81 | 其他服务业
P82 | 教育
Q83 | 卫生
Q84 | 社会工作
R85 | 新闻和出版业
R86 | 广播、电视、电影和影视录音制作业
R87 | 文化艺术业
R88 | 体育
R89 | 娱乐业
S90 | 综合


---



### <a id="concept-API-concept"></a>concept - 获取某概念股票列表

```
concept(concept_name1, concept_name2, ...)
```

获得属于某个或某几个概念的股票列表。

#### <a id="concept-API-concept-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> concept_names | *str* OR multiple *str* | 概念名称。可以从概念列表中选择一个或多个概念填写

#### <a id="concept-API-concept-return"></a>返回
属于该概念的股票order_book_id或order_book_id list.


#### <a id="concept-API-concept-example"></a>范例

```
[In]
concept('民营医院')
[Out]
['600105.XSHG',
 '002550.XSHE',
 '002004.XSHE',
 '002424.XSHE',
 ...]

[In]
concept('民营医院', '国企改革')
[Out]
['601607.XSHG',
 '600748.XSHG',
 '600630.XSHG',
 ...]
```


#### <a id="concept-API-concept-example-category"></a>概念列表


```
含H股		深圳本地		含B股		农村金融		东亚自贸		海工装备		绿色照明		稀土永磁		内贸规划			3D打印
页岩气		三网融合		风能概念		金融改革		猪肉			水域改革		风能			赛马概念		社保重仓		物联网
民营医院		黄河三角		固废处理		甲型流感		丝绸之路		融资融券		黄金概念		抗癌			国企改革		碳纤维
保障房		智能电网		石墨烯		空气治理		京津冀		分拆上市		装饰园林		振兴沈阳		智能家居		阿里概念
股期概念		新能源		生物疫苗		特斯拉		国产软件		互联金融		锂电池		保险重仓		粤港澳		自贸区
安防服务		广东自贸		汽车电子		超大盘		低碳经济		云计算		婴童概念		建筑节能		土地流转		智能机器
未股改		触摸屏		天津自贸		生物质能		前海概念		抗流感		卫星导航		多晶硅		出口退税		参股金融
准ST股		食品安全		智能穿戴		业绩预降		污水处理		重组概念		上海自贸		外资背景		信托重仓		本月解禁
体育概念		维生素		基金重仓		充电桩		IPV6概念		资产注入		生态农业		基因概念		图们江		O2O模式
铁路基建		摘帽概念		股权激励		电子支付		机器人概念	油气改革		风沙治理		央企50		水利建设		养老概念
QFII重仓		迪士尼		业绩预升		宽带提速		长株潭		超导概念		网络游戏		含可转债		4G概念		送转潜力
奢侈品		新三板		皖江区域		核电核能		海峡西岸		次新股		高校背景		券商重仓		基因测序		节能
三沙概念		日韩贸易		氢燃料		陕甘宁		文化振兴		民营银行		苹果概念		稀缺资源		基因芯片		循环经济
聚氨酯		金融参股		沿海发展		智能交通		海上丝路		ST板块		涉矿概念		蓝宝石		博彩概念		电商概念
整体上市		草甘膦		创投概念		超级细菌		信息安全		生物燃料		武汉规划		节能环保		成渝特区		军工航天
地热能		上海本地		生物育种		燃料电池		海水淡化
```


---

### <a id="research-API-index-components"></a>index_components - 获取指数成分股列表

```
index_components(order_book_id, date=None, country='cn')
```

获取某一指数的股票构成列表，也支持指数的历史构成查询。
#### <a id="research-API-index-components-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* | 指数代码，可传入order_book_id，例如'000001.XSHG'或'沪深300'。目前所支持的指数列表可以参考<a href="/data/index#Data-Index-China_index_list" target="_blank">指数数据表</a>
> date | *str*, *datetime.date*, *datetime.datetime*, *pandas Timestamp* | 查询日期，默认为最新记录日期
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-index-components-return"></a>返回
构成该指数股票的order_book_id list

#### <a id="research-API-index-components-example"></a>范例

```
[In]
index_components('000001.XSHG')
[Out]
['600000.XSHG',
 '600004.XSHG',
 ...]

[In]
index_components('000001.XSHG', date='2015-01-01')
[Out]
['600613.XSHG',
 '600239.XSHG',
 ...]
```

---

### <a id="research-API-index-weights"></a>index_weights - 获取指数历史权重

```
index_weights(order_book_id, date=None)
```

获取某一指数的历史构成以及权重。**注意**，该数据为月度更新。
#### <a id="research-API-index_weights-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* | 指数代码，可传入order_book_id，例如'000001.XSHG'或'沪深300'。目前所支持的指数列表可以参考<a href="/data/index#Data-Index-China_index_list" target="_blank">指数数据表</a>
> date | *str*, *datetime.date*, *datetime.datetime*, *pandas Timestamp* | 查询日期，默认为最新记录日期


#### <a id="research-API-index_weights-return"></a>返回
*pandas Series*，每只股票在指数中的构成权重。

#### <a id="research-API-index_weights-example"></a>范例

+ 获取上证50指数在距离20160801最近的一次指数构成结果：

```
[In]
index_weights('000016.XSHG', '20160801')
[Out]
rder_book_id
600000.XSHG    0.03750
600010.XSHG    0.00761
600016.XSHG    0.05981
600028.XSHG    0.01391
600029.XSHG    0.00822
600030.XSHG    0.03526
600036.XSHG    0.04889
600050.XSHG    0.00998
600104.XSHG    0.02122
```

---
### <a id="research-API-is_suspended"></a>is_suspended - 判断股票是否全天停牌
```
is_suspended(order_book_id, start_date=None, end_date=None, country='cn')
```
判断某只股票在一段时间（包含起止日期）是否全天停牌。

#### <a id="research-API-is_suspended-params"></a>参数

 >参数 | 类型 | 说明
 > --  | -- | --
 >order_book_id | *str*  | 某只股票的代码或股票代码列表，传入单只股票的order_book_id或symbol
 >start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期，默认为股票上市日期
 >end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |  结束日期，默认为当前日期，如果股票已经退市，则为退市日期
 > country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="research-API-is_suspended-return"></a>返回
*pandas DataFrame*
如果在查询期间内股票尚未上市，或已经退市，则函数返回None；如果开始日期早于股票上市日期，则以股票上市日期作为开始日期。

#### <a id="research-API-is_suspended-example"></a>范例

+ 获取武钢股份从2016年6月24日至今（2016年8月31日）的停牌情况：

```
[In]
is_suspended('武钢股份', start_date='20160624')
[Out]
               武钢股份
2016-06-24       False
2016-06-27        True
2016-06-28        True
2016-06-29        True
2016-06-30        True
2016-07-01        True
2016-07-04        True
2016-07-05        True
2016-07-06        True
...
2016-08-30        True
2016-08-31        True
```



---

### <a id="research-API-is_st_stock"></a>is_st_stock - 查询股票是否为ST股

```
is_st_stock(order_book_id, start_date, end_date)
```

判断一只或多只股票在一段时间（包含起止日期）内是否为ST股。

ST股包括如下:
- S\*ST-公司经营连续三年亏损，退市预警+还没有完成股改;
- \*ST-公司经营连续三年亏损，退市预警;
- ST-公司经营连续二年亏损，特别处理;
- SST-公司经营连续二年亏损，特别处理+还没有完成股改;

#### <a id="research-API-is_st_stock-params"></a>参数
> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* or *str list* | 某只股票的代码或股票代码列表，可传入order_book_id, order_book_id list, symbol, symbol list
> start_date |  *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期
> end_date |  *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |  结束日期

#### <a id="research-API-is_st_stock-return"></a>返回
*pandas DataFrame* - 查询时间段内是否为ST股的查询结果

#### <a id="research-API-is_st_stock-example"></a>范例

```
[In]
is_st_stock("002336.XSHE", "20160411", "20160510")
[Out]
	        002336.XSHE
2016-04-11	False
2016-04-12	False
...
2016-05-09	True
2016-05-10	True

[In]
is_st_stock(["002336.XSHE", "000001.XSHE"], "2016-04-11", "2016-05-10")
[Out]
	  002336.XSHE	000001.XSHE
2016-04-11	False	False
2016-04-12	False	False
...
2016-05-09	True	False
2016-05-10	True	False

```


---

### <a id="research-API-get_yield_curve"></a>get_yield_curve - 获取收益率曲线

```
get_yield_curve(start_date='2013-01-04', end_date='2014-01-04', tenor=None, country='cn')
```

获取某个国家市场在一段时间内收益率曲线水平（包含起止日期）。目前仅支持中国市场。

数据为2002年至今的**中债国债收益率曲线**，来源于中央国债登记结算有限责任公司。

#### <a id="research-API-get_yield_curve-params"></a>参数
> 参数 | 类型 | 说明
> --  | -- | --
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期，默认为'2013-01-04'
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 结束日期，默认为'2014-01-04'
> tenor | *str* | 标准期限，'0S' - 隔夜，'1M' - 1个月，'1Y' - 1年
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-get_yield_curve-return"></a>返回
*pandas DataFrame* - 查询时间段内无风险收益率曲线

#### <a id="research-API-get_yield_curve-example"></a>范例

```
[In]
get_yield_curve(start_date='20130104', end_date='20140104')

[Out]
                0S      1M      2M      3M      6M      9M      1Y      2Y  \
2013-01-04  0.0196  0.0253  0.0288  0.0279  0.0280  0.0283  0.0292  0.0310
2013-01-05  0.0171  0.0243  0.0286  0.0275  0.0277  0.0281  0.0288  0.0305
2013-01-06  0.0160  0.0238  0.0285  0.0272  0.0273  0.0280  0.0287  0.0304

                3Y      4Y   ...        6Y      7Y      8Y      9Y     10Y  \
2013-01-04  0.0314  0.0318   ...    0.0342  0.0350  0.0353  0.0357  0.0361
2013-01-05  0.0309  0.0316   ...    0.0342  0.0350  0.0352  0.0356  0.0360
2013-01-06  0.0310  0.0315   ...    0.0340  0.0350  0.0352  0.0356  0.0360
...
```

---
### <a id="research-API-get_turnover_rate"></a>get_turnover_rate - 获取历史换手率
```
get_turnover_rate(order_book_id, start_date, end_date, fields=None)
```
#### <a id="research-API-get_turnover_rate-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* or *str list* | 可输入order_book_id, order_book_id list, symbol, symbol list
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 开始日期，用户必须指定
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 结束日期，用户必须指定
> fields | *str* | 默认为所有字段。当天换手率 - `today`，过去一周平均换手率 - `week`，过去一个月平均换手率 - `month`，过去三个月平均换手率 - `three_month`，过去六个月平均换手率 - `six_month`，过去一年平均换手率 - `year`，当年平均换手率 - `current_year`，上市以来平均换手率 - `total`

#### <a id="research-API-get_turnover_rate-return"></a>返回
+ 如果只传入一个order_book_id，多个fields，返回*pandas DataFrame*
+ 如果传入order_book_id list，并指定单个field，函数会返回一个*pandas DataFrame* 
+ 如果传入order_book_id list，并指定多个fields，函数会返回一个*pandas Panel* 


#### <a id="research-API-get_turnover_rate-example"></a>范例
- 获取平安银行历史换手率情况
```
[In]
get_turnover_rate('000001.XSHE', '20160801', '20160806')
[Out]
             today    week   month  three_month  six_month    year  \
2016-08-01  0.5190  0.4478  0.3213       0.2877     0.3442  0.5027   
2016-08-02  0.3070  0.4134  0.3112       0.2843     0.3427  0.5019   
2016-08-03  0.2902  0.3460  0.3102       0.2823     0.3432  0.4982   
2016-08-04  0.9189  0.4938  0.3331       0.2914     0.3482  0.4992   
2016-08-05  0.4962  0.5031  0.3426       0.2960     0.3504  0.4994   

            current_year   total  
2016-08-01        0.3585  1.1341  
2016-08-02        0.3570  1.1341  
2016-08-03        0.3565  1.1339  
2016-08-04        0.3604  1.1339  
2016-08-05        0.3613  1.1338  
``` 



- 获取平安银行与中信银行一段时间内的周平均换手率

```
[In]
get_turnover_rate(['000001.XSHE', '601998.XSHG'], '20160801', '20160812', 'week')

[Out]
  000001.XSHE  601998.XSHG
2016-08-01       0.4478       0.1176
2016-08-02       0.4134       0.1175
2016-08-03       0.3460       0.0972
2016-08-04       0.4938       0.0937
2016-08-05       0.5031       0.0927
2016-08-08       0.4414       0.0754
2016-08-09       0.4357       0.0746
2016-08-10       0.4377       0.0779
2016-08-11       0.3679       0.1212
2016-08-12       0.4779       0.1391
```

---


### <a id="research-API-get_dividend"></a>get_dividend - 获取股票分红数据

```
get_dividend(order_book_id, start_date=None, end_date=None, adjusted=True, country='cn')
```

获取某只股票在一段时间内的分红情况（包含起止日期）。如未指定日期，则默认所有。目前仅支持中国市场。
#### <a id="research-API-get_dividend-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* | 可输入order_book_id或symbol
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 开始日期
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 结束日期
> adjusted | *bool* | 默认为True，代表经过前复权处理之后的分红数值；False代表未经任何复权处理的原始分红数值
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-get_dividend-return"></a>返回
*pandas DataFrame* - 查询时间段内某个股票的分红数据

- declaration_announcement_date: 分红宣布日，上市公司一般会提前一段时间公布未来的分红派息事件
- book_closure_date: 股权登记日
- dividend_cash_before_tax: 税前分红
- ex_dividend_date: 除权除息日，该天股票的价格会因为分红而进行调整
- payable_date: 分红到帐日，这一天最终分红的现金会到账
- round_lot: 分红最小单位，例如：10代表每10股派发dividend_cash_before_tax单位的税前现金

#### <a id="research-API-get_dividend-example"></a>范例
- 获取平安银行2013-01-04 到 2014-01-06的**经过前复权处理的**分红数据：

```
[In]
get_dividend('000001.XSHE', start_date='20130104', end_date='20140106')

[Out]
                              book_closure_date  dividend_cash_before_tax  \
declaration_announcement_date
2013-06-14                           2013-06-19                    0.9838

                              ex_dividend_date payable_date  round_lot
declaration_announcement_date
2013-06-14                          2013-06-20   2013-06-20       10.0
```



---

### <a id="research-API-get_split"></a>get_split - 获取股票拆分数据

```
get_split(order_book_id, start_date=None, end_date=None, country='cn')
```

获取某只股票在一段时间内的拆分情况（包含起止日期），如未指定日期，则默认所有。目前仅支持中国市场。

#### <a id="research-API-get_split-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* | 可输orderbook_id, symbol
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 开始日期
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  | 结束日期
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="research-API-get_split-return"></a>返回

*pandas DataFrame* - 查询时间段内的某个股票的拆分数据

- ex_dividend_date: 除权除息日，该天股票的价格会因为拆分而进行调整
- book_closure_date: 股权登记日
- split_coefficient_from: 拆分因子（拆分前）
- split_coefficient_to: 拆分因子（拆分后）

例如：每10股转增2股，则split_coefficient_from = 10, split_coefficient_to = 12.

#### <a id="research-API-get_split-return"></a>范例

- 获取平安银行2010-01-04 到 当天之间的拆分信息：

```
[In]
get_split('000001.XSHE', start_date='20100104', end_date='20140104')

[Out]
                 book_closure_date payable_date  split_coefficient_from  \
ex_dividend_date
2013-06-20              2013-06-19   2013-06-20                      10

                  split_coefficient_to
ex_dividend_date
2013-06-20                        16.0
```

---

### <a id="research-API-get_ex_factor"></a>get_ex_factor - 获取复权因子

```
get_ex_factor(order_book_id, start_date=None, end_date=None, country='cn')
```

获取复权因子。如未指定日期，则默认所有。

#### <a id="research-API-get_ex_factor-param"></a>参数


> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id | *str* | 可输入order_book_id或symbol
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 结束日期
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-get_ex_factor-return"></a>返回

*pandas dataframe* - 包含了复权因子的日期和对应的各项数值

- ex_date: 除权除息日
- order_book_id: 证券代码，证券的独特的标识符
- ex_factor: 复权因子，考虑了分红派息与拆分的影响，为一段时间内的股价调整乘数。举例来说，平安银行（'000001.XSHE'）在2016年6月15日每10股派发现金股利人民币 1.53元（含税），并以资本公积转增股本每10股转增2股。6月15日的收盘价为10.44元，其除权除息后的价格应当为 (10.44-1.53/10) / 1.2 = 8.5725.本期复权因子为10.44 / 8.5725 = 1.217847
- ex_cum_factor: 累计复权因子，X日所在期复权因子 = 当前最新累计复权因子 / 截至X日最新累计复权因子。举例来说，2016年5月05日所在期复权因子 = 122.424143 / 100.525060 = 1.217847
- ex_end_date: 复权因子所在期的截止日期

#### <a id="research-API-get_ex_factor-example"></a>范例

```
[In]
get_ex_factor('000001.XSHE', start_date='2013-01-04', end_date='2017-01-04')

[Out]
            order_book_id  ex_factor  ex_cum_factor announcement_date  \
ex_date
2013-06-20   000001.XSHE   1.614263      68.255824        2013-06-19
2014-06-12   000001.XSHE   1.216523      83.034780        2014-06-11
2015-04-13   000001.XSHE   1.210638     100.525060        2015-04-10
2016-06-16   000001.XSHE   1.217847     122.424143        2016-06-15

           ex_end_date
ex_date
2013-06-20  2014-06-11
2014-06-12  2015-04-12
2015-04-13  2016-06-15
2016-06-16         NaT

```

---
### <a id="research-API-get_trading_dates"></a>get_trading_dates - 获取交易日列表
```
get_trading_dates(start_date, end_date, country='cn')
```

获取某个国家市场的交易日列表（起止日期加入判断）。目前仅支持中国市场。

#### <a id="research-API-get_trading_dates-params"></a>参数
> 参数 | 类型 | 说明
> --  | -- | --
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |开始日期
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |结束日期
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-get_trading_dates-return"></a>返回
*datetime.date list* - 交易日期列表

#### <a id="research-API-get_trading_dates-example"></a>范例

```
[In]
get_trading_dates(start_date='20160505', end_date='20160505')
[Out]
[datetime.date(2016, 5, 5)]
```

---
### <a id="research-API-get_previous_trading_date"></a>get_previous_trading_date - 获取上一交易日

```
get_previous_trading_date(date, country='cn')
```

获取指定日期的上一交易日。


#### <a id="research-API-get_previous_trading_date-params"></a>参数
> 参数 | 类型 | 说明
> --  | -- | --
> date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp*  |指定日期
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场


#### <a id="research-API-get_previous_trading_date-return"></a>返回
*datetime.date* - 交易日期

#### <a id="research-API-get_previous_trading_date-example"></a>范例

```
[In]
get_previous_trading_date('20160502')
[Out]
[datetime.date(2016, 4, 29)]
```

---

### <a id="research-API-get_next_trading_date"></a>get_next_trading_date - 获取下一交易日

```
get_next_trading_date(date, country='cn')
```

获取指定日期的下一交易日。

#### <a id="research-API-get_next_trading_date-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* |指定日期
> country | *str* | 目前只支持中国市场 ('cn')

#### <a id="research-API-get_next_trading_date-return"></a>返回
*datetime.date* - 交易日期

#### <a id="research-API-get_next_trading_date-example"></a>范例

```
[In]
get_next_trading_date(date='2016-05-01')
[Out]
[datetime.date(2016, 5, 3)]
```

---



### <a id="research-API-fenji-get_a_by_yield"></a>fenji.get_a_by_yield - 获取分级A基金列表（利率水平查询）

```
fenji.get_a_by_yield(current_yield, listing=True)
```

获取对应利率水平的分级A基金列表。

#### <a id="research-API-fenji-get_a_by_yield-params"></a>参数

> 参数 | 类型 | 说明
>  -- | -- | --
> current_yield | *float*  | 本期利率，例如5代表5%
> listing | *bool* | 当前分级基金是否在交易所可交易，默认为True

#### <a id="research-API-fenji-get_a_by_yield-return"></a>返回

符合当前利率水平的分级A基金的order_book_id list；如果无符合内容，则返回空列表。

#### <a id="research-API-fenji-get_a_by_yield-example"></a>范例

```
[In] fenji.get_a_by_yield(4)
[Out]
['150039.XSHE']
```

### <a id="research-API-fenji-get_a_by_interest_rule"></a>fenji.get_a_by_interest_rule - 获取分级A基金列表（利率规则查询）


```
fenji.get_a_by_interest_rule(interest_rule))
```

获取对应利率规则的分级A基金列表，如果无符合内容，则返回空列表。
#### <a id="research-API-fenji-get_a_by_interest_rule-params"></a>参数

> 参数 | 类型 | 说明
> -- | -- | --
> interest_rule | *str* | 利率规则，例如：'+3.5%', '+4%', '=7%', '*1.4+0.55%', '利差'等。 您也可以在研究平台使用fenji.get_all来进行查询所有的组合可能。用户必须填写
> listing | *bool* | 该分级基金是否在交易所可交易，默认为True

#### <a id="research-API-fenji-get_a_by_interest_rule-return"></a>返回

符合当前利率规则的分级A基金的order_book_id list

#### <a id="research-API-fenji-get_a_by_interest_rule-example"></a>范例


```
[In] 
fenji.get_a_by_interest_rule("+3%")
[Out]
['502011.XSHG', '150215.XSHE', '150181.XSHE', '150269.XSHE', '150173.XSHE', '150217.XSHE', '502027.XSHG', '150255.XSHE', '150257.XSHE', '150237.XSHE', '150100.XSHE', '150177.XSHE', '502017.XSHG', '150279.XSHE', '150271.XSHE', '150051.XSHE', '150245.XSHE', '150233.XSHE', '502004.XSHG', '150200.XSHE', '150205.XSHE', '150184.XSHE', '502049.XSHG', '150207.XSHE', '150313.XSHE', '150243.XSHE', '150239.XSHE', '150273.XSHE', '150227.XSHE', '150076.XSHE', '150203.XSHE', '150209.XSHE', '150259.XSHE', '150315.XSHE', '150283.XSHE', '150241.XSHE', '150229.XSHE', '150307.XSHE', '150186.XSHE', '150231.XSHE', '502024.XSHG', '502007.XSHG', '150305.XSHE', '150018.XSHE', '150309.XSHE', '150311.XSHE', '150235.XSHE', '150143.XSHE', '150249.XSHE', '150329.XSHE', '150251.XSHE', '150169.XSHE', '150357.XSHE', '150194.XSHE', '150179.XSHE', '150164.XSHE', '150192.XSHE', '150171.XSHE', '150022.XSHE', '150275.XSHE', '150092.XSHE', '150277.XSHE']
```

### <a id="research-API-fenji-get_all"></a>fenji.get_all - 获取所有分级基金信息

```
fenji.get_all(field_list)
```

获取当前市场上所有分级基金的信息。

#### <a id="research-API-fenji-get_all-params"></a>参数
> 参数 | 类型 | 注释
> -- | -- | --
> field_list | *str list* | 希望输出的数据字段名（见下表），默认为所有字段

#### <a id="research-API-fenji-get_all-return"></a>返回

*pandas DataFrame* - 分级基金各项数据

字段名 | 注释
-- | --
a_b_propotion | 分级A：分级B的比例
conversion_date | 下次定折日
creation_date | 创立日期
current_yield | 本期利率
expire_date | 到期日，可能为NaN - 即不存在
fenji_a_order_book_id | A基代码
fenji_a_symbol | A基名称
fenji_b_order_book_id | B基代码
fenji_b_symbol | B基名称
fenji_mu_orderbook_id | 母基代码
fenji_mu_symbol | 母基名称
interest_rule | 利率规则
next_yield | 下期利率
track_index_symbol | 跟踪指数

#### <a id="research-API-fenji-get_all-example"></a>范例

```
[In] 
fenji.get_all()
[Out]
a_b_propotion	conversion_date	creation_date	current_yield	expire_date	fenji_a_order_book_id	fenji_a_symbol	fenji_b_order_book_id	fenji_b_symbol	fenji_mu_orderbook_id	fenji_mu_symbol	interest_rule	next_yield	track_index_symbol
0	7:3	2016-11-19	2014-05-22	2.5	NaN	161828	永益A	150162.XSHE	永益B	161827	银华永益	+1%	NaN	综合指数
1	1:1	2017-01-04	2015-03-17	5	NaN	150213.XSHE	成长A级	150214.XSHE	成长B级	161223	国投成长	+3.5%	5	创业成长
2	1:1	2016-12-15	2015-07-01	5.5	NaN	150335.XSHE	军工股A	150336.XSHE	军工股B	161628	融通军工	+4%	5.5	中证军工

[In] 
fenji.get_all(field_list = ['fenji_a_order_book_id', 'current_yield'])
[Out]
current_yield	fenji_a_order_book_id
0	2.5	161828
1	5	150213.XSHE
2	5.5	150335.XSHE
```

---





## <a id="other-data-API"></a>其他数据API

除了已有的IPython Notebook API之外，我们还提供了可以查询其他“神奇”数据的可能性，包括舆情数据以及以后会推出的电商、搜索数据。这会让你的策略和数据研究更加丰富多彩，enjoy!

---

### <a id="other-data-API-xueqiu-top_stocks"></a>xueqiu.top_stocks - 雪球舆论数据查询

```
xueqiu.top_stocks(field, date, freq='day', count=5, country='cn')
```

获取每日、每周或每月的某个指标的雪球数据的股票排名情况以及它的对应的统计数值.
#### <a id="other-data-API-xueqiu-top_stocks-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> field | *str* | 目前支持的雪球数据统计指标有: 昨日新增评论 - `new_comments`，总评论 - `total_comments`，昨日新增关注者 - `new_followers`，总关注者数目 - `total_followers`，卖出行为 - `sell_actions`，买入行为 - `buy_actions`
> date | *str* | 查询日期，格式为 'YYYY-mm-dd'。注意：我们最早支持的雪球数据只到2015年4月23日，之后的数据我们都会保持更新
> freq | *str* | 默认是`day`，即每日的数据统计。也支持`week` - 每周和`month` - 每月的统计
> count | *integer* | 指定返回多少个结果，默认是5个
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="other-data-API-xueqiu-top_stocks-return"></a>返回

*pandas DataFrame* - 各项舆情数据


#### <a id="other-data-API-xueqiu-top_stocks-example"></a>范例

- 获取2015年12月11日昨日新增留言数（new_comments）的前10名的股票列表：

```
[In]
xueqiu.top_stocks('new_comments', '2015-12-11')
[Out]
	order_book_id	new_comments
0	000917.XSHE		344
1	600130.XSHG		242
2	600196.XSHG		150
3	000002.XSHE		139
4	300085.XSHE		127
5	300104.XSHE		122
6	600036.XSHG		120
7	601198.XSHG		110
8	000651.XSHE		106
9	000679.XSHE		100
```

- 获取2015年12月11日的总留言数（total_comments）的前5名的股票列表：

```
[In]
xueqiu.top_stocks('total_comments', '2015-12-11', 'day', count=5)
[Out]
	order_book_id	total_comments
0	002024.XSHE		128682
1	300104.XSHE		117911
2	600036.XSHG		84343
3	600030.XSHG		71478
4	601318.XSHG		64711
```

- 获取2015年12月11日的前一周新增关注数（new_followers） 的前10名的股票列表：

```
[In]
xueqiu.top_stocks('new_followers', '2015-12-11')
[Out]
	order_book_id	new_followers
0	000917.XSHE		4459
1	600130.XSHG		1960
2	300248.XSHE		1677
3	600196.XSHG		1232
4	000673.XSHE		941
...
```

- 获取2015年12月11日的前一个月的新增关注数（new_followers）的前10名的股票列表：

```
[In]
xueqiu.top_stocks('new_followers', '2015-12-11',freq='month')
[Out]
	order_book_id	new_followers
0	000917.XSHE		20417
1	600783.XSHG		12438
2	002489.XSHE		9559
3	600283.XSHG		9537
...
```
---



### <a id="other-data-API-xueqiu-history"></a>xueqiu.history - 雪球股票历史信息查询

```
xueqiu.history(order_book_id, start_date='20150521', end_date='20160521', frequency='1d', fields=None, country='cn')
```
获取雪球股票的历史信息。

#### <a id="other-data-API-xueqiu-history-params"></a>参数

> 参数 | 类型 | 说明
> --  | -- | --
> order_book_id |*str* OR *str list* | 合约代码，可传入order_book_id, symbol
> start_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 开始日期，默认为'20150521'
> end_date | *str*, *datetime.date*, *datetime.datetime*, *pandasTimestamp* | 结束日期，默认为'20160521'
> frequency | *str* | 默认是一天`1d`，目前只支持日级别
> field | *str* | 目前支持的雪球数据统计指标有: 昨日新增评论 - `new_comments`，总评论 - `total_comments`，昨日新增关注者 - `new_followers`，总关注者数目 - `total_followers`，卖出行为 - `sell_actions`，买入行为 - `buy_actions`
> country | *str* | 默认是中国市场('cn')，目前仅支持中国市场

#### <a id="other-data-API-xueqiu-history-return"></a>返回

*pandas DataFrame* 


#### <a id="other-data-API-xueqiu-history-example"></a>范例
- 获取一段时间某只股票的总评论以及昨日新增关注着数目：

```
[In]
xueqiu.history('000001.XSHE', '20160801', '20160820', fields=['total_comments', 'new_followers'])
[Out]
	total_comments	new_followers
date		
2016-08-01	15432	112
2016-08-02	15447	81
2016-08-03	15468	45
2016-08-04	15506	230
2016-08-05	15523	104
2016-08-06	15528	40
2016-08-07	15540	37
2016-08-08	15555	51
...
```

---

