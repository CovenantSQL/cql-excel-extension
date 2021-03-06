## 关于 Quandl

Quandl 是一个针对金融投资行业的大数据平台，其数据来源包括联合国、世界银行、中央银行等公开数据，核心财务数据来自 CLS 集团，Zacks 和 ICE 等，所有的数据源自 500 多家发布商， 是为投资专业人士提供金融，经济和替代数据的首选平台，拥有海量的经济和金融数据。

## Quandl 数据索引

### CovenantSQL 使用简介

首先，我们已基于 Quandl 的开放 API 将数据存放在 CovenantSQL 中，首先应该浏览一下 CovenantSQL 的使用介绍来熟悉一下如何连接并且使用 CovenantSQL。

现在由于客户端兼容问题， 请直接使用我们的 HTTP 服务来对 Quandl 数据库进行 query，未来兼容现在的 `cql` 客户端后会第一时间更新此文档。

具体通过 HTTP 服务来使用 CovenantSQL 请参考 [Python 驱动文档](https://github.com/CovenantSQL/python-driver/blob/master/README.rst) 和 [NodeJS 驱动文档](https://github.com/CovenantSQL/node-covenantsql/blob/master/README.md)

所需参数:

```
host: 'e.morenodes.com'
port: 11108
database: '057e55460f501ad071383c95f691293f2f0a7895988e22593669ceeb52a6452a'
```

### 数据使用方法

Quandl 数据分为表以及子表两层索引

数据库中，表名为`quandl_` + `database_code`为完整表名

通过查询表名来查看表的内容

具体使用时，需要结合`quandl_updateindex` 这张索引表来使用，（查询索引表时，请务必 limit 小于 10 万，因为全表数据量过大）

使用时，请使用 where 语句限定`quandlcode`字段来查询表中的子表数据。

### 查询示例

1. 我们想要查询 欧盟委员会年度宏观经济数据库 的数据，我们找到其第一层索引的 `databasecode` 为`quandl_ameco`，于是，我们可以查询其第二层索引，用以下 SQL 命令行：

   `select * from quandl_updateindex where databasecode like 'ameco' group by quandlcode limit 10000`

2. 然后通过第三列，我们可以查看`quandlcode`对应的描述

   比如 AMECO/ALB_1_0_0_0_AAGE 对应的就是阿尔巴尼亚的进出口相关信息，时间从 1960 到 1988。

   <u>Average share of imports and exports of goods in world trade excluding intra EU trade; Foreign trade statistics (1960-1998 Former EU-15) - Albania</u>

   于是，我们可以用以下方式把这个子表给查询出来

   `select * from quandl_ameco where quandlcode like 'AMECO/ALB_1_0_0_0_AAGE' limit 10000`

3. 注意：如果内容整列为 null 的，属于表结构本身不存在的字段，可以自行去除。

## Quandl 数据 Excel 插件使用说明

您可以下载 Quandl 数据 Excel 插件，无须安装，请解压到任意文件夹中。此插件目前仅支持 Office 2010 以及以上版本ï¼office 2007 以及以下版本目前暂不支持。

### 配置 Excel 插件

解压下载后压缩包，在文件夹中有两个.xll 文件，`ClassLibrary7-AddIn.xll` 以及 `ClassLibrary7-AddIn64.xll` 文件，这两个文件分别对应 32 位的 Excel 与 64 位的 Excel，请根据您的电脑配置使用对应插件。

#### 修改 xml 配置

每个 `.xll` 文件对应一个 `.config` 的配置文件，也就是 `ClassLibrary7-AddIn.xll.config` 和`ClassLibrary7-AddIn64.xll.config`，为如下 xml 配置：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings>
    <add key="certpath" value="C:\Cert\read.data.thunderdb.io.pfx" />
    <add key="keypassword" value="covenantsql" />
    <add key="url" value="https://e.morenodes.com:11108/v1/query" />
    <add key="database" value="057e55460f501ad071383c95f691293f2f0a7895988e22593669ceeb52a6452a" />
  </appSettings>
</configuration>
```

其中有如下配置需要修改，并保存：

- certpath: 请填写 `read.data.thunderdb.io.pfx` 证书的绝对路径

#### 安装插件

有两种办法使用此 Excel 插件

1. 直接双击对应的 32 位或 64 位 Excel 的 xll 文件打开

![quandl_ext_1](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_1.png

如果弹出如上对话框，选择第一个： `仅为本回话启用此加载项`。然后新建一个 Excel 表格，如果在 Excel 的上方选项卡中看到 CovenantSQL，说明插件加载成功。

2. 打开 Excel，然后依次选择 `文件 — 选项 — 加载项`，管理选择 Excel 加载项，点击转到然后浏览选择解压后的对应版本的 `.xll` 文件，然后点击 `确定`，在选项卡中如果成功加载出 CovenantSQL 即为成功加载，如下图：

![quandl_ext_2](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_2.png)

#### 使用插件

选择 CovenantSQL 的选项卡可以看到 Quandl 数据查询，点击 Quandl 数据查询，会弹出一个窗体如下图所示：

![quandl_ext_3](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_3.png)

- 其中红色框住的列表部分是一级目录，下方则是所选项的具体描述

- 绿色按钮是通过选择的一级目录来查询二级目录。这里查询时间会随着所查询二级目录的大小以及用户自身的网络速度等因素而不同，可能查询时间会超过 1 分钟，请耐心等待。

- 黄色部分左边的列表是二级目录，右边的窗体则为列表项所选择的表的描述。

- 蓝色部分是导出数据的可选项
  - Limit Number 是一次导出的最多数据条数，默认为 10000 条，最大值可填写 100000
  - Offset Number 是从数据库的第几条开始导出，因为之前会限定条数，例如我们之前导出了 10000 条数据，但是明显数据还没有导出全部，我们可以使用此功能选择 offset 10000 来从第 10000 条开始导出（注：数据库的计数从 0 开始，所以前 10000 条为 0-9999）

现在，您å³可在 Excel 中方便的调用存在 CovenantSQL 上的 Quandl 数据。

## 附件表

---
id: quandl
title: 基于 Quandl 的金融数据分析
---

## 关于 Quandl

Quandl 是一个针对金融投资行业的大数据平台，其数据来源包括联合国、世界银行、中央银行等公开数据，核心财务数据来自 CLS 集团，Zacks 和 ICE 等，所有的数据源自 500 多家发布商， 是为投资专业人士提供金融，经济和替代数据的首选平台，拥有海量的经济和金融数据。

## Quandl 数据索引

### CovenantSQL 使用简介

首先，我们已基于 Quandl 的开放 API 将数据存放在 CovenantSQL 中，首先应该浏览一下 CovenantSQL 的使用介绍来熟悉一下如何连接并且使用 CovenantSQL。

现在由于客户端兼容问题， 请直接使用我们的 HTTP 服务来对 Quandl 数据库进行 query，未来兼容现在的 `cql` 客户端后会第一时间更新此文档。

具体通过 HTTP 服务来使用 CovenantSQL 请参考 [Python 驱动文档](https://github.com/CovenantSQL/python-driver/blob/master/README.rst) 和 [NodeJS 驱动文档](https://github.com/CovenantSQL/node-covenantsql/blob/master/README.md)

所需参数:

```
host: 'e.morenodes.com'
port: 11108
database: '057e55460f501ad071383c95f691293f2f0a7895988e22593669ceeb52a6452a'
```

### 数据使用方法

Quandl 数据分为表以及子表两层索引

数据库中，表名为`quandl_` + `database_code`为完整表名

通过查询表名来查看表的内容

具体使用时，需要结合`quandl_updateindex` 这张索引表来使用，（查询索引表时，请务必 limit 小于 10 万，因为全表数据量过大）

使用时，请使用 where 语句限定`quandlcode`字段来查询表中的子表数据。

### 查询示例

1. 我们想要查询 欧盟委员会年度宏观经济数据库 的数据，我们找到其第一层索引的 `databasecode` 为`quandl_ameco`，于是，我们可以查询其第二层索引，用以下 SQL 命令行：

   `select * from quandl_updateindex where databasecode like 'ameco' group by quandlcode limit 10000`

2. 然后通过第三列，我们可以查看`quandlcode`对应的描述

   比如 AMECO/ALB_1_0_0_0_AAGE 对应的就是阿尔巴尼亚的进出口相关信息，时间从 1960 到 1988。

   <u>Average share of imports and exports of goods in world trade excluding intra EU trade; Foreign trade statistics (1960-1998 Former EU-15) - Albania</u>

   于是，我们可以用以下方式把这个子表给查询出来

   `select * from quandl_ameco where quandlcode like 'AMECO/ALB_1_0_0_0_AAGE' limit 10000`

3. 注意：如果内容整列为 null 的，属于表结构本身不存在的字段，可以自行去除。

## Quandl 数据 Excel 插件使用说明

您可以下载 Quandl 数据 Excel 插件，无须安装，请解压到任意文件夹中。此插件目前仅支持 Office 2010 以及以上版本，office 2007 以及以下版本目前暂不支持。

### 配置 Excel 插件

解压下载后压缩包，在文件夹中有两个.xll 文件，`ClassLibrary7-AddIn.xll` 以及 `ClassLibrary7-AddIn64.xll` 文件，这两个文件分别对应 32 位的 Excel 与 64 位的 Excel，请根据您的电脑配置使用对应插件。

#### 修改 xml 配置

每个 `.xll` 文件对应一个 `.config` 的配置文件，也就是 `ClassLibrary7-AddIn.xll.config` 和`ClassLibrary7-AddIn64.xll.config`，为如下 xml 配置：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings>
    <add key="certpath" value="C:\Cert\read.data.thunderdb.io.pfx" />
    <add key="keypassword" value="covenantsql" />
    <add key="url" value="https://e.morenodes.com:11108/v1/query" />
    <add key="database" value="057e55460f501ad071383c95f691293f2f0a7895988e22593669ceeb52a6452a" />
  </appSettings>
</configuration>
```

其中有如下配置需要修改，并保存：

- certpath: 请填写 `read.data.thunderdb.io.pfx` 证书的绝对路径

#### 安装插件

有两种办法使用此 Excel 插件

1. 直接双击对应的 32 位或 64 位 Excel 的 xll 文件打开

![quandl_ext_1](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_1.png

如果弹出如上对话框，选择第一个： `仅为本回话启用此加载项`。然后新建一个 Excel 表格，如果在 Excel 的上方选项卡中看到 CovenantSQL，说明插件加载成功。

2. 打开 Excel，然后依次选择 `文件 — 选项 — 加载项`，管理选择 Excel 加载项，点击转到然后浏览选择解压后的对应版本的 `.xll` 文件，然后点击 `确定`，在选项卡中如果成功加载出 CovenantSQL 即为成功加载，如下图：

![quandl_ext_2](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_2.png)

#### 使用插件

选择 CovenantSQL 的选项卡可以看到 Quandl 数据查询，点击 Quandl 数据查询，会弹出一个窗体如下图所示：

![quandl_ext_3](https://github.com/CovenantSQL/cql-excel-extension/blob/master/refs/quandl_ext_3.png)

- 其中红色框住的列表部分是一级目录，下方则是所选项的具体描述

- 绿色按钮是通过选择的一级目录来查询二级目录。这里查询时间会随着所查询二级目录的大小以及用户自身的网络速度等因素而不同，可能查询时间会超过 1 分钟，请耐心等待。

- 黄色部分左边的列表是二级目录，右边的窗体则为列表项所选择的表的描述。

- 蓝色部分是导出数据的可选项
  - Limit Number 是一次导出的最多数据条数，默认为 10000 条，最大值可填写 100000
  - Offset Number 是从数据库的第几条开始导出，因为之前会限定条数，例如我们之前导出了 10000 条数据，但是明显数据还没有导出全部，我们可以使用此功能选择 offset 10000 来从第 10000 条开始导出（注：数据库的计数从 0 开始，所以前 10000 条为 0-9999）

现在，您即可在 Excel 中方便的调用存在 CovenantSQL 上的 Quandl 数据。

## 附件表

| database_code | description                                                                                                                                                                                                                                                                                                                                                          |                                             |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| BUNDESBANK    | Data on the German economy, money and capital markets, public finances, banking, households, Euro-area aggregates, trade and external debt.                                                                                                                                                                                                                          | 德意志联邦银行数据仓库                      |
| URC           | Advance and decline data for the NYSE, AMEX, and NASDAQ stock exchanges. From various publicly-available sources and the median value is reported.                                                                                                                                                                                                                   | 独角兽研究公司数据                          |
| DCE           | Agriculture and commodities futures from DCE, with history spanning almost a decade for select futures.                                                                                                                                                                                                                                                              | 大连商品交易所数据                          |
| WFC           | This database offers mortgage purchase and refinance rates from Wells Fargo Home Mortgage, a division of Wells Fargo Bank.                                                                                                                                                                                                                                           | 富国银行住房抵押贷款数据                    |
| USDAFNS       | Food and Nutrition Service administrates federal nutrition assistance programs for low-income households and children. Data on costs and participation rates.                                                                                                                                                                                                        | 美国农业部食品与营养服务项目数据            |
| LJUBSE        | This database contains the Ljubljana Stock Exchange indexes and is based in Ljubljana, Slovenia.                                                                                                                                                                                                                                                                     | 卢布尔雅那证券交易所数据                    |
| TOCOM         | Agriculture and commodities futures from Tokyo Commodities Exchange (TOCOM), with history spanning almost a decade for select futures.                                                                                                                                                                                                                               | 东京商品交易所数据                          |
| LBMA          | An international trade association in the London gold and silver market, consisting of central banks, private investors, producers, refiners, and other agents.                                                                                                                                                                                                      | 伦敦金银市场协会数据                        |
| WCSC          | This database is designed to provide a strategic overview of the World Bank Group’s performance toward ending extreme poverty and promoting shared prosperity.                                                                                                                                                                                                       | 世界银行企业积分卡数据                      |
| CMHC          | The CMHC is a gov’t-owned corporation that provides affordable housing to Canadian citizens and collects data such as prices, construction, and supply.                                                                                                                                                                                                              | 加拿大住房抵押贷款公司                      |
| WGEC          | Data containing commodity prices and indices from 1960 to present.                                                                                                                                                                                                                                                                                                   | 世界银行全球经济监测商品数据（GEM）         |
| FED           | Official US figures on money supply, interest rates, mortgages, government finances, bank assets and debt, exchange rates, industrial production.                                                                                                                                                                                                                    | 美联储数据公布                              |
| WPSD          | Data jointly developed by the World Bank and the International Monetary Fund, which brings together detailed public sector government debt data.                                                                                                                                                                                                                     | 世界银行公共部分支出数据                    |
| UGID          | This database offers a wide range of global indicators, covering population, public health, employment, trade, education, inflation and external debt.                                                                                                                                                                                                               | 联合国全球指标                              |
| RBA           | Central bank and monetary authority, regulates banking industry, sets interest rates, and services government's debt. Data on key economic indicators.                                                                                                                                                                                                               | 澳大利亚储备银行数据                        |
| UCOM          | This database offers comprehensive global data on imports and exports of commodities such as food, live animals, pharmaceuticals, metals, fuels and machinery.                                                                                                                                                                                                       | 联合国商品贸易数据                          |
| SIDC          | The SIDC hosts data spanning from the 1700's on solar activity, specifically sunspot activity.                                                                                                                                                                                                                                                                       | 太阳影响数据分析中心数据                    |
| ZCE           | Agriculture and commodities futures from ZCE, with history spanning almost a decade for select futures.                                                                                                                                                                                                                                                              | 郑州商品交易所数据                          |
| ZILLOW2       | Home prices and rents by size, type and tier; housing supply, demand and sales; sliced by zip code, neighbourhood, city, metro area, county and state.                                                                                                                                                                                                               | Zillow 房地产研究                           |
| USTREASURY    | The U.S. Treasury ensures the nation's financial security, manages the nation's debt, collects tax revenues, and issues currency, provides data on yield rates.                                                                                                                                                                                                      | 美国财政部数据                              |
| BTS           | BTS provides information on transportation systems in the United States. Their non-multimodal department provides statistics on air travel.                                                                                                                                                                                                                          | 美国交通运输统计局数据                      |
| USDAFAS       | The USDA Foreign Agricultural Service connects U.S. agriculture with the world markets. It provides statistics on production and exports in foreign countries.                                                                                                                                                                                                       | 美国农业部外国农业服务数据（FAS）           |
| OECD          | International organization of developed countries that promotes economic welfare. Collects data from members and others to make policy recommendations.                                                                                                                                                                                                              | 世界经济合作与发展组织数据                  |
| OPEC          | International organization and economic cartel overseeing policies of oil-producers, such as Iraq, Iran, Saudi Arabia, and Venezuela. Data on oil prices.                                                                                                                                                                                                            | 欧佩克数据                                  |
| MCX           | India's largest commodity exchange servicing futures trading in metals, energy, and agriculture. World's 3rd largest exchange in contracts and trading volume.                                                                                                                                                                                                       | 印度多种商品交易所数据                      |
| ECONOMIST     | The Big Mac index was invented by The Economist in 1986 as a lighthearted guide to whether currencies are at their “correct” level. It is based on the theory of purchasing-power parity (PPP).                                                                                                                                                                      | 经济学人 - 巨无霸指数                       |
| NSDL          | Depository in India responsible for economic development of the country that has established a national infrastructure of international standards that handles most of the securities held and settled in dematerialised form in the Indian capital market.                                                                                                          | 国家证券存管有限公司（印度）数据            |
| FRED          | Growth, employment, inflation, labor, manufacturing and other US economic statistics from the research department of the Federal Reserve Bank of St. Louis.                                                                                                                                                                                                          | 美联储经济数据                              |
| GDT           |                                                                                                                                                                                                                                                                                                                                                                      | 全球乳品贸易数据                            |
| CFFEX         | Index and bond futures from CFFEX, with history spanning almost a decade for select futures.                                                                                                                                                                                                                                                                         | 中国金融期货交易所数据                      |
| CITYPOP       | Thomas Brinkhoff provides population data for cities and administrative areas in most countries.                                                                                                                                                                                                                                                                     | Thomas Brinkhoff 的城市人口数据             |
| BCHARTS       | Exchange rates for bitcoin against a large number of currencies, from all major bitcoin exchanges, including current and historical exchange rates.                                                                                                                                                                                                                  | 比特币图表汇率数据                          |
| LOCALBTC      |                                                                                                                                                                                                                                                                                                                                                                      | Local Bitcoins 数据                         |
| JODI          | JODI oil and gas data comes from over 100 countries consisting of multiple energy products and flows in various methods of measurement.                                                                                                                                                                                                                              | JODI 石油世界数据库                         |
| UENG          | This database offers comprehensive global statistics on production, trade, conversion, and final consumption of new and renewable energy sources.                                                                                                                                                                                                                    | 联合国能源统计数据                          |
| ULMI          | This database offers comprehensive youth unemployment figures broken up by gender for all countries in the world.                                                                                                                                                                                                                                                    | 联合国劳工市场指标                          |
| MAURITIUSSE   | Stock Exchange of Mauritius indices data.                                                                                                                                                                                                                                                                                                                            | 毛里求斯证券交易所数据                      |
| UKRSE         | UKRSE presents the most current available data related to the largest stock exchanges in Ukraine. The exchange is located in Kiev and accounts for roughly three-quarters of Ukraine's total equity trading volume                                                                                                                                                   | 乌克兰交易所数据                            |
| BITSTAMP      | Bitstamp is a trading platform for Bitcoin.                                                                                                                                                                                                                                                                                                                          | Bitstamp 数据                               |
| UNAE          | This database offers global data on gross domestic product, gross national income, and gross value added by different sectors for all countries in the world.                                                                                                                                                                                                        | 联合国国民账户估算数据                      |
| UNAC          | This database offers global data on national accounts, such as assets and liabilities of households, corporations and governments.                                                                                                                                                                                                                                   | 联合国国民账户官方国家数据                  |
| UTOR          | This database offers comprehensive data on international tourism. Data includes number of tourist arrivals and tourism expenditures for all countries.                                                                                                                                                                                                               | 联合国世界旅游业数据                        |
| WFE           | A trade association of sixty publicly regulated stock, futures, and options exchanges that publishes data for its exchanges, like market capitalization.                                                                                                                                                                                                             | 世界交易所联合会数据                        |
| FRBC          | The Federal Reserve Bank of Cleveland collects data from hundreds of financial institutions, including depository institutions, bank holding companies, and other entities that is used to assess financial institution conditions and also to glean insights into how the economy and financial system are doing.                                                   | 克利夫兰联邦储备银行数据                    |
| UGEN          | This database offers comprehensive global data on a wide range of gender-related indicators, covering demography, health, education and employment.                                                                                                                                                                                                                  | 联合国性别信息                              |
| BITFINEX      | Bitfinex is a trading platform for Bitcoin, Litecoin and Darkcoin with many advanced features including margin trading, exchange and peer to peer margin funding.                                                                                                                                                                                                    | Bitfinex 数据                               |
| UGHG          | This database offers comprehensive global data on anthropogenic emissions of the six principal greenhouse gases. Data goes back to 1990.                                                                                                                                                                                                                             | 联合国温室气体清单                          |
| UIST          | This database offers global data on industrial development indicators, including output, employees, wages, value added for a wide range of industries.                                                                                                                                                                                                               | 联合国工业发展组织数据                      |
| PRAGUESE      | Price index data from the Prague Stock Exchange.                                                                                                                                                                                                                                                                                                                     | 布拉格证券交易所数据                        |
| PFTS          | Index data from the PFTS Stock Exchange, the largest marketplace in Ukraine.                                                                                                                                                                                                                                                                                         | PFTS 证券交易所（乌克兰）数据               |
| WARSAWSE      | WIG20 index has been calculated since April 16, 1994 based on the value of portfolio with shares in 20 major and most liquid companies in the WSE Main List.                                                                                                                                                                                                         | 华沙证券交易所数据                          |
| TUNISSE       | The main reference index of Tunis Stock Exchange.                                                                                                                                                                                                                                                                                                                    | 突尼斯证券交易所数据                        |
| FRKC          | FRKC is the regional central bank for the 10th District of the Federal Reserve, publishing data on banking in mostly agricultural transactions.                                                                                                                                                                                                                      | 堪萨斯城联邦储备银行数据                    |
| UENV          | This database offers global data on water and waste related indicators, including fresh water supply and precipitation, and generation and collection of waste.                                                                                                                                                                                                      | 联合国环境统计数据                          |
| NSE           | Stock and index data from the National Stock Exchange of India.                                                                                                                                                                                                                                                                                                      | 印度国家证券交易所数据                      |
| UFAO          | This database offers global food and agricultural data, covering crop production, fertilizer consumption, use of land for agriculture, and livestock.                                                                                                                                                                                                                | 联合国粮食和农业数据                        |
| TAIFEX        | Index and bond futures from TAIFEX, with history spanning over a decade for select futures.                                                                                                                                                                                                                                                                          | 台湾期货交易所数据                          |
| GDAX          | GDAX is the world’s most popular place to buy and sell bitcoin.                                                                                                                                                                                                                                                                                                      | GDAX（全球数字资产交易所）数据              |
| ARES          | ARES protects investors, contributes to development of the real estate securitization product market, and facilitates expansion of the real estate investment market.                                                                                                                                                                                                | 房地产证券化协会数据                        |
| SHADOWS       | This dataset contains the three major indicators from the Wu-Xia papers which serve to identify the shadow rates on all three major banks.                                                                                                                                                                                                                           | 影子联邦基金利率模型数据                    |
| NAAIM         | The NAAIM Exposure Index represents the average exposure to US Equity markets reported by NAAIM members.                                                                                                                                                                                                                                                             | 全国积极投资管理者协会头寸指数              |
| CBRT          | CBRT is responsible for taking measures to sustain the stability of the financial system in Turkey.                                                                                                                                                                                                                                                                  | 土耳其共和国中央银行数据                    |
| CEGH          | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 中欧天然气中心数据                          |
| FINRA         | Financial Industry Regulatory Authority provides short interest data on securities firms and exchange markets.                                                                                                                                                                                                                                                       | 美国金融业监管局数据                        |
| NASDAQOMX     | Over 35,000 global indexes published by NASDAQ OMX including Global Equity, Fixed Income, Dividend, Green, Nordic, Sharia and more. Daily data.                                                                                                                                                                                                                      | 纳斯达克 OMX 全球指数数据                   |
| EURONEXT      | Historical stock data from Euronext, the largest European exchange.                                                                                                                                                                                                                                                                                                  | 泛欧证券交易所数据                          |
| UICT          | This database offers comprehensive global data on information and communication technology, including telephone, cellular and internet usage for all countries.                                                                                                                                                                                                      | 联合国信息和通信技术数据                    |
| USAID         | US Agency for International Development provides a complete historical record of all foreign assistance provided by the United States to the rest of the world.                                                                                                                                                                                                      | 美国国际开发署数据                          |
| ZAGREBSE      | Croatia's only stock exchange. It publishes data on the performance of its stock and bond indexes.                                                                                                                                                                                                                                                                   | 萨格勒布证券交易所数据                      |
| QUITOSE       | The indexes of the national Stock Exchange of Ecuador.                                                                                                                                                                                                                                                                                                               | 基多证券交易所（厄瓜多尔）数据              |
| ECBCS         | Data in this database is derived from harmonized surveys for different sectors of the economies in the European Union (EU) and in the EU applicant countries.                                                                                                                                                                                                        | 欧盟委员会商业和消费者调查                  |
| PSE           | This database describes the distribution of top incomes in a growing number of countries. Numbers are derived using tax data.                                                                                                                                                                                                                                        | 巴黎经济学院数据                            |
| MALTASE       | The Malta Stock Exchange carries out the role of providing a structure for admission of financial instruments to its recognised lists which may subsequently be traded on a regulated, transparent and orderly market place (secondary market).The main participants in the market are Issuers, Stock Exchange Members (stockbrokers), and the investors in general. | 马耳他证券交易所数据                        |
| GPP           | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 全球石油价格                                |
| PPE           | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 波兰电力交易所（TGE）数据                   |
| UKONS         | Data on employment, investment, housing, household expenditure, national accounts, and many other socioeconomic indicators in the United Kingdom.                                                                                                                                                                                                                    | 英国国家统计局数据                          |
| NCDEX         | A professionally managed on-line multi-commodity exchange in India                                                                                                                                                                                                                                                                                                   | 国家商品及衍生品交易所（印度）数据          |
| WSE           | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 华沙证券交易所（GPW）数据                   |
| TFX           | The Tokyo Financial Exchange is a futures exchange that primary deals in financial instruments markets that handle securities and market derivatives.                                                                                                                                                                                                                | 东京金融交易所数据                          |
| WGFD          | Data on financial system characteristics, including measures of size, use, access to, efficiency, and stability of financial institutions and markets.                                                                                                                                                                                                               | 世界银行全球金融发展数据                    |
| CEPEA         | CEPEA is an economic research center at the University of Sao Paulo focusing on agribusiness issues, publishing price indices for commodities in Brazil.                                                                                                                                                                                                             | 应用经济学应用研究中心（巴西）数据          |
| SBJ           | A Japanese government statistical agency that provides statistics related to employment and the labour force.                                                                                                                                                                                                                                                        | 日本统计局数据                              |
| WGEM          | Data on global economic developments, with coverage of high-income, as well as developing countries.                                                                                                                                                                                                                                                                 | 世界银行全球经济监测                        |
| WGDF          | Data on financial system characteristics, including measures of size, use, access to, efficiency, and stability of financial institutions and markets.                                                                                                                                                                                                               | 世界银行全球发展金融                        |
| WWDI          | Most current and accurate development indicators, compiled from officially-recognized international sources.                                                                                                                                                                                                                                                         | 世界银行世界发展指标                        |
| WESV          | Company-level private sector data, covering business topics including finance, corruption, infrastructure, crime, competition, and performance measures.                                                                                                                                                                                                             | 世界银行企业调查                            |
| CHRIS         | Continuous contracts for all 600 futures on Quandl. Built on top of raw data from CME, ICE, LIFFE etc. Curated by the Quandl community. 50 years history.                                                                                                                                                                                                            | 维基连续期货                                |
| WDBU          | Data on business regulations and their enforcement for member countries and selected cities at the subnational and regional level.                                                                                                                                                                                                                                   | 世界银行存续商业数据                        |
| OSE           | The second largest securities exchange in Japan. Unlike the TSE, OSE is strongest in derivatives trading, the majority of futures and options in Japan.                                                                                                                                                                                                              | 大阪证券交易所数据                          |
| RFSS          | The Russian governmental statistical agency that publishes social, economic, and demographic statistics for Russia at the national and local levels.                                                                                                                                                                                                                 | 俄罗斯联邦统计局数据                        |
| EUREX         | Index, rate, agriculture and energy futures from EUREX, Europe's largest futures exchange, with history spanning a decade for select futures.                                                                                                                                                                                                                        | 欧洲期货交易所数据                          |
| WMDG          | Data drawn from the World Development Indicators, reorganized according to the goals and targets of the Millennium Development Goals (MDGs).                                                                                                                                                                                                                         | 世界银行千年发展目标数据                    |
| ZILLOW        | Home prices and rents by size, type and tier; housing supply, demand and sales; sliced by zip code, neighbourhood, city, metro area, county and state.                                                                                                                                                                                                               | Zillow 房地产研究                           |
| WPOV          | Indicators on poverty headcount ratio, poverty gap, and number of poor at both international and national poverty lines.                                                                                                                                                                                                                                             | 世界银行贫困统计                            |
| EUREKA        | A research company focused on hedge funds and other alternative investment funds. It publishes data on the performances of hedge funds.                                                                                                                                                                                                                              | Eurekahedge 数据                            |
| MOFJ          | Japanese government bond interest rate data, published daily by the Ministry of Finance.                                                                                                                                                                                                                                                                             | 日本财务省数据                              |
| PIKETTY       | Data on Income and Wealth from "Capital in the 21st Century", Harvard University Press 2014.                                                                                                                                                                                                                                                                         | Thomas Piketty 数据                         |
| PSX           | Daily closing stock prices from the Pakistan Stock Exchange.                                                                                                                                                                                                                                                                                                         | 巴基斯坦证交所数据                          |
| SGX           | Asian securities and derivatives exchange that trades in equities for many large Singaporean and other Asian companies. Listed on its own exchange.                                                                                                                                                                                                                  | 新加坡交易所数据                            |
| UIFS          | This database offers comprehensive data on international financial indicators, such as average earnings, bond yields, government revenues and expenditures.                                                                                                                                                                                                          | 联合国国际金融统计                          |
| UINC          | This database offers global data on production of industrial commodities, such as ores and minerals, food products, transportable goods, and metal products.                                                                                                                                                                                                         | 联合国工业商品数据                          |
| INSEE         | INSEE is the national statistical agency of France. It collects data on France's economy and society, such as socioeconomic indicators and national accounts.                                                                                                                                                                                                        | 国家统计和经济研究所（法国）数据            |
| SNB           | Central bank responsible for monetary policy and currency. Data on international accounts, interest rates, money supply, and other macroeconomic indicators.                                                                                                                                                                                                         | 瑞士国家银行数据                            |
| ODE           | A non-profit commodity exchange in the Kansai region of Japan that trades in seven key agricultural commodities.                                                                                                                                                                                                                                                     | 大阪道岛商品交易所数据                      |
| WGEN          | Data describing gender differences in earnings, types of jobs, sectors of work, farmer productivity, and entrepreneurs’ firm sizes and profits.                                                                                                                                                                                                                      | 世界银行性别统计                            |
| WHNP          | Key health, nutrition and population statistics.                                                                                                                                                                                                                                                                                                                     | 世界银行健康营养与人口统计                  |
| WIDA          | Data on progress on aggregate outcomes for IDA (International Development Association) countries for selected indicators.                                                                                                                                                                                                                                            | 世界银行国际发展协会                        |
| ECMCI         | Updated monthly, this database provides monetary conditions index values in the Euro zone. History goes back to 1999.                                                                                                                                                                                                                                                | 欧盟委员会货币状况指数                      |
| NBSC          | Statistics of China relating to finance, industry, trade, agriculture, real estate, and transportation.                                                                                                                                                                                                                                                              | 中国国家统计局数据                          |
| MAS           | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 新加坡金融管理局数据                        |
| MGEX          | A marketplace of futures and options contracts for regional commodities that facilitates trade of agricultural indexes.                                                                                                                                                                                                                                              | 明尼阿波利斯谷物交易所数据                  |
| WWGI          | Data on aggregate and individual governance indicators for six dimensions of governance.                                                                                                                                                                                                                                                                             | 世界银行全球治理指标                        |
| ISM           | ISM promotes supply-chain management practices and publishes data on production and supply chains, new orders, inventories, and capital expenditures.                                                                                                                                                                                                                | 供应管理研究所                              |
| PERTH         | The Perth Mint’s highs, lows and averages of interest rates and commodities prices, updated on a monthly basis.                                                                                                                                                                                                                                                      | 珀斯铸币厂数据                              |
| UKR           | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | 乌克兰交易所数据                            |
| FRBNY         | The FRBNY is the largest regional central bank in the US. Sets monetary policy for New York, most of Connecticut and New Jersey, and some territories.                                                                                                                                                                                                               | 纽约联邦储备银行数据                        |
| FRBP          | The FRBP is a regional central bank for the Federal Reserve. It publishes data on business confidence indexes, GDP, consumption, and other economic indicators.                                                                                                                                                                                                      | 费城联邦储备银行数据                        |
| FMSTREAS      | The monthly receipts/outlays and deficit/surplus of the United States of America.                                                                                                                                                                                                                                                                                    | 美国财政部 - 财务管理处数据                 |
| EIA           | US national and state data on production, consumption and other indicators on all major energy products, such as electricity, coal, natural gas and petroleum.                                                                                                                                                                                                       | 美国能源信息管理局数据                      |
| SOCSEC        | Provides data on the US social security program, particularly demographics of beneficiaries; disabled, elderly, and survivors.                                                                                                                                                                                                                                       | 美国社会保障局数据                          |
| TFGRAIN       | Cash price of corn and soybeans including basis to front month futures contract.                                                                                                                                                                                                                                                                                     | 顶级飞行谷物合作社数据                      |
| IRPR          | Puerto Rico Institute of Statistics statistics on manufacturing.                                                                                                                                                                                                                                                                                                     | 波多黎各统计研究所数据                      |
| BCHAIN        | Blockchain is a website that publishes data related to Bitcoin, updated daily.                                                                                                                                                                                                                                                                                       | blockchain.com 数据                         |
| BITCOINWATCH  | Bitcoin mining statistics.                                                                                                                                                                                                                                                                                                                                           | Bitcoin Watch 数据                          |
| ODA           | IMF primary commodity prices and world economic outlook data, published by Open Data for Africa. Excellent cross-country macroeconomic data.                                                                                                                                                                                                                         | 国际货币基金组织跨国宏观经济统计            |
| WADI          | A collection of development indicators on Africa, including national, regional and global estimates.                                                                                                                                                                                                                                                                 | 世界银行非洲发展指标                        |
| WEDU          | Internationally comparable indicators on education access, progression, completion, literacy, teachers, population, and expenditures.                                                                                                                                                                                                                                | 世界银行教育统计                            |
| WGLF          | Indicators of financial inclusion measures on how people save, borrow, make payments and manage risk.                                                                                                                                                                                                                                                                | 世界银行全球 Findex（全球金融包容性数据库） |
| WWID          | The World Wealth and Income Database aims to provide open and convenient access to the most extensive available database on the historical evolution of the world distribution of income and wealth, both within countries and between countries.                                                                                                                    | 世界财富和收入数据库                        |
| BTER          | Historical exchange rate data for crypto currencies.                                                                                                                                                                                                                                                                                                                 | 比特儿数据                                  |
| BP            | BP is a large energy producer and distributor. It provides data on energy production and consumption in individual countries and larger subregions.                                                                                                                                                                                                                  | BP 能源生产和消费数据                       |
| AMFI          | This database represents fund information from The Association of Mutual Funds in India.                                                                                                                                                                                                                                                                             | 印度共同基金协会数据                        |
| BOC           | Economic, financial and banking data for Canada. Includes interest rates, inflation, national accounts and more. Daily updates.                                                                                                                                                                                                                                      | 加拿大银行统计数据库                        |
| BLSE          | US national and state-level employment and unemployment statistics, published by the Bureau of Labor Statistics.                                                                                                                                                                                                                                                     | 美国劳工统计局就业与失业统计                |
| BLSI          | US national and state-level inflation data, published by the Bureau of Labor Statistics.                                                                                                                                                                                                                                                                             | 美国劳工统计局通货膨胀和价格统计            |
| BITAL         | This database provides data on stock future contracts from the Borsa Italiana, now part of London Stock Exchange Group.                                                                                                                                                                                                                                              | 意大利证交所数据                            |
| BLSB          | US work stoppage statistics, published by the Bureau of Labor Statistics.                                                                                                                                                                                                                                                                                            | 美国劳工统计局薪酬福利统计                  |
| ECB           | The central bank for the European Union oversees monetary policy and the Euro, and provides data on related macroeconomic variables.                                                                                                                                                                                                                                 | 欧洲中央银行数据                            |
| BOJ           |                                                                                                                                                                                                                                                                                                                                                                      | 日本银行数据                                |
| BOF           | The Banque de France is responsible for monetary policy through policies of the European System of Central Banks and providing data on key economic indictors.                                                                                                                                                                                                       | 法国银行数据                                |
| LPPM          | Price data on the market-clearing level for Palladium and Platinum around the world.                                                                                                                                                                                                                                                                                 | 白金和钯价格数据                            |
| JOHNMATT      | Current and historical data on platinum group metals such as prices, supply, and demand.                                                                                                                                                                                                                                                                             | 稀有金属价格数据                            |
| NAHB          | Housing and economic indices for the United States.                                                                                                                                                                                                                                                                                                                  | 美国住房指数                                |
| RATEINF       | Inflation Rates and the Consumer Price Index CPI for Argentina, Australia, Canada, Germany, Euro area, France, Italy, Japan, New Zealand and more.                                                                                                                                                                                                                   | 通货膨胀率                                  |
| RENCAP        | Data on the IPO market in the United States.                                                                                                                                                                                                                                                                                                                         | IPO 数据                                    |
| ML            | Merrill Lynch, a major U.S. bank, publishes data on yield rates for corporate bonds in different regions.                                                                                                                                                                                                                                                            | 公司债券收益率数据                          |
| MULTPL        | No description for this database yet.                                                                                                                                                                                                                                                                                                                                | S&P 500                                     |
| RICI          | Composite, USD based, total return index, representing the value of a basket of commodities consumed in the global economy.                                                                                                                                                                                                                                          | 商品指数                                    |
| AAII          | American Association of Individual Investor’s sentiment data.                                                                                                                                                                                                                                                                                                        | 投资者情绪                                  |
| WORLDAL       | World Aluminium capacity and production in thousand metric tonnes.                                                                                                                                                                                                                                                                                                   | 铝价格                                      |
| WGC           | The World Gold Council is a market development organization for the gold industry. It publishes data on gold prices in different currencies.                                                                                                                                                                                                                         | 黄金价格                                    |
| MX            | Montreal Exchange is a derivatives exchange that trades in futures contracts and options for equities, indices, currencies, ETFs, energy, and interest rates.                                                                                                                                                                                                        | 加拿大期货数据                              |
| UMICH         | The University of Michigan’s consumer survey - data points for the most recent 6 months are unofficial; they are sourced from articles in the Wall Street Journal.                                                                                                                                                                                                   | 消费者情绪                                  |
| BUCHARESTSE   | The Bucharest Stock Exchange publishes data on it activity in equity, rights, bonds, fund units, structured products, and futures contracts.                                                                                                                                                                                                                         | 布加勒斯特证券交易所数据                    |
| BSE           | End of day prices, indices, and additional information for companies trading on the Bombay Stock Exchange in India.                                                                                                                                                                                                                                                  | 孟买证券交易所数据                          |
| CFTC          | Weekly Commitment of Traders and Concentration Ratios. Reports for futures positions, as well as futures plus options positions. New and legacy formats.                                                                                                                                                                                                             | 商品期货交易委员会报告                      |
| ACC           | Chemical Activity Barometer (CAB) published by the American Chemistry Council (ACC). The CAB is an economic indicator that helps anticipate peaks and troughs in the overall US economy and highlights potential trends in other industries in the US.                                                                                                               | 美国化学理事会数据                          |
| AMECO         | Annual macro-economic database of the European Commission's Directorate General for Economic and Financial Affairs (DG ECFIN).                                                                                                                                                                                                                                       | 欧盟委员会年度宏观经济数据库                |
| MORTGAGEX     | Historical housing data on ARM indexes.                                                                                                                                                                                                                                                                                                                              | 可调利率抵押贷款指数                        |
| BUCHARESTSE   | The Bucharest Stock Exchange publishes data on it activity in equity, rights, bonds, fund units, structured products, and futures contracts.                                                                                                                                                                                                                         | 布加勒斯特证券交易所数据                    |
| AMECO         | Annual macro-economic database of the European Commission's Directorate General for Economic and Financial Affairs (DG ECFIN).                                                                                                                                                                                                                                       | 欧盟委员会年度宏观经济数据库                |
| ACC           | Chemical Activity Barometer (CAB) published by the American Chemistry Council (ACC). The CAB is an economic indicator that helps anticipate peaks and troughs in the overall US economy and highlights potential trends in other industries in the US.                                                                                                               | 美国化学理事会数据                          |
| ABMI          | Indicators of Chinese and Japanese bond markets, such as size and composition, market liquidity, yield returns, and volatility.                                                                                                                                                                                                                                      | 亚洲债券市场计划                            |
| BITAL         | This database provides data on stock future contracts from the Borsa Italiana, now part of London Stock Exchange Group.                                                                                                                                                                                                                                              |                                             |
| BANKRUSSIA    | Primary indicators from the Bank of Russia, including data on the banking sector, money supply, financial markets and macroeconomic statistical data.                                                                                                                                                                                                                | 俄罗斯银行数据                              |
| FMAC          | Data from Freddie Mac’s Primary Mortgage Market Survey and other region specific historical mortgage rates.                                                                                                                                                                                                                                                          | 房地美数据                                  |
| BOC           | Economic, financial and banking data for Canada. Includes interest rates, inflation, national accounts and more. Daily updates.                                                                                                                                                                                                                                      | 加拿大银行统计数据库                        |
| BSE           | End of day prices, indices, and additional information for companies trading on the Bombay Stock Exchange in India.                                                                                                                                                                                                                                                  | 孟买证券交易所数据                          |
