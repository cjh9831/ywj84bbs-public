# 澳门每期主数据标准

**schema_name:** macau_draw_master_schema
**schema_version:** 1.0
**market:** macau
**status:** active

---

## 1. 设计目的

本 Schema 用于统一保存澳门每一期的：

* 原始资料
* 起卦卦象
* 八肖资料
* 号码与生肖映射
* 候选号码
* 28/24/20/16/12/8 六级分析资料
* 实际开奖结果
* 数据验证及版本信息

核心原则：

> 原始资料与分析结果严格分离，所有派生结果必须能够追溯来源。

---

## 2. 数据身份

| 字段             | 类型      | 必填 | 说明                       |
| -------------- | ------- | -: | ------------------------ |
| draw_id        | string  |  是 | 每期唯一 ID，例如 `MC-2026-235` |
| year           | integer |  是 | 年份                       |
| period         | integer |  是 | 期号                       |
| draw_date      | string  |  否 | 开奖日期                     |
| market         | string  |  是 | 固定为 `macau`              |
| source         | string  |  是 | 数据来源                     |
| source_version | string  |  否 | 原始资料版本                   |

### draw_id 规则

澳门：

`MC-年份-期号`

例如：

`MC-2026-235`

---

## 3. RAW 原始资料

原始资料必须完整保存。

| 字段              | 类型     | 必填 | 说明     |
| --------------- | ------ | -: | ------ |
| raw_data        | object |  是 | 本期原始资料 |
| raw_text        | string |  否 | 原始文字   |
| raw_source_file | string |  否 | 原始文件名称 |
| raw_import_time | string |  是 | 导入时间   |

### RAW 原则

1. 原始资料不得覆盖。
2. 原始资料不得因为分析结果而修改。
3. 发现错误时建立修正版，不删除旧版本。
4. 必须能够追溯原始来源文件。

---

## 4. 澳门卦象资料

澳门起卦体系**不在 Schema 阶段预先限定**。

实际数据库出现哪一种起卦资料，就建立对应记录。

| 字段               | 类型     | 必填 | 说明         |
| ---------------- | ------ | -: | ---------- |
| hexagrams        | array  |  否 | 本期全部澳门起卦卦象 |
| hexagram_sources | array  |  否 | 各卦象来源      |
| hexagram_version | string |  否 | 卦象资料版本     |

### 单个卦象记录建议结构

| 字段                   | 类型      | 说明        |
| -------------------- | ------- | --------- |
| source               | string  | 起卦来源      |
| stroke               | integer | 笔画，如适用    |
| divination_value     | string  | 起卦数字或原始数值 |
| hexagram_name        | string  | 卦名        |
| upper_trigram        | string  | 上卦        |
| lower_trigram        | string  | 下卦        |
| moving_line          | string  | 动爻，如有     |
| changed_hexagram     | string  | 变卦，如有     |
| original_description | string  | 原始卦象描述    |
| source_file          | string  | 来源文件      |
| verified             | boolean | 是否核验      |

### 澳门卦象原则

1. 以实际数据库为唯一依据。
2. 不凭记忆补造卦象。
3. 不因为某一期缺资料而猜测。
4. 不同起卦来源必须分别保存。
5. 后续发现新的起卦体系，可以直接追加。
6. 原始卦象不能被分析结果覆盖。
7. 卦象修正必须保留版本记录。

---

## 5. 八肖资料

八肖属于辅助筛选资料。

| 字段              | 类型      | 必填 | 说明         |
| --------------- | ------- | -: | ---------- |
| zodiac_8        | array   |  否 | 本期数据库提供的八肖 |
| zodiac_source   | string  |  否 | 八肖来源       |
| zodiac_verified | boolean |  否 | 是否核验       |

### 八肖原则

八肖可以参与号码筛选，但：

> 八肖不等于最终号码。

八肖不能修改或覆盖原始卦象。

---

## 6. 49个固定号码与生肖映射

49个号码属于号码全集及生肖映射基础资料。

**49码不是分析层级。**

| 字段               | 类型     | 说明      |
| ---------------- | ------ | ------- |
| number_mapping   | object | 号码与生肖映射 |
| zodiac_numbers   | object | 各生肖对应号码 |
| mapping_version  | string | 映射规则版本  |
| excluded_numbers | array  | 明确排除号码  |

号码全集：

`01–49`

生肖映射按照实际使用的年份规则及 `mapping_version` 保存。

---

## 7. 候选号码

卦象资料与八肖资料经过分析后产生候选号码。

| 字段                | 类型     | 说明     |
| ----------------- | ------ | ------ |
| candidate_numbers | array  | 候选号码   |
| candidate_source  | array  | 候选来源   |
| candidate_reason  | object | 筛选原因   |
| candidate_version | string | 候选规则版本 |

候选号码必须可以追溯：

```text
澳门原始资料
      ↓
卦象
      ↓
八肖
      ↓
号码映射
      ↓
候选号码
```

---

## 8. 六级分析层

正式分析层级固定为：

```text
八肖
 ↓
28码
 ↓
24码
 ↓
20码
 ↓
16码
 ↓
12码
 ↓
8码
```

### 重要规定

**49码不属于分析层级。**

49码仅作为号码全集及生肖映射基础。

| 字段           | 类型    | 说明    |
| ------------ | ----- | ----- |
| candidate_28 | array | 28码候选 |
| candidate_24 | array | 24码候选 |
| candidate_20 | array | 20码候选 |
| candidate_16 | array | 16码候选 |
| candidate_12 | array | 12码候选 |
| candidate_8  | array | 8码候选  |

每一层必须能够说明：

1. 上一层输入。
2. 本层删除号码。
3. 本层保留号码。
4. 筛选规则。
5. 规则版本。
6. 数据来源。

---

## 9. 实际开奖结果

实际开奖结果独立保存。

| 字段              | 类型      | 说明     |
| --------------- | ------- | ------ |
| actual_result   | object  | 实际开奖资料 |
| actual_zodiac   | string  | 实际生肖   |
| actual_number   | integer | 实际号码   |
| result_verified | boolean | 是否核验   |

### 实际结果原则

实际结果用于：

* 历史验证
* 统计
* 回测
* 数据质量检查

实际结果不得反向修改原始卦象、八肖或原始候选过程。

---

## 10. 分析状态

| 字段                 | 类型      | 说明                                          |
| ------------------ | ------- | ------------------------------------------- |
| analysis_status    | string  | pending / processing / completed / verified |
| verified           | boolean | 数据是否完成核验                                    |
| verification_notes | string  | 核验说明                                        |
| notes              | string  | 其他备注                                        |

---

## 11. 来源追踪

所有重要资料必须保存来源。

最低要求：

```text
source
source_file
source_version
import_time
verified
```

目标：

> 任意一个结构化号码，都可以向上追溯到对应的原始资料。

---

## 12. 数据流

```text
RAW 原始资料
      ↓
澳门卦象数据库
      ↓
八肖数据库
      ↓
49个固定号码 / 生肖映射
      ↓
候选号码
      ↓
28码
      ↓
24码
      ↓
20码
      ↓
16码
      ↓
12码
      ↓
8码
      ↓
实际结果
      ↓
统计 / 验证 / 回测
```

---

## 13. 数据不可逆原则

### 基础资料

以下属于基础数据：

```text
RAW
卦象
八肖
生肖号码映射
实际开奖结果
```

基础资料不能因为分析流程而覆盖。

### 派生资料

以下属于分析产生的数据：

```text
candidate_numbers
28
24
20
16
12
8
```

派生数据可以重新计算。

但重新计算时必须记录：

```text
candidate_version
rule_version
schema_version
```

---

## 14. 新卦象追加规则

澳门后续出现新的起卦资料时：

**允许新增，不覆盖旧资料。**

例如未来数据库发现新的起卦来源：

```text
hexagrams:
  - source_A
  - source_B
  - source_C
  - new_source
```

直接追加新的来源记录。

禁止：

```text
删除旧卦象
↓
用新资料覆盖
```

---

## 15. 数据验证规则

每一期进入结构化数据库后检查：

* [ ] draw_id 唯一
* [ ] 年份正确
* [ ] 期号正确
* [ ] 来源明确
* [ ] RAW 已保存
* [ ] 卦象资料已保存（如有）
* [ ] 八肖资料已保存（如有）
* [ ] 号码映射版本明确
* [ ] 候选号码有来源
* [ ] 28码有来源
* [ ] 24码有来源
* [ ] 20码有来源
* [ ] 16码有来源
* [ ] 12码有来源
* [ ] 8码有来源
* [ ] 实际结果独立保存
* [ ] 数据已经核验

---

## 16. Schema 版本记录

### v1.0

建立澳门每期主数据统一标准：

* 基本期数信息
* RAW 原始资料
* 澳门卦象容器
* 八肖资料
* 49个固定号码及生肖映射
* 候选号码
* 28/24/20/16/12/8 六级分析
* 实际开奖结果
* 来源追踪
* 数据验证
* Git 版本控制
