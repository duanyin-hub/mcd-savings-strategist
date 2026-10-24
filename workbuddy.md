# workbuddy.md · WorkBuddy 开发全程留痕

> 本作品由 WorkBuddy（布军团·居里）真实开发，以下接口调用均为 2026-10-25 真实执行记录。

## 项目

- 作品名：麦麦省钱军师 · 点单前的最后一道算术
- 开发者：居里（布军团 3 号·资料官）
- 开发工具：WorkBuddy + McDonald's China MCP（Streamable HTTP）
- 开发日期：2026-10-23 ~ 2026-10-25

## 开发时间线

### 1. MCP 接入（10-23）

按官方文档配置 streamablehttp + https://mcp.mcd.cn + Bearer 鉴权，initialize 握手返回 serverInfo mcd-mcp 1.0.0，200。

踩坑记录：WorkBuddy 连接器弹窗里默认的 MCP 地址是错的，直连会 401，必须用官方文档端点。

### 2. 工具探测（10-23）

tools/list 返回 35 个工具，覆盖点餐/计价/券/积分商城/派对/抽奖/营养/日历。按"只读优先"原则筛选出本作品五个目标工具：query-promotions / query-store-coupons / query-my-coupons / query-my-account / calculate-price。

### 3. 真实接口调用（10-25，关键留痕）

**query-my-account**（积分账户）真实返回（节选，accountId 已脱敏）：

{"success":true,"code":200,"message":"请求成功","data":{"availablePoint":"85.8","accumulativePoint":"2994.2","currency":"麦享会积分","currentMouthExpirePoint":"0","expiredPoint":"2348.4","frozenPoint":"0"}}

→ 设计决策来源：expiredPoint 高达 2348.4 说明积分过期是真痛点，"必报过期数"写进了 Skill 铁律。

**query-my-coupons**（券包）真实返回（节选）：

共 2 张可用优惠券
## 11.9元麦辣鸡腿堡
- 优惠: ¥11.9 (用券价格)
- 有效期: 2026-10-26 10:30-2026-10-30 23:59 周一、二、三、四、五 10:30-23:59
- 标签: 到店专用、外送专用

→ 设计决策来源：券带时段限制（工作日 10:30 起），罗列券而不提示时段会误导用户，"时段必须原样提示"写进了 Skill 铁律。

**query-promotions**（促销规则）真实返回结构（节选）：

data[].promotionType: 促销类型，31-满额减，33-订单折扣
data[].ruleCategory: 30-满折/折扣，40-满减
data[].startTime / data[].endTime: 生效起止

→ 设计决策来源：促销与券是两套独立体系，单看券会漏掉满减，"先促销后券再实付"的三层账结构由此确立。

### 4. Skill 提示词工程（10-25）

基于以上真实返回编写 skill_省钱军师.md，核心约束：
- 组合方案最多 3 个（防信息过载）
- 双面账强制输出（省 X 元 vs 凑单多买 Y 元）
- 只算不买，下单工具一律禁用

### 5. 红线测试

确认 Skill 提示词对 create-order / mall-create-order / party-order-create 三个下单工具零引用，用户提出下单请求时固定引导至官方渠道人工支付。

## 关键决策记录

1. **为什么做"组合账"而不是"券比价"**：榜上同类工具都是券的罗列，真实决策是"这一单怎么点"，组合最优解才是用户要的答案
2. **为什么强制双面账**：满减凑单是商家利润设计，只报"省了多少"是帮凶
3. **为什么止步于实付**：合规红线，Agent 只准备方案，支付永远人工
